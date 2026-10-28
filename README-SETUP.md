# JCT Cases PWA — setup guide

This package contains a branded installable PWA shell for Julie Charitable Trust.
It keeps the existing Google Apps Script web app as the backend/UI.

## Files
- `index.html` — PWA shell that embeds the current Apps Script web app.
- `manifest.webmanifest` — app name, icon, launch URL and standalone display.
- `sw.js` — caches only the PWA shell/static assets. It does **not** cache case records or support offline registration/search.
- `icons/jct-icon-192.png` and `icons/jct-icon-512.png` — generated from the supplied trust logo.

Existing Apps Script URL configured in `index.html`:
`https://script.google.com/macros/s/AKfycbwrxE-UignTUAwfRCLRXyZ1TQLaf_Z6VLICvjaihZGnO_5RTns58a90qeKOwhDBzxRznQ/exec`

## Important integration requirement

The shell embeds Apps Script in an iframe. Google Apps Script normally restricts framing. Before expecting this to work, update the three HTML page returns in `doGet(e)` (Home, Register and Search) to allow framing by the PWA host:

Add this method to each page's returned `HtmlOutput` chain:
`.setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)`

For the Home template, the return chain should look like:
```javascript
return template.evaluate()
  .setTitle("Julie Charitable Trust - Home")
  .addMetaTag("viewport", "width=device-width, initial-scale=1")
  .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
```

For Register and Search, add the same `.setXFrameOptionsMode(...)` to their existing `createHtmlOutputFromFile(...)` chains.

**Security note:** `ALLOWALL` permits other websites to frame these pages. Only do this if you accept that trade-off and your app's deployment/access settings protect sensitive case data. Do not make private case data publicly accessible just to install a PWA.

## Navigation requirement inside the iframe

The current Home page may use `target="_top"` or JavaScript such as `window.top.location.href = ...`. That escapes the PWA shell and opens Apps Script outside the installed app window.

For the two Home page buttons, use the deployed URL and navigate the **iframe itself**, not the top window:
```html
<a href="https://script.google.com/macros/s/AKfycbwrxE-UignTUAwfRCLRXyZ1TQLaf_Z6VLICvjaihZGnO_5RTns58a90qeKOwhDBzxRznQ/exec?page=register" target="_self">Case Registration</a>
<a href="https://script.google.com/macros/s/AKfycbwrxE-UignTUAwfRCLRXyZ1TQLaf_Z6VLICvjaihZGnO_5RTns58a90qeKOwhDBzxRznQ/exec?page=search" target="_self">Search Case Details</a>
```
Remove any `onclick` handler that assigns to `window.top.location.href` for these links. Keep each button's existing inner HTML and CSS classes. Inspect Register/Search pages for any other navigation links using `_top` and adjust only if they must stay inside the PWA shell.

## Hosting on GitHub Pages

1. Sign in to GitHub and create a new repository, e.g. `jct-cases-pwa`.
2. Upload all files in this package, preserving the `icons` folder structure.
3. In the repository, open **Settings → Pages**.
4. Choose **Deploy from a branch**, select `main` and `/ (root)`, then save.
5. Wait for GitHub Pages to publish the site. It will provide an HTTPS URL similar to `https://YOUR-USERNAME.github.io/jct-cases-pwa/`.
6. Open that URL. First check the app iframe loads and registration/search navigation stays inside the shell.

Do not distribute the URL to all staff until the integration test passes.

## Install on Android

1. Open the GitHub Pages HTTPS URL in Chrome.
2. Wait for the app to load.
3. Open Chrome's menu and select **Install app** if available, otherwise **Add to home screen**.
4. Name it `JCT Cases` and install/add it.
5. Launch from the new icon and test registration and search.

## Install on iPhone

1. Open the GitHub Pages HTTPS URL in Safari.
2. Tap Share → **Add to Home Screen**.
3. Name it `JCT Cases`, then tap Add.
4. Open it from the home screen and test the app.

## Known limitations and precautions

- This package does not make the Apps Script app offline-capable. Registration, search, Sheets and photos still need network access.
- The iframe approach requires the Apps Script `ALLOWALL` change and compatible navigation.
- Some Google sign-in/deployment configurations may not work as expected inside an iframe. Test with the actual staff account and deployment access policy.
- If iframe embedding or Google authentication fails, the safer fallback is a branded PWA launch screen with an **Open JCT Cases** button that opens the Apps Script URL in the browser. A fully integrated PWA may require moving the frontend to a host that can call a suitable authenticated backend.
- Keep the Apps Script deployment's access permissions restricted to the intended staff where possible. Do not share case records publicly.
