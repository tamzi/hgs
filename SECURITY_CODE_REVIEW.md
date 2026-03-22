# Security & Code Review — HGS Repository

## Overview

This repository contains 26 HTML5 browser games (13 Construct 2, 13 custom JS/CreateJS/Three.js). Below is a comprehensive review of security risks and areas for improvement.

---

## Security Risks

### 1. CRITICAL — Client-Side Gambling Logic With No Server Validation

**Affected games:** `baccarat`, `red_dog`, `roulette_royale`, `3d_soccer_slot`

All casino/gambling game outcomes (win/loss, payouts, bet validation) are determined entirely in client-side JavaScript. Parameters like `win_occurrence`, `payout` multipliers, `min_bet`, `max_bet`, and `money` are set in plain HTML and can be trivially modified by any user via browser DevTools.

**Examples:**
- `comple/baccarat/baccarat.htm:23` — `win_occurrence: 40` controls win rate
- `comple/red_dog/red_dog.htm:23` — `win_occurrence: 40`, payout array exposed
- `comple/roulette_royale/roulette_royale.htm:27` — `win_occurrence: 30`, `casino_cash: 4000`

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

**Risk:** Clickjacking, score manipulation, and data exfiltration if embedded in a malicious iframe host.

**Recommendation:**
- Replace `parent.*` calls with `window.parent.postMessage()` using a specific `targetOrigin`.
- Validate the origin of incoming messages with `event.origin` checks.
- Add `X-Frame-Options` or `Content-Security-Policy: frame-ancestors` headers to restrict embedding to trusted domains.

---

### 3. HIGH — Outdated JavaScript Libraries With Known Vulnerabilities

| Library | Version Used | Current Version | CVEs/Issues |
|---------|-------------|-----------------|-------------|
| jQuery | 2.0.3 (2013) | 3.7+ | XSS via `$()` selector (CVE-2015-9251, CVE-2019-11358, CVE-2020-11022, CVE-2020-11023) |
| jQuery | 2.1.1 (2014) | 3.7+ | Same as above |
| CreateJS | 2013.12.12 | 2.0+ | Multiple unfixed issues |
| CreateJS | 2014.12.12 | 2.0+ | Same |

**Risk:** Known XSS vulnerabilities in jQuery < 3.5.0. While direct exploitation is limited in a canvas-based game, any future DOM interaction or plugin use could be exploited.

**Recommendation:** Upgrade jQuery to 3.7+ and CreateJS to the latest version. Test each game for compatibility after upgrading.

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

The return value is never sanitized or decoded. While currently only compared to `"true"`, if this value is ever used in DOM manipulation or passed to `parent`, it could enable reflected XSS.

**Recommendation:** Apply `decodeURIComponent()` and sanitize/escape any parameter values before use.

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

**Recommendation:** Add CSP headers restricting `script-src` to `'self'`, and disallowing `unsafe-inline` where possible.

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

- Empty `<title>` tags across all games
- No `lang` attribute on `<html>` elements
- No `<meta charset>` using modern syntax
- No favicon references

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
