# Versioning

_Part of [REST API](README.md) interview notes._

## Table of Contents

**🟢 Easy**
- [1. Why do REST APIs need versioning?](#1-why-do-rest-apis-need-versioning)
- [2. What is URL path versioning?](#2-what-is-url-path-versioning)

**🟡 Medium**
- [3. What is header-based versioning?](#3-what-is-header-based-versioning)
- [4. What is media-type (content negotiation) versioning?](#4-what-is-media-type-content-negotiation-versioning)
- [5. How do you deprecate an old API version gracefully?](#5-how-do-you-deprecate-an-old-api-version-gracefully)

**🔴 Hard**
- [6. What are the tradeoffs between URL versioning and header versioning?](#6-what-are-the-tradeoffs-between-url-versioning-and-header-versioning)
- [7. How would you support multiple API versions in the same codebase without massive duplication?](#7-how-would-you-support-multiple-api-versions-in-the-same-codebase-without-massive-duplication)
- [8. How do you communicate breaking changes to API consumers ahead of time?](#8-how-do-you-communicate-breaking-changes-to-api-consumers-ahead-of-time)
- [9. What's the difference between versioning the whole API vs. versioning individual endpoints?](#9-whats-the-difference-between-versioning-the-whole-api-vs-versioning-individual-endpoints)
- [10. Does semantic versioning apply to REST API versioning?](#10-does-semantic-versioning-apply-to-rest-api-versioning)

---

### 1. Why do REST APIs need versioning? 🟢

- Once an API has real consumers, you can't freely change response shapes/behavior without breaking them. Versioning lets you introduce breaking changes in a **new** version while existing clients keep working against the old one, until they're ready to migrate.

[↑ Back to top](#table-of-contents)

---

### 2. What is URL path versioning? 🟢

- The version is embedded directly in the URL path — the most common and easiest-to-understand approach.

```
GET /api/v1/users
GET /api/v2/users
```

[↑ Back to top](#table-of-contents)

---

### 3. What is header-based versioning? 🟡

- The version is specified via a custom request header instead of the URL — keeps URLs version-agnostic/"clean," but is less discoverable (you can't tell the version just by looking at the URL, e.g. in browser address bars or logs).

```http
GET /api/users
X-API-Version: 2
```

[↑ Back to top](#table-of-contents)

---

### 4. What is media-type (content negotiation) versioning? 🟡

- The version is encoded into the `Accept` header's media type itself, leveraging HTTP's existing content negotiation mechanism rather than a custom header.

```http
GET /api/users
Accept: application/vnd.myapp.v2+json
```

[↑ Back to top](#table-of-contents)

---

### 5. How do you deprecate an old API version gracefully? 🟡

- Announce the deprecation well in advance, return a `Deprecation` header (and optionally `Sunset` with a removal date) on requests to the old version, keep it functioning throughout a transition window, and monitor usage to know when it's safe to actually remove.

```http
Deprecation: true
Sunset: Wed, 31 Dec 2026 23:59:59 GMT
```

[↑ Back to top](#table-of-contents)

---

### 6. What are the tradeoffs between URL versioning and header versioning? 🔴

- **URL versioning**: simple, visible, cacheable per-version (different URLs cache independently), but "pollutes" the URL and technically means the same resource has multiple different URIs across versions.
- **Header versioning**: keeps URLs clean/stable, arguably more "correct" REST (the resource identity doesn't change, just the representation) — but harder to test/explore manually (can't just paste a URL in a browser), and shared caches need to vary by header, which is less universally supported than varying by URL.

[↑ Back to top](#table-of-contents)

---

### 7. How would you support multiple API versions in the same codebase without massive duplication? 🔴

- Keep the business logic/service layer **version-agnostic**, and isolate version differences to a thin adapter layer (e.g. separate controllers/serializers per version that call the same underlying services) — avoid duplicating actual business logic across versions, only the request/response shaping differs.

[↑ Back to top](#table-of-contents)

---

### 8. How do you communicate breaking changes to API consumers ahead of time? 🔴

- Publish a changelog/migration guide, send advance notice (email, dashboard banners) to registered API consumers, use deprecation headers (Q5) on the affected endpoints, and ideally run the old and new behavior side-by-side for a defined transition period before fully removing the old one.

[↑ Back to top](#table-of-contents)

---

### 9. What's the difference between versioning the whole API vs. versioning individual endpoints? 🔴

- **Whole-API versioning** (`/api/v1/...` globally): simple mental model, but forces a major version bump even if only one endpoint changed, and clients must "upgrade" everything at once.
- **Per-endpoint/resource versioning**: more granular, lets one resource evolve independently of others — more flexible, but harder to reason about overall API "version" as a single concept, and more bookkeeping.

[↑ Back to top](#table-of-contents)

---

### 10. Does semantic versioning apply to REST API versioning? 🔴

- Not directly in the same way as a software library — most REST APIs only expose a **major** version number in the URL/header (`v1`, `v2`), since clients can't "pin" to an exact minor/patch version of a live API the way they pin a library dependency. Minor, backward-compatible changes (new optional fields, new endpoints) typically roll out within the same major version without a version bump at all.

> [!IMPORTANT]
> **Follow-up questions:**
> - Why is adding a new optional field to a response generally considered a non-breaking change, while removing or renaming an existing field is breaking?

[↑ Back to top](#table-of-contents)

---
