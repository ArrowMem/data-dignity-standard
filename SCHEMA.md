# The declaration file - format v0.5

*[Version française](SCHEMA.fr.md)*

Defines the grammar every clause in the standard tests against. Kept deliberately boring: the
llms.txt lesson is that plain JSON at a fixed, guessable path beats anything clever.

## Location

```
https://<site>/.well-known/data-dignity.json
```

The `/.well-known/` prefix is the IETF-standardized location for origin-level metadata
(RFC 8615) - `security.txt` lives there, so does the pattern this borrows. `data-dignity.json`
is this standard's name within that namespace; it is not yet an IETF-registered well-known
suffix, and registering it under RFC 8615's process is planned once the format stabilizes.

**Scope: a declaration covers exactly the origin that serves it, and nothing else.** An origin
is the full tuple the web defines it as (RFC 6454): scheme, canonical host, effective port -
never a string prefix, never a hostname suffix. The file at
`https://example.com/.well-known/data-dignity.json` speaks for `example.com` only - not
`api.example.com`, not `shop.example.com`, not any other host. Each origin that handles
agent-delivered data publishes its own declaration. A checker MUST NOT apply one origin's
declaration to another. **The same rule holds mid-session: when an agent follows a navigation,
redirect or API transition to a new origin, the new origin is outside the prior declaration
unless it separately declares** - a transaction that starts on a covered origin does not carry
that coverage to the origins it lands on.

## Version security

A format version is a security contract, and accepting an old contract forever is a downgrade
channel: a site that prefers weaker scrutiny would simply publish the oldest accepted format
and keep every protection added since off its own back. So the rule is explicit:

- **A checker enforces a minimum graded version as policy.** A declaration older than that
  minimum MAY be parsed and displayed for information, but it MUST NOT receive a
  current-version grade on any clause or composite - it is reported as **LEGACY**, which is
  neither a letter grade nor "undeclared": it means *declared under an obsolete security
  contract*.
- **Version negotiation MUST NOT silently downgrade security requirements.** A checker never
  applies an older format's weaker rules and then reports the result on the current scale.
- Older formats sunset: once a version's successor has been published for a defined comment
  period, the older version leaves the graded set. The current graded set is exactly:
  **v0.5**.

## Strict serialization

The declaration is a security-policy object, so how it parses is itself a security property -
the nastiest implementation class is parser differential: a validator reading one meaning, a
checker a second, an agent acting on the most privileged third. Normative rules:

- A declaration MUST be valid UTF-8-encoded JSON. Malformed encoding is rejection, not repair.
- **Duplicate object member names MUST cause rejection.** RFC 8259 warns that duplicate names
  make behavior implementation-defined - in this format that is a policy-confusion attack
  (`"receipt_support": false` and `"receipt_support": true` in one object), so it is an error,
  never a choice.
- **Unknown members in the core grammar MUST cause rejection.** Everything site-specific goes
  under the single `extensions` member, whose contents a consumer MUST ignore when it does not
  recognize them. Strict core, explicit extensions - never "anything anywhere".
- Implementations MUST enforce bounded total size, bounded nesting depth, bounded string
  lengths and bounded array lengths - a declaration with ten thousand third parties or
  legal holds is an attack on the checker, not a disclosure - and MUST parse
  deterministically: no parser-dependent coercion, no acceptance of almost-JSON.
- Every declaration value is attacker-controlled data and MUST NOT be interpreted as
  executable instructions by any consumer - checker, agent, or anything downstream.

## Shape

This is an illustrative example, not any real organization's actual declaration:

```json
{
  "version": "0.5",
  "declared": "2026-01-01T00:00:00Z",
  "expires": "2026-07-01T00:00:00Z",
  "last_reviewed": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": "90d",
    "network_metadata": "30d",
    "notes": "Identity is support tickets, kept 90 days to resolve disputes. Nothing else is stored."
  },
  "custom_classes": {
    "loyalty-profile": ["derived", "identity"]
  },
  "third_parties": [
    { "name": "Example CDN Co", "identifier": "example-cdn.example", "role": "infrastructure",
      "recipient": "direct", "purpose": "content delivery", "data_seen": ["network_metadata"],
      "jurisdiction": ["example-region"] },
    { "name": "Example Model Host", "identifier": "models.example", "role": "model_provider",
      "recipient": "downstream", "via": "example-cdn.example", "purpose": "inference",
      "data_seen": ["content"], "jurisdiction": ["example-region"] }
  ],
  "jurisdiction": {
    "processing": ["example-region"],
    "storage": ["example-region"],
    "backups": ["example-region"],
    "legal": ["example-region"]
  },
  "deletion": {
    "handle": "https://example.com/api/agent-delete",
    "method": "POST",
    "auth": "receipt-token",
    "covers": ["identity", "network_metadata"],
    "legal_holds": [
      { "covers": "consent records", "purpose": "proving consent was given",
        "legal_basis": "applicable-privacy-law", "review_by": "2027-01-01T00:00:00Z",
        "reason": "consent must remain provable under applicable privacy law" }
    ]
  },
  "receipt_support": false,
  "extensions": {}
}
```

### Data classes

Free-text descriptions of data invite radically different readings of the same declaration, so
`data_seen`, `retention` keys and `deletion.covers` draw from one **closed** vocabulary:

`content` (what the interaction produced - prompts, messages, uploads), `identity` (who the
interaction was with), `authentication` (tokens, credentials), `network_metadata` (IP,
headers), `device_metadata` (fingerprint, user agent), `financial`, `health`, `location`,
`files`, `derived` (anything computed from the above - embeddings, scores, profiles).

**Custom classes are additional to the standard vocabulary, never a substitute for it.** A
declaration MAY name site-specific classes, but each one MUST be mapped to one or more standard
classes in `custom_classes`, and a declaration MUST NOT use a custom class to avoid
classification under a standard class it plainly fits. A vague label ("operational telemetry",
"risk intelligence") with no mapping is a semantic evasion, and a checker grades any unmapped
custom class as undeclared - mechanically, not by human judgement. **The mapping must also
preserve sensitivity**: a custom class whose data plainly fits `health`, `financial`,
`location` or `authentication` MUST map to that class - mapping it only to a broader,
blander one (`derived`, `content`) to dilute what it is, is the same evasion one level up,
and graded the same way. Two more invariants close the remaining ambiguity:
**a custom class name MUST NOT equal any standard class name** - a custom class called
`health` mapped to `derived` would give one token two meanings, which is the exact
parser-confusion class this format exists to kill, so the schema rejects the collision
outright (namespaced names like `example.com/loyalty-profile` are the recommended shape) -
and **when a custom class maps to multiple standard classes, every mapped class's rules
apply simultaneously**: retention, deletion, jurisdiction and disclosure obligations all
attach, the strictest reading governs, and a declaration never chooses the blandest mapped
class as the governing one. "Agent-delivered data" means every class above that an agent's
interaction causes the site to receive - including metadata the agent did not deliberately
send.

### Field notes

- `declared` / `expires` / `last_reviewed` - a declaration is a claim about now, not forever.
  `expires` is REQUIRED: an expired declaration is graded as stale, not as current truth,
  because a site can change its entire infrastructure long after publishing a clean
  declaration. The timeline is checked as arithmetic, not just as syntax: **`declared` must
  precede `expires`, `expires` must fall within twelve months of `declared`, `last_reviewed`
  must not postdate `expires`, and `declared` must not sit in the future** beyond ordinary
  clock skew - a syntactically valid date from 2099 is a freshness bypass, and a declaration
  violating any of these is graded stale, exactly as if it had expired.
- `retention` - **a machine-checkable map from data class to retention duration**: `none`,
  `indefinite`, or a count of days/months/years (`90d`, `6m`, `2y`). `content` is always
  required. **Coverage rule: every class that appears anywhere in the declaration - in any
  third party's `data_seen`, in `deletion.covers`, or via a custom-class mapping - MUST have a
  retention entry**, so a checker can verify mechanically that nothing seen is unaccounted for,
  with no interpretation required. Most real sites will have SOME identity-adjacent retention
  for legitimate reasons (support, fraud prevention, legal requirements) even when they store
  zero content - per-class honesty is rewarded over a blanket "we retain nothing" that a check
  would falsify. `notes` stays free-text for the human explanation.
- `third_parties` - **the processing graph, not just the first hop.** Disclosure is
  transitive: a declaration covers every downstream recipient known or contractually permitted
  to receive agent-delivered data through a declared processor, subprocessor, model provider,
  infrastructure provider or other intermediary - "we send it to A" is not honest when A
  forwards it to C, and "A sees it" while hiding C behind A is the loophole this rule closes.
  Each entry carries a REQUIRED `recipient` position (`direct` or `downstream`) and a
  REQUIRED `role` (`processor`, `subprocessor`, `infrastructure`, `analytics`, `security`,
  `model_provider`) - a party whose position or kind is unstated leaves the graph
  incomplete, and an incomplete graph is graded as one. The graph itself obeys hard
  invariants: **every `identifier` is unique within the declaration; `via` is REQUIRED on a
  downstream recipient and MUST equal exactly one declared party's identifier; `via` is
  forbidden on a direct recipient; and the declared graph MUST be finite and acyclic** - a
  cycle is not a disclosure, it is an invalid declaration graded F, and bounded node counts
  keep a hostile graph from becoming a traversal attack on the checker. Identifiers compare
  canonicalized (a lowercase DNS name or canonical origin, never mixed freely with URL
  variants of itself). The chain MUST terminate at declared boundaries - no declared party
  may be a door to undeclared ones. **`identifier` is REQUIRED and is the machine-readable
  identity - a stable domain or operator-controlled URI; `name` is presentation only.**
  Corporate families and lookalike names ("Example AI", "Example AI Analytics", an acquired
  vendor) make a bare name unverifiable; an identifier is what a checker can actually match
  observed traffic against. "Sees" includes server-side forwarding that no outside network
  observer can watch. An empty array is a valid, gradeable answer.
- `jurisdiction` - where the data is processed, stored and backed up, as separate answers,
  because they are usually different places. Region codes preferred where independently
  checkable. Legal jurisdiction (whose law applies) may differ from physical location; where
  they differ, say both.
- `deletion` - a real, callable handle. The whole point is machine-callable, not a support
  ticket paragraph. **`deletion.handle` MUST have the same origin as the declaration that
  advertises it** - the full origin tuple, compared after canonicalization (lowercase,
  IDNA/punycode-normalized host, no trailing dot, no userinfo, effective port 443), never by
  string prefix. A cross-origin deletion handle is the cleanest capability-exfiltration path
  in this format - an agent that later exercises the declared path would carry its deletion
  token to whatever origin the declaration named - so it is not a style rule, it is the
  boundary. A checker grades a cross-origin handle F on this clause. The declaration must also
  say how a caller proves it is entitled to delete what it asks to delete (`auth`), because an
  unauthenticated deletion endpoint is a weapon pointed at the site's own users. Requirements
  on the endpoint itself:
  - it MUST require evidence tying the request to the interaction being deleted (a receipt
    token where receipts exist, or an equivalent per-interaction secret) - never delete on an
    unauthenticated request;
  - **the token travels in a request header or body, never in a URL query parameter** - query
    strings leak through access logs, proxy logs, analytics, browser history, referrer
    propagation and monitoring, and a leaked deletion token is a leaked capability;
  - it MUST be idempotent - the same deletion requested twice is not an error;
  - it MUST NOT reveal whether a token is valid other than by processing an authorized
    request - uniform error responses and uniform timing, because a validity oracle lets an
    attacker confirm a stolen token before spending it;
  - its success response MUST say what actually happened: `deleted`, `queued` (with an
    expected completion time), or `nothing-held` - a bare HTTP 200 is not an answer;
  - anything it cannot delete must appear in `legal_holds` - and **a legal hold is bounded,
    never a magic word**: each entry names the specific records it covers, the specific
    purpose, a `legal_basis` identifier, and a `review_by` date (or expiry) after which the
    hold must be re-justified. A blanket or indefinite hold with no stated scope and no
    review condition is invalid; a hold claiming to cover every class is a red flag graded as
    such. The standard does not judge whether the cited law is right - it prevents "required
    by applicable law" from excusing everything forever;
  - it SHOULD rate-limit and monitor for abuse - authenticated deletion is still a write
    endpoint an adversary can hammer;
  - **deletion is complete or it says why not**: every class the declaration retains (any
    retention entry other than `none`) MUST appear in `deletion.covers` or in a legal
    hold - a "working deletion path" that quietly excludes the indefinitely-retained class
    is the composition bug between clause 1 and clause 4, and it grades as undeclared for
    what it excludes;
  - `queued` is a promise with a date, not a parking lot: `expected_by` MUST be present and
    MUST NOT sit more than thirty days out, and the final state MUST be queryable - a queue
    that can be re-queued forever is a deletion path that never deletes.

  The minimal request/response shape, normative so implementations converge and probes have
  one surface to test rather than many:

  ```json
  POST /api/agent-delete
  { "interaction_id": "an-opaque-id", "deletion_token": "an-opaque-single-use-token" }

  200 { "status": "deleted", "covers": ["identity", "network_metadata"], "completed": "2026-01-02T00:00:00Z" }
  200 { "status": "queued", "expected_by": "2026-01-09T00:00:00Z" }
  200 { "status": "nothing-held" }
  401 { "status": "unauthorized" }
  410 { "status": "token-expired-or-used" }
  ```
- `receipt_support` - whether the site supports per-interaction receipts. `false` is an honest,
  gradeable answer, not a failure by itself. Where supported, a receipt is a small JSON object:

  ```json
  {
    "interaction_id": "an-opaque-id",
    "received_classes": ["content", "identity"],
    "retention": { "content": "none", "identity": "30d" },
    "third_parties": ["example-cdn.example"],
    "deletion_token": "an-opaque-single-use-token"
  }
  ```

  A receipt travels as an HTTPS response body - never a cookie, whose ambient-transmission
  semantics are exactly wrong for a capability - and **a response carrying a live deletion
  capability MUST be explicitly non-cacheable (`Cache-Control: no-store`) and MUST NOT be
  stored by any intermediary**, or the receipt-to-CDN-to-proxy chain becomes a capability
  leak no token binding can fully contain. **A receipt MUST NOT echo the content it
  describes.** It names data classes and handling
  state, never the words, files or values themselves - a receipt that restates a health
  question is a fresh copy of the most sensitive thing in the interaction. **A receipt's
  `interaction_id` MUST identify one interaction unambiguously within its origin** -
  conceptually bound to the origin, an interaction nonce and a timestamp - so an agent's
  ledger entry and a site's receipt can be reconciled and neither side can point one receipt
  at a different interaction after the fact. Until receipts are signed, a receipt is itself a
  *claim* at the declared evidence level - it does not prove the site generated it for that
  interaction, and a checker MUST NOT label receipt contents higher than declared. The signed
  receipt (a signature over the canonical interaction record) is where the `signature`
  extension below earns its place. **A receipt carrying a `deletion_token` is carrying a live
  capability, and both sides treat it that way**: the site keeps the token out of its own logs
  and caches, and the agent stores it in its protected capability store (see the ledger rules
  in the main document) - **never in plain exports, never in logs, crash reports or
  analytics, and never in a model's prompt context**, because a token that has passed through
  a language model's context has been disclosed to everything that context reaches. The
  issuance path is the weak link the binding rules exist for: intermediaries that cache
  receipts, framework middleware that logs them, extensions that read them are all named
  risks the token's single-use, short-lived binding contains but does not erase.
- `extensions` - the one home for anything site-specific beyond `custom_classes`. Namespaced
  keys (`"vendor.example/feature"`), contents ignored by consumers that do not recognize
  them, never a place to restate or contradict a core member.
- `signature` - reserved, and **v0.5 consumers MUST ignore it entirely**: no checker, agent
  or intermediary may treat its presence as evidence of anything, because no interoperable
  profile exists yet and a homemade cryptographic interpretation shipped early is how crypto
  disasters start. When the profile is defined - only after independent cryptographic
  review - HTTP Message Signatures (RFC 9421) is the candidate mechanism and JSON
  Canonicalization (RFC 8785) the candidate serialization, rather than invented rules.
  Until then HTTPS transport is the trust model, stated plainly as a limitation rather than
  dressed up as a guarantee.

## Capability binding and the replay model

One principle closes a whole family of composition attacks, so it is stated once and applies
everywhere: **every sensitive capability this ecosystem mints - a deletion token, a receipt
token, a future bulk-operation authorization, a future signature - MUST be bound to its
ORIGIN, its INTERACTION, its PURPOSE, an EXPIRY, and a NONCE.** A capability that any holder
can spend anywhere, on anything, forever, is a bearer credential waiting to leak across a
layer boundary; the IETF's sender-constrained-token work (RFC 9449) documents exactly this
failure mode. Applied here:

- a deletion token is valid only at the origin that issued it, only for the interaction it
  names, only for deletion, **only for the exact classes and operation it was issued for** -
  a token minted to delete one interaction's content must be unusable as "delete the
  account" - single-use, and short-lived, and non-exportable where the platform can enforce
  that;
- **the replay model is formal, not implied**: a capability or receipt carries its
  interaction nonce, issue time, expiry and audience (the origin it is for), and MUST NOT be
  accepted outside that exact scope or validity window - by the site, by the agent, or by
  anything replaying traffic between them. Implementations do not invent their own replay
  semantics;
- possession of a token is never authorization by itself - an agent holding a ledger full of
  deletion tokens has the *means* to delete, and still needs the person's explicit
  authorization to *act* (see the agent-side rules in the main document);
- the concrete token construction is deliberately unprofiled here, pending the same
  independent cryptographic review as the signature extension - the binding requirements are
  normative now, the format is not.

## Rules for checkers

The checker is an HTTP client processing input an adversary controls, and a hostile
declaration is the obvious way to attack it. Any conforming checker:

- MUST NOT dereference `deletion.handle` unsolicited. Presence and shape are checkable from
  the declaration alone; actually calling a deletion endpoint happens only with the site's
  consent and synthetic data, or not at all.
- MUST refuse any declared URL that resolves to loopback, private, link-local, multicast or
  otherwise reserved address space. **Resolution is where this defense is usually lost, so the
  requirement is precise: every resolved address MUST be validated - every A and every AAAA
  record, after any CNAME chain, including IPv4-mapped IPv6 forms - and the connection MUST be
  made to an address that was validated, without a fresh resolver lookup between validation
  and connection that could change the destination. If resolution is ambiguous, split between
  safe and unsafe addresses, or changes during the check, the request fails closed.**
- MUST compare origins as origins - scheme, canonical host, effective port, per RFC 6454 and
  RFC 3986's normalization rules - handling default ports, IPv6 literals, IDN/punycode forms,
  trailing-dot hostnames and userinfo explicitly, and MUST NOT compare origins by string
  prefix or hostname suffix.
- MUST NOT follow a redirect to a different origin than the one it is checking, and MUST
  bound the WHOLE fetch, not each hop: a maximum redirect count, a maximum total duration
  and a maximum total byte budget across every hop together - thirty same-origin redirects
  of ten megabytes each is a resource attack that per-response limits never see.
- MUST enforce strict response-size and timeout limits on everything it fetches, applied to
  the DECODED representation as well as bytes on the wire - a small compressed body that
  inflates a thousandfold is a decompression bomb, so both sizes and the expansion ratio
  are bounded.
- MUST use an explicitly controlled egress path: **ambient proxy configuration
  (`HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`), proxy credentials and any other network
  authority inherited from the execution environment are ignored**, because a poisoned
  proxy variable re-routes a perfectly validated destination through an attacker - the
  networking layer beneath the SSRF defense is part of the SSRF defense.
- MUST NOT attach credentials, cookies or ambient authority to any request a declaration
  caused it to make.
- MUST enforce the serialization rules above (UTF-8, duplicate-name rejection, unknown-core
  rejection, bounds) with a single deterministic parser - and treat **every declaration value
  as attacker-controlled data, never as executable instructions - in reports, and equally in
  anything downstream**: a `name`, `purpose`, `reason` or custom class may end up in a log
  line, a shell argument, a query, a URL, a spreadsheet export or a metrics label, and it is
  untrusted text in every one of those places, not just in rendered HTML.
- MUST record the provenance of what it graded: fetched from the origin or served by a cache
  (an intermediary can serve a stale or substituted file), signed or unsigned - and MUST NOT
  present an unsigned, cache-served declaration as equivalent evidence to an
  origin-authenticated one.
- MUST keep its evidence deterministic underneath its grades: **every individual test result
  is exactly one of PASS, FAIL, UNKNOWN or NOT_TESTED, and grades are computed from those
  facts by the published scoring rule.** Judgement may decide what to test; it never decides
  what a recorded fact says. Two checkers with the same facts reach the same grade, and
  incomplete evidence shows up as UNKNOWN/NOT_TESTED facts, never as optimistic letters.
  Every recorded fact carries its evidence window - when it was observed, by which checker
  and test-profile version - because a verified A is a claim about a bounded window, never a
  timeless property: a site graded in August can change in September, and a grade that
  outlives its evidence is the false comfort this standard exists to end.
- MUST score transport-observed third parties and server-declared processors as separate
  facts, never merged into one "verified third parties" claim - the first is evidence, the
  second is a declaration, and an attacker defeats the strongest empirical test by keeping
  the dangerous behavior entirely server-side.
- MUST state its observation surface when it reports observed evidence: observed at the
  client, on the transport, at the origin, from a server-side disclosure, or by an external
  monitor. Two checkers watching different surfaces see different traffic; an "observed" claim
  that does not say where it observed is not comparable, and grades that disagree across
  surfaces are a finding, not a contradiction. **A structural ceiling follows and is stated so
  no grade overclaims: purely server-side behavior - a subprocessor relationship, an internal
  forward - can never reach "verified" from external observation alone.** Those claims stay at
  "declared" until something stronger exists (a signed attestation, a transparency report, a
  legal process), and a checker MUST NOT label them higher.
- SHOULD, when running retention probes, make its synthetic traffic genuinely hard to
  fingerprint - randomized payloads, realistic timing, varied vantage points and networks,
  periodically varied test choreography - and SHOULD re-check for reappearance at spaced
  intervals (immediately, after a day, after a week or more) before concluding "not
  reappeared". **Active testing is classified as black-box behavioral evidence, never as
  verification of internal state**: a sophisticated site can fingerprint test *behavior* even
  when it cannot recognize test *data*, so a clean probe is evidence the site behaved for
  this probe, from this vantage point, in this window - the honest claim, and the only one a
  probe can make. The full canary methodology is its own document on the roadmap; these
  floors hold in the meantime.
- MUST apply the version-security rules above to earlier-format files: parse and display,
  report LEGACY, never grade under superseded rules on the current scale.

One rule reaches past the checker to every agent consuming this format: **no server
response, declaration, receipt or site-provided text can grant itself authority** - site
data never becomes agent instructions, and a page announcing that "policy requires" the
agent to disclose more, delete something, or change its security posture is content to be
displayed, never a directive to be followed.

Sites are held to the mirror rule: **a site MUST NOT special-case traffic it believes to be a
conformance test.** Retention tests use randomized, unmarked synthetic data precisely so that
recognizing and suppressing the test is harder than simply behaving; a site caught doing it is
graded as declared-and-false, the worst grade on the sheet.

**A limit stated so nobody reads past it: reappearance testing does not establish absence of
transformed, inferred, aggregated, hashed, embedded, classified or otherwise derived
representations.** A site can delete the words and keep an embedding, a classification, a risk
score, a profile vector - the exact text never reappears, and something derived from it is
retained all the same. That is why `derived` is its own declaration class with its own
retention entry and its own separately-scored facts, why recognition-style probes (does the
site *behave* as if it remembers) are part of the verification roadmap alongside exact replay,
and why a clean reappearance test is evidence for `content`, never for `derived`.

## The dignity threat model

The adversaries this format is designed against, named so every rule can be traced to the
attacker it stops and every implementation can test against the same list:

- **T1 - dishonest site**: declares false things. Countered by probes, the
  declared-and-false-caps-composite rule, and evidence labels.
- **T2 - malicious declaration**: the file itself as attack input. Countered by strict
  serialization, SSRF rules, same-origin deletion, untrusted-values rule.
- **T3 - compromised CDN or cache**: serves a stale or substituted declaration. Countered by
  provenance recording, `expires`, and eventually the signature extension.
- **T4 - malicious or leaky processor**: a declared party forwarding onward. Countered by
  transitive disclosure and the declared-boundary rule.
- **T5 - compromised agent**: the user's own tool turned. Countered by the ledger split,
  capability binding, and bulk-deletion authorization.
- **T6 - malicious agent plugin or injected prompt**: authorization forgery from inside.
  Countered by possession-is-not-authorization and the explicit bulk-deletion ceremony.
- **T7 - network attacker**: interception and replay. Countered by HTTPS, the replay model,
  and capability expiry.
- **T8 - compromised ledger**: the disclosure history as loot. Countered by the
  audit/capability split and no-content rule.
- **T9 - test-aware site**: behaves only for the auditor. Countered by unmarked randomized
  canaries, varied choreography, and the black-box-evidence classification.
- **T10 - parser/implementation differential**: three components reading three meanings.
  Countered by strict serialization, the closed core, and the deterministic facts model.

## The composition attacks this format is built against

Each rule above closes a single hole; this section names the chains, because every step below
is individually permitted somewhere and the chain is the attack. The canonical one - the
dignity downgrade chain: a site publishes a valid-looking declaration; hides meaningful
categories behind unmapped custom classes; declares a cross-origin deletion endpoint; issues
receipts whose deletion tokens the agent dutifully stores; a later autonomous cleanup run
follows the declared path and carries the token to the attacker's origin; the attacker now
holds a live capability, and the agent's own ledger holds the full disclosure history for a
second-stage compromise. The custom-class mapping rule, the same-origin deletion MUST,
capability binding, the bulk-authorization rule and the ledger split each cut one link of that
chain; together they cut all of them. The other standing chains: staying on an obsolete
format version to dodge every rule added since (closed by version security), CDN-shared
infrastructure making two origins look like one (closed by exact-origin scope and the
mid-session transition rule), hiding a recipient behind a declared aggregator (closed by
transitive disclosure), a checker turned into a weapon through the URLs and values a
declaration feeds it (closed by the checker rules above), and **trust laundering**: a
mediocre origin routing data through a processor with an excellent grade, so the agent
reads the intermediary's A as covering data it received transitively - it does not; a grade
speaks for a party's own declared conduct, never for another origin's data flowing through
it, and per-flow data lineage (the roadmap's answer) is the only thing that ever will.

## Open questions

- The concrete signature profile and the concrete deletion-token construction (see above) -
  both defined as extension points with normative binding requirements, deliberately
  unprofiled pending independent cryptographic review.
- Versioning and migration. v0.5 is a breaking change to v0.4's published grammar
  (`role` and `recipient` became required on every third party, custom-class names colliding
  with standard classes are rejected, and the `via` cross-field rules became schema-level) -
  versioned rather than mutated in place, exactly what the version-security section above
  demands of everyone else. Earlier formats are handled by those same rules: parsed,
  displayed, reported LEGACY, never graded on the current scale.

## Status

v0.5 draft, published for comment alongside the main standard document - and **frozen**:
the grammar and invariants above are stable for this comment period. Feedback is triaged,
not folded in on arrival; changes land batched in the next version, and only a demonstrated
exploit, an implementer-reported ambiguity, or the independent cryptographic review reopens
the draft early.
