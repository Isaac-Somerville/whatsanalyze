# Privacy & Offline Usage Notes

This document records the privacy posture, assumptions, and safe-use steps for this project, so future runs (especially offline) are predictable and non-surprising.

---

## Summary (TL;DR)

* **No intentional data exfiltration** is performed by the application when run offline.
* Analytics / telemetry code **exists in the codebase** but is **disabled or guarded** so it becomes a no-op.
* File uploads are processed **locally in the browser**.
* Any remaining network errors seen offline are due to **optional CDN assets** (fonts/icons) and are harmless.

---

## Analytics / Telemetry

### Present in dependencies

Some dependencies reference telemetry or observability libraries (e.g. Nuxt telemetry, OpenTelemetry). Their **presence in `pnpm-lock.yaml` does not mean they are active at runtime**.

They only become active if explicitly initialised or configured.

### Application-level analytics

The app contains Google Analytics helper code via:

* `gtagValues.js`

Originally this assumed `$nuxt.$gtag` always existed. When analytics is disabled or the app is offline, this caused a runtime crash.

### Mitigation applied

Analytics calls are now **guarded**:

* If `$gtag` is missing, the function returns immediately
* No network requests are made
* Application logic continues normally

Result: analytics becomes a **no-op** when offline or disabled.

---

## Network Behaviour (Expected)

When running **offline**, you may see browser console messages such as:

* `ERR_INTERNET_DISCONNECTED`
* Requests to `fonts.googleapis.com`
* Requests to `cdn.jsdelivr.net`

These are caused by:

* Google Fonts (Roboto)
* Material Design Icons

### Important

* These assets are **purely cosmetic**
* Failure to load them **does not affect data handling or privacy**
* No user data is sent in these requests

---

## File Handling

* Uploaded files are read using the browser `FileReader` API
* Files are parsed **entirely client-side**
* No automatic upload, sync, or background transmission occurs

If the browser is offline, transmission is **technically impossible**.

---

## Recommended Safe Run Procedure

For maximum privacy assurance:

1. Disconnect from Wi-Fi / Ethernet
2. Start the app locally:

   ```bash
   pnpm dev
   ```
3. Access only via:

   * `https://localhost:3000`
4. Ignore CDN/font warnings in DevTools
5. Confirm **no requests succeed** in the Network tab

---

## Threat Model Notes

What this setup protects against:

* Accidental analytics events
* Silent third-party data uploads
* Background telemetry during file processing

What it does *not* attempt to protect against:

* A compromised browser
* A malicious dependency deliberately added in future
* OS-level malware

---

## Future Maintenance Checklist

If revisiting this project later:

* [ ] Confirm `gtagEvent` is still guarded
* [ ] Check `nuxt.config.js` for analytics modules
* [ ] Re-run offline once to confirm no crashes
* [ ] Review new dependencies added since last audit

---

*Last updated: offline-safe configuration verified.*
