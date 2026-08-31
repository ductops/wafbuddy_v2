

# 🛡️ WAFBUDDY_v2 Migration Investigator

> **An evidence-driven, local-first troubleshooting workbench for WAF migrations, proxy/origin failures, HAR comparison, and application-security investigations.**

**WAF Migration Investigator** is a portable browser application designed to help engineers understand what changed when an application moves from one WAF, CDN, reverse proxy, or delivery architecture to another.

Load browser HAR evidence, compare **before vs. after**, investigate individual requests, inspect cache and rate-limit behavior, identify bot/challenge clues, troubleshoot origin connectivity, and generate practical commands for the engineer or customer sitting on the other side of the screen.

No server is required.

No backend is required.

Your investigation stays local.

---

## 🎯 The Problem

A WAF migration rarely fails with a helpful message saying:

> “Your upstream TLS SNI is incorrect.”

Instead, you get:

```text
403
429
502
504
redirect loop
login stopped working
API behaves differently
cache misses everything
CAPTCHA keeps appearing
works directly against origin
fails behind the WAF
```

And then the investigation begins.

The browser sees one part of the path.

The WAF sees another.

The CDN has its own state.

NGINX, IIS, Apache, WordPress, containers, load balancers, and application frameworks each know another piece of the story.

**WAF Migration Investigator brings those fragments together into one evidence-oriented workspace.**

---

# 🔎 What Can It Investigate?

```text
                 WAF Migration Investigator

 Browser / Client
       │
       │ HAR + timing + headers
       ▼
 ┌───────────────┐
 │   WAF / CDN   │
 │               │
 │ Cache         │
 │ Rate Limit    │
 │ Bot / CAPTCHA │
 │ TLS / Headers │
 └───────┬───────┘
         │
         │ proxy / upstream
         ▼
 ┌───────────────┐
 │    Origin     │
 │               │
 │ NGINX         │
 │ IIS           │
 │ Apache        │
 │ WordPress     │
 │ APIs          │
 └───────────────┘
```

The Investigator helps answer questions such as:

* What changed between the legacy and protected application paths?
* Did a previously successful request become a `403`, `429`, `502`, or `504`?
* Is caching behaving differently?
* Are static objects producing HITs after warm-up?
* Is personalized content accidentally cacheable?
* Is rate limiting being enforced?
* Are multiple users possibly collapsing into the same source identity?
* Does the HAR contain bot, automation, CAPTCHA, or browser-challenge evidence?
* Did redirects, hostnames, cookies, or forwarded headers change?
* Is TLS or SNI likely involved?
* Did application latency increase after the WAF was introduced?
* Is the problem actually at the origin rather than the WAF?
* What should the customer run next to prove it?

---

# ⚡ Ten-Second Investigation

The UI is designed around a simple principle:

> **Every screen should answer its primary question in roughly ten seconds.**

Start with:

```text
Load Primary HAR
        +
Load Comparison HAR
        │
        ▼
Before / After
        │
        ├── Errors
        ├── Cache
        ├── Rate limiting
        ├── Bot / challenge
        ├── Timing / p95
        ├── Headers
        ├── Hostnames
        └── Origin clues
```

Then drill into the evidence only where something looks interesting.

---

# 🆚 Before / After HAR Comparison

One of the primary workflows is comparing two browser captures.

For example:

```text
PRIMARY / PROTECTED
Application through new WAF
         ↕
COMPARISON / BEFORE
Legacy WAF or direct/original path
```

The Investigator keeps the two evidence feeds separate and identifies which HAR is supplying information throughout the interface.

Feeds can also be individually enabled or disabled.

This makes it much easier to answer:

> **“What actually changed?”**

rather than simply asking:

> “Does the new environment work?”

### Comparison candidates include

* HTTP status changes
* failed requests
* redirects
* cache HIT / MISS / BYPASS behavior
* static-resource cache effectiveness
* p95 latency
* TTFB
* request duration
* response size
* server headers
* proxy/CDN headers
* hostname changes
* server IP
* rate-limit evidence
* bot/automation signals
* CAPTCHA/challenge behavior
* cookies
* forwarded-header behavior
* origin-related clues

---

# 🧪 Request-Level Investigation

Every interesting request can become its own investigation.

Select a request to inspect:

```text
GET /wp-login.php
HTTP 403
```

The request detail view combines:

* request headers
* response headers
* status
* hostname
* server IP
* content type
* response size
* redirects
* cache evidence
* connection timing
* DNS timing
* TLS timing
* TTFB
* total request time
* WAF/CDN clues
* origin clues
* enforcement evidence

Instead of merely displaying headers, the Investigator attempts to explain **why the evidence matters**.

---

# 🚦 Rate-Limit Investigation

Rate limiting can be particularly confusing during migrations.

The Investigator recognizes client-visible evidence such as:

```text
HTTP 429
Retry-After
RateLimit-Limit
RateLimit-Remaining
RateLimit-Reset
RateLimit-Policy

X-RateLimit-Limit
X-RateLimit-Remaining
X-RateLimit-Reset

X-Rate-Limit-*
```

and provider-specific variants when present.

When enough information exists, the request popup can turn raw headers into something easier to reason about:

```text
RATE LIMIT

Limit       1000
Remaining    995
Consumed     0.5%
Status       Observed
```

Or:

```text
RATE LIMIT ENFORCED

HTTP         429
Remaining      0
Retry-After   30

Confidence: HIGH
```

This helps distinguish:

**“rate-limit metadata exists”**

from:

**“the client was actually throttled.”**

### Migration gotcha

A particularly important investigation is **source identity**.

If forwarded client identity changes during migration, many users behind a proxy or NAT can unexpectedly appear to the WAF as one client.

That can turn:

```text
100 users × 10 requests
```

into what the rate limiter perceives as:

```text
1 source × 1000 requests
```

The Investigator calls attention to this possibility instead of assuming the configured threshold is simply too low.

---

# 🤖 Bot, Automation & CAPTCHA Evidence

HAR files can also contain surprisingly useful clues about bot enforcement.

The Investigator examines combinations of:

* User-Agent
* HTTP status
* bot-related headers
* bot scores
* redirects
* challenge URLs
* challenge cookies
* clearance cookies
* response content
* CAPTCHA markers
* browser-verification pages
* automation clients
* challenge follow-up requests

Possible classifications include:

```text
Bot detection / automation signal

Bot prevention likely

Challenge / CAPTCHA likely
```

These are intentionally presented as **evidence-based assessments**, not absolute declarations.

A HAR may prove that the browser received a challenge.

It normally cannot prove which internal WAF rule generated it.

That requires correlation with security telemetry.

---

# 🧭 Provider Insight Packs

The core Investigator remains vendor-neutral.

Provider-specific interpretation is applied as an optional **insight lens**.

Current selections include:

```text
Vendor Neutral
Check Point WAF
AWS WAF / CloudFront
Azure WAF / Front Door
Cloudflare
F5
```

Changing the provider does **not** change the HAR evidence.

It changes how that evidence is interpreted and which troubleshooting questions are suggested.

That makes the same Investigator useful on both sides of a migration.

For example:

```text
Cloudflare
    ↓
Check Point WAF

AWS WAF / CloudFront
    ↓
Check Point WAF

F5
    ↓
Check Point WAF
```

The engineer can first understand the legacy behavior and then switch lenses while validating the destination architecture.

---

# 🧱 Origin & Backend Troubleshooting

A browser HAR only shows the browser-facing portion of the transaction.

Many WAF problems happen here:

```text
Browser
   │
   ▼
 WAF
   │
   │  ← mystery lives here
   ▼
Origin
```

The Investigator therefore includes an **Origin Investigation** workflow covering common failure points such as:

* upstream hostname
* origin URL
* TCP port
* DNS resolution
* TLS
* certificate chain
* certificate expiration
* SAN/CN
* SNI
* Host header
* source-IP allowlisting
* reverse proxy configuration
* application listener
* health checks
* forwarded headers
* upstream response time
* timeout behavior
* APIs and application paths

This becomes especially useful for:

```text
502 Bad Gateway
504 Gateway Timeout
```

because those errors frequently indicate that the browser can reach the WAF while the WAF cannot successfully complete the upstream transaction.

---

# 🖥️ Server Troubleshooting Guides

Migration runbooks include practical guidance for common backend platforms.

Current focus includes:

### IIS / ASP.NET

* bindings
* application pools
* URL Rewrite
* forwarded headers
* authentication
* certificates
* Windows networking
* IIS logs
* PowerShell diagnostics

The Investigator distinguishes where possible between commands appropriate for:

```text
Windows Server + IIS
```

and commands that can safely be executed from:

```text
Windows Desktop / Engineer Workstation
```

### NGINX

Useful investigation areas include:

* `proxy_pass`
* upstream configuration
* Host preservation
* `X-Forwarded-For`
* `X-Forwarded-Proto`
* TLS/SNI
* access logs
* error logs
* timeout configuration
* header size
* body size
* upstream response timing

### Apache

Guidance includes common proxy, TLS, header, virtual-host, and logging checks.

### WordPress

Special attention is given to migration-sensitive behavior such as:

* canonical hostname
* HTTP/HTTPS awareness
* redirects
* login
* cookies
* admin paths
* proxy headers
* cache behavior
* dynamic/personalized content

---

# ⌁ Domain Quick Probe

Sometimes the engineer doesn't have a HAR.

Sometimes the customer cannot send one.

Sometimes you just need:

> **“Run this and send me the output.”**

The **Domain Quick Probe** generates portable troubleshooting commands for:

* cross-platform `curl`
* Windows PowerShell
* Linux shell environments

The commands gather evidence around:

```text
DNS
TCP
TLS
HTTP status
redirects
headers
cache
timing
remote IP
certificate
SNI
```

Returned output can be pasted directly back into the Investigator for analysis.

This creates another evidence path:

```text
Customer
   │
   │ copy command
   ▼
Terminal
   │
   │ paste output
   ▼
WAF Migration Investigator
   │
   ▼
Evidence + recommendations
```

---

# 🧊 Cache Investigation

Caching gets its own visual investigation because raw cache headers are easy to misinterpret.

The Investigator distinguishes concepts including:

### HIT

The cache served an existing object.

Often desirable for:

* images
* CSS
* JavaScript
* fonts
* public downloads
* versioned static resources

### MISS

The cache did not have a usable object and fetched upstream.

A first request being a MISS can be completely normal.

Repeated identical cacheable requests remaining MISSes are much more interesting.

### BYPASS

The cache intentionally avoided storing or serving the object.

Often appropriate for:

* login
* account
* cart
* checkout
* payment
* admin
* tokens
* authenticated APIs
* personalized responses

### p95

Instead of allowing averages to hide ugly outliers, the Investigator exposes **95th-percentile latency**.

If p95 is `1.8 s`:

> 95% of observed requests completed within approximately 1.8 seconds; the slowest 5% took longer.

That often reveals migration pain that average latency hides.

---

# 🧬 Header Investigation

Headers frequently tell the migration story.

The Investigator highlights useful signals around:

```text
Host
Forwarded
X-Forwarded-For
X-Forwarded-Host
X-Forwarded-Proto
Via
Server
Cache-Control
Age
X-Cache
Location
Set-Cookie
Authorization
Content-Type
Retry-After
RateLimit-*
Server-Timing
```

These can expose problems involving:

* wrong canonical hostname
* lost client identity
* HTTP/HTTPS confusion
* proxy-awareness
* redirect loops
* authentication
* cookie scope
* caching
* rate limiting
* CDN behavior
* origin identity

---

# 🧠 GenAI Security Investigation

The Investigator is also being designed for modern application-security scenarios where WAF functionality extends beyond traditional HTTP attacks.

The GenAI investigation area provides room for evidence and guidance involving:

* prompt injection
* sensitive-data leakage
* AI/API usage controls
* unsafe input
* unsafe output
* model/API endpoints
* authentication
* rate limiting
* bot/automation activity
* AI endpoint caching risks
* security policy enforcement

This area is intentionally designed to evolve as AI application-security controls mature.

---

# 🗃️ Session Vault

Investigations can become long-lived.

The built-in **Session Vault** allows an engineer to:

```text
Save
Load
Edit
Delete
```

named investigation sessions.

A session can preserve items such as:

* Primary HAR
* Comparison HAR
* feed state
* insight provider
* origin evidence
* domain-probe evidence
* server-guide selection
* AI-security state
* active workspace

Resetting the current workspace does not erase saved investigations.

---

# 📦 Portable Investigation Bundles

Sessions can also leave the browser.

Use:

```text
Export Session
Import Session
```

to create a portable:

```text
*.wmi-session.json
```

bundle.

The bundle can contain the investigation state and HAR evidence needed to reopen the case on another machine or browser.

Conceptually:

```text
Capture the case
      ↓
Save the case
      ↓
Export the case
      ↓
Move the case
      ↓
Import the case
      ↓
Continue investigating
```

The bundle format is versioned so future Investigator releases can evolve while retaining a migration path for older cases.

---

# 🔐 Local-First by Design

WAF troubleshooting evidence can contain sensitive information.

HAR files may include:

* cookies
* session identifiers
* authorization headers
* tokens
* internal hostnames
* API paths
* query parameters
* request bodies
* customer data

For that reason, the Investigator is designed as a **local browser application**.

```text
HAR
 │
 ▼
Browser
 │
 ▼
Investigator
 │
 └──── analysis remains local
```

No cloud backend is required for the core workflow.

### ⚠️ Treat exported investigation bundles as sensitive

A portable session may contain the original HAR evidence.

Handle exported bundles according to the same security requirements you would apply to the source HAR files.

---

# 🧭 Evidence Boundary

One of the most important concepts in the project:

> **Know what the evidence can prove.**

A HAR can provide excellent evidence about:

* browser requests
* browser responses
* HTTP status
* redirects
* timing
* headers
* cookies
* cache behavior
* challenge flows
* client-visible WAF/CDN behavior

A HAR normally **cannot prove**:

* WAF-to-origin routing
* private DNS resolution
* backend firewall decisions
* source-IP allowlists
* private certificate trust
* origin-side SNI behavior
* backend application logs
* internal WAF rule evaluation

The Investigator deliberately calls out that boundary.

The goal is not to pretend the HAR knows everything.

The goal is to use what it **does** know to determine the smartest next test.

---

# 🧰 Migration Workflow

A practical investigation often looks like this:

```text
1. Capture legacy/origin HAR
             │
2. Capture protected/new-WAF HAR
             │
3. Load both into Investigator
             │
4. Review Before / After
             │
5. Find meaningful deltas
             │
6. Drill into suspicious requests
             │
7. Inspect cache/rate/bot/header evidence
             │
8. Run Domain / Origin probes
             │
9. Correlate with WAF + backend logs
             │
10. Fix configuration
             │
11. Capture again
             │
12. Prove the delta disappeared
```

That last step matters.

A troubleshooting tool should not merely help find the problem.

It should help **prove the fix**.

---

# 🧪 Synthetic Demo

Don't have customer evidence yet?

Use:

**Load Full Demo**

The built-in synthetic investigation demonstrates examples involving:

* protected and legacy HARs
* origin evidence
* cache HIT/MISS
* rate limiting
* bot activity
* CAPTCHA/challenge behavior
* redirects
* errors
* server/header differences
* GenAI security scenarios

This makes the Investigator useful for:

* learning
* demos
* workshops
* migration planning
* troubleshooting practice
* customer conversations

without exposing production traffic.

---

# 🏗️ Deployment Philosophy

The current application is intentionally simple:

```text
index.html
```

Open it in a modern browser.

That's the deployment.

The project favors:

* portability
* local execution
* low dependencies
* easy customer use
* visual investigation
* evidence over assumptions
* copy/paste diagnostics
* teachable troubleshooting
* progressive disclosure of complexity

---

# 🛡️ Security Philosophy

WAF Migration Investigator follows a simple troubleshooting principle:

> **Observed evidence and inferred conclusions are not the same thing.**

For example:

```text
Observed:
HTTP 429
Retry-After: 30

Strong inference:
Some form of throttling is occurring.

Not yet proven:
Which rule fired.
Which product generated it.
Which identity was counted.
Why the threshold was exceeded.
```

The interface tries to preserve that distinction.

That makes recommendations more defensible and reduces the temptation to blame whichever component was most recently changed.

---

# 🚧 Project Direction

Areas being explored for future releases include:

* richer migration replay
* deeper request classification
* automated evidence correlation
* investigation findings/notebook
* enhanced origin diagnostics
* additional provider insight packs
* architecture recommendations
* configuration evidence
* expanded AI-security investigation
* migration validation reports
* portable investigation packages
* investigation export/reporting

The guiding question remains:

> **Where is this actually failing?**

---

# 🤝 Contributing

Ideas, troubleshooting patterns, additional WAF/CDN evidence signatures, server-specific commands, provider mappings, and real-world migration lessons are welcome.

Particularly useful contributions include cases where:

```text
Symptom
   ↓
Evidence
   ↓
Investigation
   ↓
Root Cause
   ↓
Fix
```

can be turned into a reusable troubleshooting pattern for the next engineer.

---

## Final Thought

WAF migrations live at the intersection of networking, HTTP, TLS, proxies, security policy, application behavior, caching, identity, and increasingly AI security.

No single log tells the entire story.

**WAF Migration Investigator exists to make the fragments understandable.**

> **Discover the evidence.
> Compare the paths.
> Explain what changed.
> Find the boundary.
> Prove the fix.**
