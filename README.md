# Security Research

A running list of bugs I've found and reported. Each one was disclosed privately first, and only written up here once a fix shipped or the plugin was pulled from the directory.

A lot of what's here is prototype pollution in small npm packages that turn user input into objects. The rest is a mix: three unauthenticated account takeovers, two unauthenticated uploads that reach code execution, a multisite privilege boundary that let a subsite admin run PHP across a whole network, an access-control bug in NocoDB, a SQL injection that arrives through an imported calendar feed, and a handful of SSRFs. Twenty CVEs so far, plus four advisories that shipped without one.

The highest is CVE-2026-14560 at 10.0: an unauthenticated file upload that trusts the browser about the file type and keeps the filename you give it.

[![Credited advisories](https://img.shields.io/badge/GitHub_Advisories-credit%3A0xBassia-2188FF?style=flat-square&logo=github)](https://github.com/advisories?query=credit%3A0xBassia)
[![CVEs](https://img.shields.io/badge/CVEs-20-58a6ff?style=flat-square)](https://github.com/advisories?query=credit%3A0xBassia)
[![Advisories without a CVE](https://img.shields.io/badge/GHSA_without_CVE-4-388bfd?style=flat-square)](#advisories-without-a-cve)

## The list

| CVE | Where | Severity | Bug | Advisory |
|:----|:------|:---------|:----|:---------|
| CVE-2026-55671 | zitadel/zitadel (Go) | Low | SSRF and denylist bypass in outgoing HTTP components | [GHSA](https://github.com/advisories/GHSA-29jh-8cfq-rr8x) |
| CVE-2026-47378 | nocodb | Medium | Hidden columns leak through public shared views | [GHSA](https://github.com/advisories/GHSA-4w6r-5c2j-qf5f) |
| CVE-2026-46510 | form-data-objectizer | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-m2hg-wjq3-28wq) |
| CVE-2026-46509 | @ranfdev/deepobj | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-x7q7-fchv-8h2j) |
| CVE-2026-45325 | @tmlmobilidade/utils | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-cmxg-94mg-jq94) |
| CVE-2026-45302 | parse-nested-form-data | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-xp7r-j8r6-j9h3) |
| CVE-2026-44483 | @rvf/set-get | High (8.2) | Prototype pollution | [GHSA](https://github.com/advisories/GHSA-c567-44rc-m5hq) |
| CVE-2026-14560 | Teddy Bear Customize Addon (<= 1.0.5) | Critical (10.0) | Unauthenticated arbitrary file upload to RCE | [WPScan](https://wpscan.com/vulnerability/aba51906-91dc-4e75-ad44-373fe128deee) |
| CVE-2026-14559 | Teddy Bear Customize Addon (<= 1.0.5) | Critical (9.8) | Unauthenticated account takeover, password never verified | [WPScan](https://wpscan.com/vulnerability/90cbbed5-9662-4405-88b8-bd55854dccdf) |
| CVE-2026-14565 | Advanced Customized Prompts (<= 1.0.1) | High (8.0) | Subscriber+ stored XSS via product popup configuration | [WPScan](https://wpscan.com/vulnerability/613e5421-9da5-408a-b35d-b0def567e8db) |
| CVE-2026-14562 | Teddy Bear Customize Addon (<= 1.0.5) | Medium (5.3) | Unauthenticated order and attachment data disclosure | [WPScan](https://wpscan.com/vulnerability/890f55f9-0bd0-4816-a666-a735b41ce5c7) |
| CVE-2026-14563 | Advanced Customized Prompts (<= 1.0.1) | Unrated | Unauthenticated account takeover, password never verified | [WPScan](https://wpscan.com/vulnerability/7c7ad196-dff3-485f-9a50-8705bd796fb3) |
| CVE-2026-14566 | Advanced Customized Prompts (<= 1.0.1) | Unrated | Subscriber+ WooCommerce order metadata tampering | [WPScan](https://wpscan.com/vulnerability/01094477-a9ad-41d9-9acd-f6ed37e6e605) |
| CVE-2026-14561 | Authora, Easy Login with Mobile Number (< 1.7.7) | Critical (9.8) | Unauthenticated account takeover via OTP disclosure | [WPScan](https://wpscan.com/vulnerability/4025601f-ed33-4772-b716-a9979830e10d) |
| CVE-2026-17533 | All-in-One WP Migration and Backup (< 7.108) | High (7.2) | Subsite admin to network-wide PHP execution | [WPScan](https://wpscan.com/vulnerability/13b57cdc-d954-4db6-94c2-53ad04ab0d34) |
| CVE-2026-91024 | Booking Manager (< 2.1.21) | Medium (6.8) | Author+ SQLi via imported iCalendar feed UID | [WPScan](https://wpscan.com/vulnerability/a5472152-0b96-4fc6-af90-2f1b01811237) |
| CVE-2026-9815 | MagicForm (<= 0.1.3) | High | Unauthenticated file upload to RCE | [WPScan](https://wpscan.com/vulnerability/043f449f-fc65-4218-83d2-7742e62f2af3) |
| CVE-2026-12516 | Fediverse Embeds (< 1.5.8) | High (7.5) | Unauthenticated SSRF via media proxy | [WPScan](https://wpscan.com/vulnerability/2ac80164-03b7-4966-b022-833b4194de80) |
| CVE-2026-12517 | Fediverse Embeds (< 1.5.8) | Medium (5.3) | Unauthenticated SSRF via site-info endpoint | [WPScan](https://wpscan.com/vulnerability/460a996f-e27d-47e8-9d68-9e6be93100c0) |
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

On its own this rarely does anything dramatic. It gets dangerous when something downstream reads one of those polluted properties: a config flag that was supposed to be undefined and is now `true`, a missing `isAdmin` that suddenly answers yes, a template that renders a value it never should have. In all five cases below the vulnerable function could be driven straight from HTTP form data, so there was no login step and nothing unusual to configure. A plain POST was enough to reach it.

The fix is boring and the same every time: reject `__proto__`, `constructor` and `prototype` while walking the path, or build the object with `Object.create(null)` so there's no prototype to poison.

## Notes on each one

<details>
<summary><b>CVE-2026-14560</b>: Teddy Bear Customize Addon, <= 1.0.5 (unauthenticated upload to RCE)</summary>

<br>

My first 10.0, and it got there by stacking three small decisions that are each survivable on their own.

The upload handler took the browser's word for what the file was, trusting the client-supplied content type instead of checking the actual bytes. It kept the original filename rather than generating its own. And the endpoint had no authentication in front of it. Any one of those is a bad idea. Together they mean an anonymous request can put a `.php` file on disk, under a name it chose, in a place the server will happily execute.

Nothing clever is required, which is why it scores what it scores. Every metric that can be maximised is: no privileges, no interaction, network reachable, and full impact on all three of confidentiality, integrity and availability.

The plugin was removed from the directory rather than patched, and WPScan is holding the proof of concept.

[WPScan](https://wpscan.com/vulnerability/aba51906-91dc-4e75-ad44-373fe128deee)

</details>

<details>
<summary><b>CVE-2026-14559</b>: Teddy Bear Customize Addon, <= 1.0.5 (unauthenticated account takeover)</summary>

<br>

Supply an email address, get a session. The login path issued an authenticated session for whatever address you handed it without ever checking the password.

There's no bypass technique to describe here and no trick to it. The check simply isn't in the code. Any registered address works, administrators included, and there's nothing to brute force because there's nothing being compared.

I find these more interesting than they look. A missing comparison reads as normal code during review, because the absence of a line is much harder to notice than a wrong line. CVSS 9.8.

[WPScan](https://wpscan.com/vulnerability/90cbbed5-9662-4405-88b8-bd55854dccdf)

</details>

<details>
<summary><b>CVE-2026-14562</b>: Teddy Bear Customize Addon, <= 1.0.5 (unauthenticated order data disclosure)</summary>

<br>

The third one in the same plugin. An endpoint returned WooCommerce order metadata along with the URLs of files customers had uploaded to their orders, and never checked whether the caller had any relationship to the order being asked about.

So you could walk order IDs and read other people's order details, then follow the attachment URLs and pull whatever they had uploaded. On a store selling customised products those attachments are often personal: photos, names, artwork for the thing being made.

Rated 5.3 because it's read-only, though for the people whose files are sitting behind those URLs that scoring feels a little abstract.

[WPScan](https://wpscan.com/vulnerability/890f55f9-0bd0-4816-a666-a735b41ce5c7)

</details>

<details>
<summary><b>CVE-2026-14563</b>: Advanced Customized Prompts, <= 1.0.1 (unauthenticated account takeover)</summary>

<br>

The same bug as CVE-2026-14559, in a different plugin, found the same week. An unauthenticated action issued a session for a supplied email address without verifying the password. Administrators included, and unregistered addresses got an account created for them.

WPScan published this one without a CVSS score. By the metrics it looks identical to the 9.8 above, but I'm not going to put a number on it that the advisory doesn't carry, so it sits here unrated.

Plugin removed from the directory, proof of concept withheld.

[WPScan](https://wpscan.com/vulnerability/7c7ad196-dff3-485f-9a50-8705bd796fb3)

</details>

<details>
<summary><b>CVE-2026-14565</b>: Advanced Customized Prompts, <= 1.0.1 (Subscriber+ stored XSS)</summary>

<br>

Saving the popup configuration attached to a product went through with no capability check, no ownership check and no nonce, and the stored values were never escaped on the way back out.

So the lowest-privileged account the platform has, a subscriber, could write JavaScript into a product that anyone later viewing it would run. On a WooCommerce store the people most likely to open that product in the admin are the ones with the permissions worth stealing.

Four separate controls would each have stopped it independently: capability, ownership, nonce, output escaping. None of them were there. CVSS 8.0.

[WPScan](https://wpscan.com/vulnerability/613e5421-9da5-408a-b35d-b0def567e8db)

</details>

<details>
<summary><b>CVE-2026-14566</b>: Advanced Customized Prompts, <= 1.0.1 (Subscriber+ order metadata tampering)</summary>

<br>

Same missing trio as the XSS above, pointed at a different target. Updating the custom metadata on a WooCommerce order item took an order ID from the request and did no capability, ownership or nonce check on it, so any logged-in user could edit the metadata on orders belonging to other people.

Classic IDOR, CWE-639. Published without a CVSS score, so it's unrated here too.

[WPScan](https://wpscan.com/vulnerability/01094477-a9ad-41d9-9acd-f6ed37e6e605)

</details>

<details>
<summary><b>CVE-2026-14561</b>: Authora, Easy Login with Mobile Number, < 1.7.7 (unauthenticated account takeover)</summary>

<br>

The one that still surprises me. Authora lets people sign in with a mobile number and a one-time code. The AJAX action that generates that code also hands it back in its own JSON response, together with a valid verification token. The secret that is supposed to travel out of band to the user's phone gets returned to whoever asked for it.

So if you know someone's registered number you can request a code, read it straight out of the reply, and complete the login as them. Administrators included. There is no authentication anywhere in the chain. Asking for a number that isn't registered yet is bad in a different direction: the verify step creates a fresh account and logs you into it.

The fix is the obvious one. Send the code to the phone and keep it out of the response body. Fixed in 1.7.7. CVSS 9.8, the highest I've had.

[WPScan](https://wpscan.com/vulnerability/4025601f-ed33-4772-b716-a9979830e10d)

</details>

<details>
<summary><b>CVE-2026-17533</b>: All-in-One WP Migration and Backup, < 7.108 (subsite admin to network-wide PHP execution)</summary>

<br>

WordPress multisite exists to keep tenants apart. A subsite administrator runs their own site and is deliberately denied the ability to install plugins or themes, because that would mean executing code on a box shared with every other tenant on the network.

The plugin's migration import wasn't restricted to network administrators, so an ordinary subsite admin could reach it. An import is by definition a way to write files and have them run, which puts arbitrary PHP execution across the whole network in the hands of someone who was never meant to have it. The code execution isn't really the interesting part. The interesting part is that the privilege boundary multisite is built around simply wasn't being checked.

WPScan is holding the proof of concept until 13 September 2026 so people have time to update, so there isn't one here either. Fixed in 7.108. CVSS 7.2.

[WPScan](https://wpscan.com/vulnerability/13b57cdc-d954-4db6-94c2-53ad04ab0d34)

</details>

<details>
<summary><b>CVE-2026-55671</b>: zitadel/zitadel (SSRF and denylist bypass)</summary>

<br>

First Go target on this list. Zitadel makes outbound HTTP requests in several places where the destination comes from user configuration: webhook notification channels, OIDC back-channel logout endpoints, and SAML metadata fetches. Those URLs weren't being validated against the internal denylist at all.

The denylist itself had gaps as well. It could be walked past with DNS rebinding, it followed redirects, it allowed an HTTPS to HTTP downgrade, and it was missing a number of common private ranges. On cloud deployments still permitting IMDSv1, the metadata endpoint was reachable.

Scored Low because these features expect particular response shapes, which limits how much you can actually read back. Fixed in 4.15.2. CVSS 2.3.

[Advisory](https://github.com/advisories/GHSA-29jh-8cfq-rr8x)

</details>

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
<summary><b>CVE-2026-91024</b>: Booking Manager, < 2.1.21 (Author+ SQL injection)</summary>

<br>

First SQL injection on this list, and what makes it worth writing up is where the input comes from.

Booking Manager can import bookings from an external iCalendar feed. An author-level user points it at a URL, the plugin fetches the `.ics` file, and reads the UID out of each event into `sync_gid`. That value then goes into a SQL query with no sanitising and no escaping.

So the injection point isn't a form field or a query parameter. It's a line inside a calendar file sitting on a server the attacker controls. The plugin treats the feed as trusted because a logged-in user configured the import, but the contents of that feed were never that user's to vouch for. Anything that crosses the network and lands in a query is input, no matter who typed the URL.

Author is not a high bar on a site that takes guest contributors. Fixed in 2.1.21. CVSS 6.8.

[WPScan](https://wpscan.com/vulnerability/a5472152-0b96-4fc6-af90-2f1b01811237)

</details>

<details>
<summary><b>CVE-2026-9815</b>: MagicForm, <= 0.1.3 (unauthenticated upload to RCE)</summary>

<br>

MagicForm exposes an unauthenticated AJAX upload action, and when a form leaves a field's extension allowlist empty the plugin stops validating the file type at all. So you can upload a PHP file and run code on the server, no login required. Reported through WPScan and credited to me. High.

[WPScan](https://wpscan.com/vulnerability/043f449f-fc65-4218-83d2-7742e62f2af3)

</details>

<details>
<summary><b>CVE-2026-12516</b>: Fediverse Embeds, < 1.5.8 (unauthenticated SSRF via media proxy)</summary>

<br>

The plugin has a media-proxying endpoint that fetches a URL server-side and hands you back the response body, but it never checks where that URL points. No auth needed. So you can aim it at internal services or private-network addresses (cloud metadata, localhost admin panels, anything the server can reach) and read the response. Full-read SSRF, effectively an open proxy too. CVSS 7.5.

[WPScan](https://wpscan.com/vulnerability/2ac80164-03b7-4966-b022-833b4194de80)

</details>

<details>
<summary><b>CVE-2026-12517</b>: Fediverse Embeds, < 1.5.8 (unauthenticated SSRF via site-info endpoint)</summary>

<br>

Same plugin, second sink. The site-info endpoint fetches a remote page server-side and returns the parsed metadata. The gating nonce is printed on any public page that carries an embed, so it isn't really a barrier, and again the destination isn't validated. You get back parsed page metadata rather than the raw body, so it leaks a bit less than the media proxy, hence the lower score. CVSS 5.3.

[WPScan](https://wpscan.com/vulnerability/460a996f-e27d-47e8-9d68-9e6be93100c0)

</details>

<details>
<summary><b>CVE-2026-9067</b>: Schema & Structured Data for WP & AMP, < 1.60 (unauthenticated upload)</summary>

<br>

A WordPress plugin one. An upload path was missing both the authentication check and proper file-type validation, which let an unauthenticated visitor drop arbitrary media onto the server. Reported through WPScan / Automattic rather than GitHub, which is why it doesn't show up under my GitHub advisory credits. High.

[WPScan](https://wpscan.com/vulnerability/7fac98eb-f82c-4705-a956-aba650945826)

</details>

## Advisories without a CVE

Four more that were published as GitHub Security Advisories but never had a CVE assigned. Same process, same rules. They don't show up under `credit:0xBassia` either, because the global Advisory Database only indexes advisories that receive a CVE, so these stay on the repo that published them.

| Advisory | Where | Severity | Bug |
|:---------|:------|:---------|:----|
| [GHSA-mrf2-rxph-r28h](https://github.com/Kunena/Kunena-Forum/security/advisories/GHSA-mrf2-rxph-r28h) | Kunena Forum (<= 7.0.4) | High (8.2) | Unauthenticated attachment privacy modification |
| [GHSA-wfph-gf24-pjqg](https://github.com/Kunena/Kunena-Forum/security/advisories/GHSA-wfph-gf24-pjqg) | Kunena Forum (<= 7.0.4) | Medium (4.3) | Missing CSRF check on the topic rating endpoint |
| [GHSA-px35-hwj4-wqrh](https://github.com/Kunena/Kunena-Forum/security/advisories/GHSA-px35-hwj4-wqrh) | Kunena Forum (<= 7.0.4) | Low (3.5) | Arbitrary-user avatar overwrite |
| [GHSA-354h-gmhv-mr9c](https://github.com/TryGhost/Ghost/security/advisories/GHSA-354h-gmhv-mr9c) | TryGhost/Ghost (< 6.27.0) | Low (2.7) | SSRF in the webhook trigger |

### The Kunena three came from one thread

I reported the avatar bug first. `UserController::upload()` was the only state-changing task in that controller that didn't call `Session::checkToken()`, and it also took a caller-supplied `userid` and wrote the avatar to a path derived from it. So a logged-in member could overwrite anyone's avatar, and because there was no token check, a crafted page could make an administrator do it on their behalf.

While the fix was in review, a maintainer left a comment on the pull request: *"I think it would be good to check all ajax calls, better safe then sorry."* That's an open invitation, so I went through the rest of them.

Two more turned up in `TopicController`, and both were worse than the one I started with:

- `setprivate` had no token check, no authentication check and no ownership check at all. An unauthenticated request could mark any attachment private just by knowing its numeric ID, and the IDs are sequential. Scored 8.2, which made it the highest of the three.
- `setrate` had no token check either, and its authorization condition was `$user->exists() || $this->config->ratingEnabled`. With the common `ratingEnabled` default, that second branch means an anonymous request writes straight through.

The lesson I took from it: when a maintainer tells you the class probably repeats elsewhere in the codebase, believe them and go look. The best of the three findings came from the audit, not from the original report.

### Ghost

Ghost's webhooks let a staff user register a URL that the server calls on certain events. The destination wasn't validated, so a staff user could point one at an internal address and use the Ghost server to probe things it shouldn't reach.

Low impact, since you need staff access first and you don't get much back. But it's structurally the same bug as the zitadel one above: a feature whose entire job is *fetch a URL the user gave us*, shipped without a check on where that URL points. That pattern is everywhere once you start looking for it. Fixed in 6.27.0. CVSS 2.7.

## Two plugins, six bugs, one habit

Six of the CVEs above came out of two WooCommerce add-ons I looked at in the same stretch, and they failed in almost exactly the same way.

Both had authentication paths that issued a session without ever checking a password. Both had AJAX actions that changed or returned other people's data with no capability check, no ownership check and no nonce. It reads like the same missing mental step in two different codebases: the author thought about what the feature should do for a legitimate user, and never about who else could call it.

Neither plugin got patched. Both were pulled from the WordPress.org directory instead, which is what usually happens when the author has moved on. It closes the hole for anyone installing fresh, and does nothing at all for sites that already have the plugin sitting there.

## A note on disclosure

Everything here went through the project's private reporting channel first (GitHub Security Advisories, or WPScan for the WordPress ones) and stayed quiet until it was resolved. Usually that means a patch. Twice it meant the plugin was pulled from the WordPress.org directory instead, because nobody was maintaining it any more. Either way I waited for the advisory to go public before writing anything here, and I'm not publishing working exploits, just the root cause and enough detail to understand the class.

If you maintain a package and want another set of eyes on it, turn on private vulnerability reporting and send me a note at [0xbassia@gmail.com](mailto:0xbassia@gmail.com). Happy to help.
