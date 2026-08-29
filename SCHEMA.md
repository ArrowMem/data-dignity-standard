# The declaration file - format v0.3

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

**Scope: a declaration covers exactly the origin that serves it, and nothing else.** The file at
`https://example.com/.well-known/data-dignity.json` speaks for `example.com` only - not
`api.example.com`, not `shop.example.com`, not any other host. Each origin that handles
agent-delivered data publishes its own declaration. A checker MUST NOT apply one origin's
declaration to another. **The same rule holds mid-session: when an agent follows a navigation,
redirect or API transition to a new origin, the new origin is outside the prior declaration
unless it separately declares** - a transaction that starts on a covered origin does not carry
that coverage to the origins it lands on.

## Shape

This is an illustrative example, not any real organization's actual declaration:

```json
{
  "version": "0.3",
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
      "purpose": "content delivery", "data_seen": ["network_metadata"],
      "jurisdiction": ["example-region"] }
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
    "covers": ["identity"],
    "legal_holds": [
      { "covers": "consent records", "reason": "consent must remain provable under applicable privacy law" }
    ]
  },
  "receipt_support": false
}
```

### Data classes

Free-text descriptions of data invite radically different readings of the same declaration, so
`data_seen`, `retention` keys and `deletion.covers` draw from one controlled vocabulary:

`content` (what the interaction produced - prompts, messages, uploads), `identity` (who the
interaction was with), `authentication` (tokens, credentials), `network_metadata` (IP,
headers), `device_metadata` (fingerprint, user agent), `financial`, `health`, `location`,
`files`, `derived` (anything computed from the above - embeddings, scores, profiles).

**Custom classes are additional to the standard vocabulary, never a substitute for it.** A
declaration MAY name site-specific classes, but each one MUST be mapped to one or more standard
classes in `custom_classes`, and a declaration MUST NOT use a custom class to avoid
classification under a standard class it plainly fits. A vague label ("operational telemetry",
"risk intelligence") with no mapping is a semantic evasion, and a checker grades any unmapped
custom class as undeclared. "Agent-delivered data" means every class above that an agent's
interaction causes the site to receive - including metadata the agent did not deliberately send.

### Field notes

- `declared` / `expires` / `last_reviewed` - a declaration is a claim about now, not forever.
  `expires` is REQUIRED: an expired declaration is graded as stale, not as current truth,
  because a site can change its entire infrastructure long after publishing a clean
  declaration. Twelve months is the longest sensible window.
- `retention` - **a machine-checkable map from data class to retention duration**: `none`,
  `indefinite`, or a count of days/months/years (`90d`, `6m`, `2y`). `content` is always
  required. **Coverage rule: every class that appears anywhere in the declaration - in any
  third party's `data_seen`, in `deletion.covers`, or via a custom-class mapping - MUST have a
  retention entry**, so a checker can verify mechanically that nothing seen is unaccounted for,
  with no interpretation required. Most real sites will have SOME identity-adjacent retention
  for legitimate reasons (support, fraud prevention, legal requirements) even when they store
  zero content - per-class honesty is rewarded over a blanket "we retain nothing" that a check
  would falsify. `notes` stays free-text for the human explanation.
- `third_parties` - every entity that sees agent-delivered data, with a `role` (`processor`,
  `subprocessor`, `infrastructure`, `analytics`, `security`, `model_provider`), what it sees,
  and where it processes. **`identifier` is REQUIRED and is the machine-readable identity - a
  stable domain or operator-controlled URI; `name` is presentation only.** Corporate families
  and lookalike names ("Example AI", "Example AI Analytics", an acquired vendor) make a bare
  name unverifiable; an identifier is what a checker can actually match observed traffic
  against. "Sees" includes server-side forwarding that no outside network observer can watch -
  a site that stores nothing but forwards everything to a model provider has disclosed nothing
  until that provider is named. An empty array is a valid, gradeable answer.
- `jurisdiction` - where the data is processed, stored and backed up, as separate answers,
  because they are usually different places. Region codes preferred where independently
  checkable. Legal jurisdiction (whose law applies) may differ from physical location; where
  they differ, say both.
- `deletion` - a real, callable handle. The whole point is machine-callable, not a support
  ticket paragraph. **`deletion.handle` MUST have the same origin as the declaration that
  advertises it.** A cross-origin deletion handle is the cleanest capability-exfiltration path
  in this format - an agent that later exercises the declared path would carry its deletion
  token to whatever origin the declaration named - so it is not a style rule, it is the
  boundary. A checker grades a cross-origin handle F on this clause. The declaration must also
  say how a caller proves it is entitled to delete what it asks to delete (`auth`), because an
  unauthenticated deletion endpoint is a weapon pointed at the site's own users. Requirements
  on the endpoint itself:
  - it MUST require evidence tying the request to the interaction being deleted (a receipt
    token where receipts exist, or an equivalent per-interaction secret) - never delete on an
    unauthenticated request;
  - it MUST be idempotent - the same deletion requested twice is not an error;
  - its success response MUST say what actually happened: `deleted`, `queued` (with an
    expected completion time), or `nothing-held` - a bare HTTP 200 is not an answer;
  - anything it cannot delete must appear in `legal_holds`, each entry naming what is kept and
    the legal reason, machine-readable. "We keep X because law Y" passes; silence fails;
  - it SHOULD rate-limit and monitor for abuse - authenticated deletion is still a write
    endpoint an adversary can hammer.

  The minimal request/response shape, normative so implementations converge and probes have
  one surface to test rather than many:

  ```json
  POST /api/agent-delete
  { "interaction_id": "an-opaque-id", "deletion_token": "an-opaque-single-use-token" }

  200 { "status": "deleted", "covers": ["identity"], "completed": "2026-01-02T00:00:00Z" }
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

  **A receipt MUST NOT echo the content it describes.** It names data classes and handling
  state, never the words, files or values themselves - a receipt that restates a health
  question is a fresh copy of the most sensitive thing in the interaction. **A receipt's
  `interaction_id` MUST identify one interaction unambiguously within its origin** -
  conceptually bound to the origin, an interaction nonce and a timestamp - so an agent's
  ledger entry and a site's receipt can be reconciled and neither side can point one receipt
  at a different interaction after the fact. **A receipt carrying a `deletion_token` is
  carrying a live capability, and both sides treat it that way**: the site keeps the token
  out of its own logs and caches, the agent stores it in its protected capability store (see
  the ledger rules in the main document), never in plain exports, and intermediaries that
  cache receipts are a named risk the token's single-use, short-lived binding exists to
  contain. A future signed receipt (a signature over the canonical interaction record) is
  where the `signature` extension below earns its place.
- `signature` - OPTIONAL, reserved: a declaration may carry a detached signature binding it to
  its origin, so an agent can detect a tampered or replayed file even past a compromised CDN.
  HTTP Message Signatures (RFC 9421) is the candidate mechanism. The concrete profile
  (algorithms, key discovery) is deliberately not specified here: it ships only after
  independent cryptographic review, and until then HTTPS transport is the trust model, stated
  plainly as a limitation rather than dressed up as a guarantee.

## Capability binding

One principle closes a whole family of composition attacks, so it is stated once and applies
everywhere: **every sensitive capability this ecosystem mints - a deletion token, a receipt
token, a future bulk-operation authorization, a future signature - MUST be bound to its
ORIGIN, its INTERACTION, its PURPOSE, an EXPIRY, and a NONCE.** A capability that any holder
can spend anywhere, on anything, forever, is a bearer credential waiting to leak across a
layer boundary; the IETF's sender-constrained-token work (RFC 9449) documents exactly this
failure mode. Applied here:

- a deletion token is valid only at the origin that issued it, only for the interaction it
  names, only for deletion, single-use, and short-lived;
- possession of a token is never authorization by itself - an agent holding a ledger full of
  deletion tokens has the *means* to delete, and still needs the person's explicit
  authorization to *act* (see the agent-side rules below);
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
- MUST NOT follow a redirect to a different origin than the one it is checking.
- MUST enforce strict response-size and timeout limits on everything it fetches.
- MUST NOT attach credentials, cookies or ambient authority to any request a declaration
  caused it to make.
- MUST treat **every declaration value as attacker-controlled data, never as executable
  instructions - in reports, and equally in anything downstream**: a `name`, `purpose`,
  `reason` or custom class may end up in a log line, a shell argument, a query, a URL, a
  spreadsheet export or a metrics label, and it is untrusted text in every one of those
  places, not just in rendered HTML.
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
  fingerprint - randomized payloads, realistic timing, varied vantage points and networks -
  and SHOULD re-check for reappearance at spaced intervals (immediately, after a day, after a
  week or more) before concluding "not reappeared": retention that surfaces through a batch
  pipeline or a delayed export is invisible to a single immediate probe. The full canary
  methodology is its own document on the roadmap; these floors hold in the meantime.
- MUST, when it encounters an earlier-format file, say so in its output and grade against
  that format's own weaker rules - "v0.1: missing expires and deletion.auth, graded under
  v0.1 rules" - never silently upgrade or downgrade a declaration across format versions.

Sites are held to the mirror rule: **a site MUST NOT special-case traffic it believes to be a
conformance test.** Retention tests use randomized, unmarked synthetic data precisely so that
recognizing and suppressing the test is harder than simply behaving; a site caught doing it is
graded as declared-and-false, the worst grade on the sheet.

**A limit stated so nobody reads past it: reappearance testing does not establish absence of
transformed retention.** A site can delete the words and keep an embedding, a classification,
a risk score, a profile vector - the exact text never reappears, and something derived from it
is retained all the same. That is why `derived` is its own declaration class with its own
retention entry, why recognition-style probes (does the site *behave* as if it remembers) are
part of the verification roadmap alongside exact replay, and why a clean reappearance test is
evidence for `content`, never for `derived`.

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
chain; together they cut all of them. The other standing chains: CDN-shared infrastructure
making two origins look like one (closed by exact-origin scope and the mid-session transition
rule), and a checker turned into a weapon through the URLs and values a declaration feeds it
(closed by the checker rules above).

## Open questions

- The concrete signature profile and the concrete deletion-token construction (see above) -
  both defined as extension points with normative binding requirements, deliberately
  unprofiled pending independent cryptographic review.
- Versioning and migration. v0.3 is a breaking change to v0.2's grammar (`retention` became a
  per-class duration map, `third_parties[].identifier` became required, `custom_classes` was
  added) made while the standard has no adopters - the honest moment for it. v0.1 files remain
  readable; checkers grade earlier formats against their own weaker requirements and note
  what is missing.

## Status

v0.3 draft, published for comment alongside the main standard document.
