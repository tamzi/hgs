# Security & Code Review — HGS Repository

## Overview

This repository contains 26 HTML5 browser games (13 Construct 2, 13 custom JS/CreateJS/Three.js). Below is a comprehensive review of security risks and areas for improvement.

---

## Security Risks

### 1. CRITICAL — Client-Side Gambling Logic With No Server Validation

**Affected games:** `baccarat`, `red_dog`, `roulette_royale`, `3d_soccer_slot`

All casino/gambling game outcomes (win/loss, payouts, bet validation) are determined entirely in client-side JavaScript. Parameters like `win_occurrence`, `payout` multipliers, `min_bet`, `max_bet`, and `money` are set in plain HTML and can be trivially modified by any user via browser DevTools.

**Examples:**

In `comple/baccarat/baccarat.htm`, the game settings object exposes:
```javascript
win_occurrence: 40  // controls win rate — trivially editable in DevTools
```

In `comple/red_dog/red_dog.htm`, both win rate and payout arrays are client-side:
```javascript
win_occurrence: 40,
payout: [1, 2, 3, 5, 10]  // payout multipliers fully exposed
```

In `comple/roulette_royale/roulette_royale.htm`, casino cash and win rate are plaintext:
```javascript
win_occurrence: 30,
casino_cash: 4000
```

**Risk:** A user can set `win_occurrence: 100` or modify payout multipliers to always win. If these games are ever used with real money or rewards, this is a critical exploit.

**Recommendation:** Move all game outcome logic, bet validation, and payout calculations to a server-side backend. The client should only render results received from the server.

---

### 2. HIGH — Unsafe Parent Frame Communication (No Origin Validation)

**Affected files:** All 12 `c2ctl.js` files and all `.htm` files with `parent.__ctlArcade*` calls.

Games communicate with their parent frame using direct `parent.*` calls without any origin validation via `postMessage`. A malicious parent page could embed these games and intercept or spoof score data, session events, and share events.

**Example (`comple/going_nuts_c2/c2ctl.js:1-5`):**
```javascript
function ctlArcadeSaveScore(iScore){
    if(parent.__ctlArcadeSaveScore){
        parent.__ctlArcadeSaveScore({score:iScore});
    }
}
```

**Example (`comple/baccarat/baccarat.htm:57-61`):**
```javascript
$(oMain).on("start_session", function(evt) {
    if(getParamValue('ctl-arcade') === "true"){
        parent.__ctlArcadeStartSession();
    }
});
```

**Risk:** Score manipulation and data exfiltration if embedded within the same origin by a malicious page. Note that browsers block cross-origin `parent.*` property access, so a cross-origin embedder cannot directly read or invoke `parent.__ctlArcade*` functions. The primary risk is same-origin embedding abuse or future migration to `postMessage` without proper origin checks.

**Recommendation:**
- Replace `parent.*` calls with `window.parent.postMessage()` using a specific `targetOrigin` (not `"*"`).
- Validate the origin of incoming messages with `event.origin` checks against an allowlist.
- Define a typed message schema with nonce/timestamp to prevent replay attacks.
- Add `X-Frame-Options` or `Content-Security-Policy: frame-ancestors` headers to restrict embedding to trusted domains.

---

### 3. HIGH — Outdated JavaScript Libraries With Known Vulnerabilities

| Library | Version Used | Current Version | CVEs/Issues |
|---------|-------------|-----------------|-------------|
| jQuery | 2.0.3 (2013) | 4.0.0 (Jan 2026) | XSS via `$()` selector (CVE-2015-9251, CVE-2019-11358, CVE-2020-11022, CVE-2020-11023) |
| jQuery | 2.1.1 (2014) | 4.0.0 (Jan 2026) | Same as above |
| CreateJS | 2013.12.12 | 1.0.0 (Sep 2017) | No longer actively maintained; multiple unfixed issues |
| CreateJS | 2014.12.12 | 1.0.0 (Sep 2017) | Same |

**Risk:** Known XSS vulnerabilities in jQuery < 3.5.0. While direct exploitation is limited in a canvas-based game, any future DOM interaction or plugin use could be exploited.

**Recommendation:** Upgrade jQuery to 4.0.0 and CreateJS to 1.0.0 (latest stable). Note that CreateJS is no longer actively maintained — consider migrating to an alternative canvas library for long-term support. Test each game for compatibility after upgrading.

---

### 4. MEDIUM — URL Parameter Parsing Without Sanitization

**Affected:** All games with `getParamValue()` function.

```javascript
function getParamValue(b) {
    for (var d = window.location.search.substring(1).split("&"), a = 0; a < d.length; a++) {
        var c = d[a].split("=");
        if (c[0] == b) return c[1]
    }
}
```

The return value is never sanitized or decoded. Currently, this value is only compared to the string `"true"` and is never inserted into the DOM or forwarded to any sink, so the present XSS risk is theoretical rather than exploitable. However, if this utility is ever reused for DOM manipulation or passed to `parent.postMessage`, it could enable reflected XSS.

**Recommendation:** Apply `decodeURIComponent()` and sanitize/escape any parameter values before use. As a defense-in-depth measure, this should be addressed before the codebase grows.

---

### 5. MEDIUM — Console Logging of Share Data (Information Disclosure)

**Affected file:** `comple/going_nuts_c2/c2ctl.js:44-48`

```javascript
function ctlArcadeShareEvent(szImg, szTitle, szMsg, szMsgShare){
    console.log(szImg);
    console.log(szTitle);
    console.log(szMsg);
    console.log(szMsgShare);
    ...
}
```

**Risk:** Debug logging left in production code. Leaks share content (images, titles, messages) to the browser console. Could expose user-specific data.

**Recommendation:** Remove all `console.log()` calls from production code, or gate them behind a debug flag.

---

### 6. MEDIUM — No Content Security Policy (CSP)

None of the 26 HTML files include a `Content-Security-Policy` meta tag or header.

**Risk:** Without CSP, the games are vulnerable to code injection if any XSS vector is found. An attacker could load external scripts, exfiltrate data, or modify game behavior.

**Recommendation:** Adopt a phased CSP approach:
1. **Phase 1 (immediate):** Add a permissive CSP that reflects current code reality — allow `'self'` and `'unsafe-inline'` for `script-src` since games currently rely on inline `<script>` blocks for initialization.
2. **Phase 2 (prerequisite refactor):** Move all inline JavaScript to external `.js` files (see Code Quality §4 below), or adopt nonce-based CSP (`script-src 'nonce-<random>'`).
3. **Phase 3 (strict):** Once inline scripts are eliminated, tighten CSP to disallow `'unsafe-inline'` entirely.

Applying a strict CSP that disallows inline scripts without first refactoring the existing inline `<script>` blocks will break game startup.

---

### 7. LOW — Deprecated HTML5 AppCache

**Affected:** 12 Construct 2 games using `offline.appcache` manifests.

AppCache is deprecated and removed from modern browsers. It has known security issues including cache poisoning over insecure networks.

**Recommendation:** Migrate to Service Workers for offline support.

---

## Code Quality Improvements

### 1. No Build System or Dependency Management

There is no `package.json`, no bundler (webpack/vite), and no build pipeline. Libraries are vendored as minified files with no version tracking.

**Recommendation:** Add a `package.json` to track dependencies. Use a bundler for minification, tree-shaking, and consistent builds.

### 2. No Error Handling

Games have virtually no try-catch blocks, no error boundaries, and no error reporting. If a game asset fails to load or a runtime error occurs, the game silently breaks.

**Recommendation:** Add error handling around asset loading and game initialization. Consider an `onerror` global handler for diagnostics.

### 3. Duplicated Code Across Games

The `getParamValue()` function, `sizeHandler()`, `isIOS()` detection, and CTL Arcade integration code are copy-pasted identically across all 26 games.

**Recommendation:** Extract shared utilities into a common `shared/utils.js` file and reference it from each game.

### 4. Inline JavaScript in HTML Files

All game initialization code is embedded in `<script>` blocks within `.htm` files. This makes the code harder to maintain, test, and audit.

**Recommendation:** Move initialization code into separate `.js` files (e.g., `init.js`) and load them via `<script src="...">`.

### 5. No Linting or Formatting

No `.eslintrc`, `.prettierrc`, or equivalent configuration exists. Code style is inconsistent (mixed tabs/spaces, inconsistent naming).

**Recommendation:** Add ESLint and Prettier configurations. Run a one-time format pass across all custom JavaScript.

### 6. Missing HTML Best Practices

Several games are missing standard HTML best practices. Note that some games already include non-empty titles, modern `<meta charset>` declarations, and favicon links — the issues below apply to a subset of games, not all 26 uniformly:

- Empty or missing `<title>` tags in some games
- No `lang` attribute on `<html>` elements (affects all games)
- Missing `<meta charset="UTF-8">` in some games (others already have it)
- Missing favicon references in some games

### 7. Hardcoded Text Strings

Game text (UI labels, messages) is hardcoded in JavaScript constants. Only `word_finder` supports multiple languages via separate wordlist files.

**Recommendation:** For games requiring localization, move strings to JSON locale files.

---

## Summary

| Severity | Finding | Type |
|----------|---------|------|
| CRITICAL | Client-side gambling logic, no server validation | Security |
| HIGH | Unsafe parent frame communication, no origin check | Security |
| HIGH | Outdated jQuery (2.0.3/2.1.1) with known CVEs | Security |
| MEDIUM | Unsanitized URL parameter parsing | Security |
| MEDIUM | Debug console.log left in production | Security |
| MEDIUM | No Content Security Policy | Security |
| LOW | Deprecated AppCache usage | Security |
| — | No build system or dependency management | Quality |
| — | No error handling | Quality |
| — | Duplicated utility code across 26 games | Quality |
| — | Inline JavaScript in HTML | Quality |
| — | No linting or formatting config | Quality |
| — | Missing HTML best practices | Quality |
| — | Hardcoded text strings | Quality |
