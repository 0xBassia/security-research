<div align="center">

# 🛡️ Security Research

### Vulnerability disclosures & root-cause analysis by [Mohamed Bassia (0xBassia)](https://github.com/0xBassia)

<a href="https://github.com/advisories?query=credit%3A0xBassia"><img src="https://img.shields.io/badge/Published%20CVEs-7-ff2d78?style=for-the-badge&labelColor=0d1117"/></a>
<a href="https://github.com/advisories?query=credit%3A0xBassia"><img src="https://img.shields.io/badge/High%20Severity-6%C3%97%20(CVSS%208.2)-d12026?style=for-the-badge&labelColor=0d1117"/></a>
<a href="https://github.com/advisories?query=credit%3A0xBassia"><img src="https://img.shields.io/badge/Focus-Prototype%20Pollution%20%26%20Web-00d4ff?style=for-the-badge&labelColor=0d1117"/></a>

</div>

---

This repository catalogs my coordinated vulnerability disclosures — each entry links to the official advisory and summarizes the **root cause**, **impact**, and **vulnerability class**. All findings were reported responsibly and fixed by the maintainers before publication.

## 📋 Disclosure Index

| CVE | Target | Severity | Class | Advisory |
|:---|:---|:---:|:---|:---:|
| CVE-2026-47378 | `nocodb` | 🟠 Medium | Broken access control | [GHSA](https://github.com/advisories/GHSA-4w6r-5c2j-qf5f) |
| CVE-2026-46510 | `form-data-objectizer` | 🔴 High `8.2` | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-m2hg-wjq3-28wq) |
| CVE-2026-46509 | `@ranfdev/deepobj` | 🔴 High `8.2` | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-x7q7-fchv-8h2j) |
| CVE-2026-45325 | `@tmlmobilidade/utils` | 🔴 High `8.2` | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-cmxg-94mg-jq94) |
| CVE-2026-45302 | `parse-nested-form-data` | 🔴 High `8.2` | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-xp7r-j8r6-j9h3) |
| CVE-2026-44483 | `@rvf/set-get` | 🔴 High `8.2` | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-c567-44rc-m5hq) |
| CVE-2026-9067 | `Schema & Structured Data for WP & AMP` | 🔴 High | Unauthenticated file upload | [WPScan](https://wpscan.com/vulnerability/7fac98eb-f82c-4705-a956-aba650945826) |

---

## 🧬 Theme: Prototype Pollution via Nested-Path Assignment

Six of these findings share a single root-cause pattern. Libraries that take a **user-controlled key path** and write it into an object — `set(obj, path, value)`, `setValueAtPath()`, form-data parsers that expand bracket/dot notation into nested objects — are dangerous when they walk the path **without rejecting reserved keys**.

```js
// Vulnerable pattern — no guard on reserved keys
function setPath(obj, path, value) {
  const keys = path.split('.');          // attacker controls `path`
  let cur = obj;
  for (let i = 0; i < keys.length - 1; i++) {
    cur = cur[keys[i]] ??= {};           // traversing "__proto__" lands on Object.prototype
  }
  cur[keys.at(-1)] = value;              // pollutes the prototype chain for ALL objects
}

// A single request reaches the sink:
setPath({}, '__proto__.polluted', 'yes');
({}).polluted; // => 'yes'   ← every plain object now carries the attacker's property
```

**Why it matters:** prototype pollution is rarely the final bug — it's a primitive. Polluting `Object.prototype` can escalate to denial of service, property-injection that flips security flags (`isAdmin`, `isAuthenticated`), or, when a gadget exists downstream, remote code execution. In each library below the sink was reachable directly from HTTP form data, so **no authentication and no special configuration was required**.

**The fix (applied in each):** reject `__proto__`, `constructor`, and `prototype` segments during path traversal, or assign with `Object.defineProperty` / null-prototype objects.

---

## 🔎 Per-CVE Summary

### CVE-2026-46510 — `form-data-objectizer`
Prototype pollution through **bracket-notation form keys**. The objectizer expanded `field[__proto__][x]`-style keys into nested objects without filtering reserved segments, so a crafted multipart field reached `Object.prototype`. **CVSS 8.2 (High).** → [advisory](https://github.com/advisories/GHSA-m2hg-wjq3-28wq)

### CVE-2026-45302 — `parse-nested-form-data`
`parseFormData()` walked bracket/dot-notation field names into nested objects. A field name beginning with `__proto__` (or containing `.__proto__.` mid-path) traversed onto `Object.prototype` and assigned there — polluting every plain object in the running process from a single request. **CVSS 8.2 (High).** → [advisory](https://github.com/advisories/GHSA-xp7r-j8r6-j9h3)

### CVE-2026-45325 — `@tmlmobilidade/utils`
Prototype pollution in `setValueAtPath()` — the path-assignment helper assigned object-type path segments without rejecting reserved keys. **CVSS 8.2 (High).** → [advisory](https://github.com/advisories/GHSA-cmxg-94mg-jq94)

### CVE-2026-46509 — `@ranfdev/deepobj`
Improperly controlled modification of object prototype attributes via dot-notation paths in the deep-object helper. **CVSS 8.2 (High).** → [advisory](https://github.com/advisories/GHSA-x7q7-fchv-8h2j)

### CVE-2026-44483 — `@rvf/set-get`
Prototype pollution in the set/get helper, reachable in practice via `@rvf/core`'s `preprocessFormData` — meaning the sink was driven directly by HTTP form data in consuming applications. **CVSS 8.2 (High).** → [advisory](https://github.com/advisories/GHSA-c567-44rc-m5hq)

### CVE-2026-47378 — `nocodb`
**Broken access control** (not prototype pollution): hidden columns were exposed through public shared-view endpoints. Fields a creator had hidden from a shared view were still returned by the underlying API, leaking data that was meant to be access-restricted. **Medium.** → [advisory](https://github.com/advisories/GHSA-4w6r-5c2j-qf5f)

### CVE-2026-9067 — `Schema & Structured Data for WP & AMP` (< 1.60)
**Unauthenticated arbitrary media upload** in the WordPress plugin — an upload path lacked authentication/authorization and adequate file-type validation, allowing an unauthenticated actor to place arbitrary media on the server. Disclosed via WPScan / Automattic. **High.** → [WPScan](https://wpscan.com/vulnerability/7fac98eb-f82c-4705-a956-aba650945826)

---

## 🤝 Disclosure Ethics

Every issue here was reported through the project's private vulnerability reporting channel (GitHub Security Advisories, WPScan/Automattic) and disclosed only after a fix shipped. No proof-of-concept that enables harm is published; these writeups describe vulnerability *classes* and root causes for defensive and educational value.

<div align="center">

<sub>Found something in your dependency tree? Enable Private Vulnerability Reporting — I'm always happy to help maintainers.</sub>

<br/><br/>

<a href="mailto:0xbassia@gmail.com"><img src="https://img.shields.io/badge/Report%20%2F%20Contact-0xbassia@gmail.com-00ff9c?style=for-the-badge&logo=gmail&logoColor=white&labelColor=0d1117"/></a>

</div>
