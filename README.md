# Security Research

A running list of bugs I've found and reported. Each one was disclosed privately first and only written up here after the maintainer shipped a fix.

Most of what I've reported lately is prototype pollution in small npm packages that turn user input into objects. There's also an access-control bug in NocoDB and an unauthenticated upload in a WordPress plugin. Seven CVEs so far.

[![Credited advisories](https://img.shields.io/badge/GitHub_Advisories-credit%3A0xBassia-2188FF?style=flat-square&logo=github)](https://github.com/advisories?query=credit%3A0xBassia)
[![CVEs](https://img.shields.io/badge/CVEs-7-ff2d78?style=flat-square)](https://github.com/advisories?query=credit%3A0xBassia)

## The list

| CVE | Where | Severity | Bug | Advisory |
|:----|:------|:---------|:----|:---------|
| CVE-2026-47378 | nocodb | Medium | Hidden columns leak through public shared views | [GHSA](https://github.com/advisories/GHSA-4w6r-5c2j-qf5f) |
| CVE-2026-46510 | form-data-objectizer | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-m2hg-wjq3-28wq) |
| CVE-2026-46509 | @ranfdev/deepobj | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-x7q7-fchv-8h2j) |
| CVE-2026-45325 | @tmlmobilidade/utils | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-cmxg-94mg-jq94) |
| CVE-2026-45302 | parse-nested-form-data | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-xp7r-j8r6-j9h3) |
| CVE-2026-44483 | @rvf/set-get | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-c567-44rc-m5hq) |
| CVE-2026-9067 | Schema & Structured Data for WP & AMP (< 1.60) | High | Unauthenticated media upload | [WPScan](https://wpscan.com/vulnerability/7fac98eb-f82c-4705-a956-aba650945826) |

## Why so many of these are prototype pollution

A surprising number of small libraries take a key path from the user, something like `user[name]` or `a.b.c`, and write it straight into an object without ever asking whether that path is trying to reach the prototype. If you can sneak `__proto__` into the path, you end up assigning to `Object.prototype`, and after that every plain object in the process quietly carries your value.

The bug almost always looks like this:

```js
function setPath(obj, path, value) {
  const keys = path.split('.');     // path comes from the request
  let cur = obj;
  for (let i = 0; i < keys.length - 1; i++) {
    cur = cur[keys[i]] ??= {};      // walking "__proto__" lands you on Object.prototype
  }
  cur[keys.at(-1)] = value;         // and now you've written to the prototype
}

setPath({}, '__proto__.polluted', 'yes');
({}).polluted; // 'yes' (a brand new object already has the property)
```

On its own this rarely does anything dramatic. It gets dangerous when something downstream reads one of those polluted properties: a config flag that was supposed to be undefined and is now `true`, a missing `isAdmin` that suddenly answers yes, a template that renders a value it never should have. In all six cases below the vulnerable function could be driven straight from HTTP form data, so there was no login step and nothing unusual to configure. A plain POST was enough to reach it.

The fix is boring and the same every time: reject `__proto__`, `constructor` and `prototype` while walking the path, or build the object with `Object.create(null)` so there's no prototype to poison.

## Notes on each one

<details>
<summary><b>CVE-2026-46510</b>: form-data-objectizer (prototype pollution)</summary>

<br>

The objectizer expanded bracket-notation form keys into nested objects. Feed it a key like `field[__proto__][x]` and it walked right onto the prototype. Reachable from a normal multipart form submission. CVSS 8.2.

[Advisory](https://github.com/advisories/GHSA-m2hg-wjq3-28wq)

</details>

<details>
<summary><b>CVE-2026-45302</b>: parse-nested-form-data (prototype pollution)</summary>

<br>

`parseFormData()` turns bracket and dot notation field names into nested objects. A field name that starts with `__proto__`, or has `.__proto__.` somewhere in the middle, walks onto `Object.prototype` and assigns there. One request pollutes every plain object in the process. CVSS 8.2.

[Advisory](https://github.com/advisories/GHSA-xp7r-j8r6-j9h3)

</details>

<details>
<summary><b>CVE-2026-45325</b>: @tmlmobilidade/utils (prototype pollution)</summary>

<br>

Same class, different sink. The `setValueAtPath()` helper assigned object-type path segments without filtering reserved keys, so a crafted path reached the prototype. CVSS 8.2.

[Advisory](https://github.com/advisories/GHSA-cmxg-94mg-jq94)

</details>

<details>
<summary><b>CVE-2026-46509</b>: @ranfdev/deepobj (prototype pollution)</summary>

<br>

The deep-object helper let dot-notation paths modify prototype attributes. No guard on the reserved keys while traversing. CVSS 8.2.

[Advisory](https://github.com/advisories/GHSA-x7q7-fchv-8h2j)

</details>

<details>
<summary><b>CVE-2026-44483</b>: @rvf/set-get (prototype pollution)</summary>

<br>

The interesting part here is the reach. The set/get helper itself was vulnerable, and `@rvf/core`'s `preprocessFormData` wired it directly to form data in apps that used the validation library. So the sink wasn't buried in some utility nobody calls, it sat on the request path. CVSS 8.2.

[Advisory](https://github.com/advisories/GHSA-c567-44rc-m5hq)

</details>

<details>
<summary><b>CVE-2026-47378</b>: nocodb (broken access control)</summary>

<br>

Not prototype pollution this time. Columns a creator had hidden from a public shared view were still being returned by the API behind that view. The UI hid them, the endpoint didn't, so anyone with the share link could read data that was meant to stay hidden. Medium.

[Advisory](https://github.com/advisories/GHSA-4w6r-5c2j-qf5f)

</details>

<details>
<summary><b>CVE-2026-9067</b>: Schema & Structured Data for WP & AMP, < 1.60 (unauthenticated upload)</summary>

<br>

A WordPress plugin one. An upload path was missing both the authentication check and proper file-type validation, which let an unauthenticated visitor drop arbitrary media onto the server. Reported through WPScan / Automattic rather than GitHub, which is why it doesn't show up under my GitHub advisory credits. High.

[WPScan](https://wpscan.com/vulnerability/7fac98eb-f82c-4705-a956-aba650945826)

</details>

## A note on disclosure

Everything here went through the project's private reporting channel first (GitHub Security Advisories, or WPScan for the WordPress one) and stayed quiet until a patch was out. I'm not publishing working exploits, just the root cause and enough detail to understand the class.

If you maintain a package and want another set of eyes on it, turn on private vulnerability reporting and send me a note at [0xbassia@gmail.com](mailto:0xbassia@gmail.com). Happy to help.
