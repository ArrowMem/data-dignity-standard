# The declaration file - format v0.2

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
declaration to another.

## Shape

This is an illustrative example, not any real organization's actual declaration:

```json
{
  "version": "0.2",
  "declared": "2026-01-01T00:00:00Z",
  "expires": "2026-07-01T00:00:00Z",
  "last_reviewed": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": ["support-ticket"],
    "notes": "Support tickets are kept for 90 days to resolve disputes. Nothing else is stored."
  },
  "third_parties": [
    { "name": "Example CDN Co", "role": "infrastructure", "purpose": "content delivery",
      "data_seen": ["network_metadata"], "jurisdiction": ["example-region"] }
  ],
  "jurisdiction": {
    "processing": ["example-region"],
    "storage": ["example-region"],
    "backups": ["example-region"]
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

A declaration MAY add site-specific classes, but everything it retains or shares must be
covered by at least one named class. "Agent-delivered data" means every class above that an
agent's interaction causes the site to receive - including metadata the agent did not
deliberately send.

### Field notes

- `declared` / `expires` / `last_reviewed` - a declaration is a claim about now, not forever.
  `expires` is REQUIRED from v0.2: an expired declaration is graded as stale, not as current
  truth, because a site can change its entire infrastructure long after publishing a clean
  declaration. Twelve months is the longest sensible window.
- `retention` - what's kept, split into `content` and `identity`. Most real sites will have
  SOME identity-adjacent retention for legitimate reasons (support, fraud prevention, legal
  requirements) even when they store zero content - the split rewards declaring that honestly
  rather than a blanket "we retain nothing" that a check would falsify.
- `third_parties` - every entity that sees agent-delivered data, named, with a `role`
  (`processor`, `subprocessor`, `infrastructure`, `analytics`, `security`, `model_provider`),
  what it sees, and where it processes. "Sees" includes server-side forwarding that no outside
  network observer can watch - a site that stores nothing but forwards everything to a model
  provider has disclosed nothing until that provider is named. An empty array is a valid,
  gradeable answer.
- `jurisdiction` - where the data is processed, stored and backed up, as separate answers,
  because they are usually different places. Region codes preferred where independently
  checkable. Legal jurisdiction (whose law applies) may differ from physical location; where
  they differ, say both.
- `deletion` - a real, callable handle. The whole point is machine-callable, not a support
  ticket paragraph. From v0.2 the declaration must also say how a caller proves it is entitled
  to delete what it asks to delete (`auth`), because an unauthenticated deletion endpoint is a
  weapon pointed at the site's own users. Requirements on the endpoint itself:
  - it MUST require evidence tying the request to the interaction being deleted (a receipt
    token where receipts exist, or an equivalent per-interaction secret) - never delete on an
    unauthenticated request;
  - it MUST be idempotent - the same deletion requested twice is not an error;
  - its success response MUST say what actually happened: `deleted`, `queued` (with an
    expected completion time), or `nothing-held` - a bare HTTP 200 is not an answer;
  - anything it cannot delete must appear in `legal_holds`, each entry naming what is kept and
    the legal reason, machine-readable. "We keep X because law Y" passes; silence fails.
- `receipt_support` - whether the site supports per-interaction receipts. `false` is an honest,
  gradeable answer, not a failure by itself. Where supported, a receipt is a small JSON object:

  ```json
  {
    "interaction_id": "an-opaque-id",
    "received_classes": ["content", "identity"],
    "retention": { "content": "none", "identity": "30d" },
    "third_parties": ["Example CDN Co"],
    "deletion_token": "an-opaque-single-use-token"
  }
  ```

  **A receipt MUST NOT echo the content it describes.** It names data classes and handling
  state, never the words, files or values themselves - a receipt that restates a health
  question is a fresh copy of the most sensitive thing in the interaction.
- `signature` - OPTIONAL, reserved from v0.2: a declaration may carry a detached signature
  binding it to its origin, so an agent can detect a tampered or replayed file even past a
  compromised CDN. HTTP Message Signatures (RFC 9421) is the candidate mechanism. The concrete
  profile (algorithms, key discovery) is deliberately not specified here: it ships only after
  independent cryptographic review, and until then HTTPS transport is the trust model, stated
  plainly as a limitation rather than dressed up as a guarantee.

## Rules for checkers

The checker is an HTTP client processing input an adversary controls, and a hostile
declaration is the obvious way to attack it. Any conforming checker:

- MUST NOT dereference `deletion.handle` unsolicited. Presence and shape are checkable from
  the declaration alone; actually calling a deletion endpoint happens only with the site's
  consent and synthetic data, or not at all.
- MUST refuse any declared URL that resolves to loopback, private, link-local, multicast or
  otherwise reserved address space, and MUST connect to the address it resolved and checked,
  so a DNS answer cannot change between the check and the request.
- MUST NOT follow a redirect to a different origin than the one it is checking.
- MUST enforce strict response-size and timeout limits on everything it fetches.
- MUST NOT attach credentials, cookies or ambient authority to any request a declaration
  caused it to make.
- MUST treat every fetched byte as untrusted when rendering reports - a site name is
  attacker-controlled text, not markup.

Sites are held to the mirror rule: **a site MUST NOT special-case traffic it believes to be a
conformance test.** Retention tests use randomized, unmarked synthetic data precisely so that
recognizing and suppressing the test is harder than simply behaving; a site caught doing it is
graded as declared-and-false, the worst grade on the sheet.

## Open questions

- The concrete signature profile (see `signature` above) - defined as an extension point,
  deliberately unprofiled pending independent cryptographic review.
- Versioning and migration between declaration versions. v0.1 files remain readable; a checker
  grades them against v0.1's weaker requirements and notes the missing `expires`.

## Status

v0.2 draft, published for comment alongside the main standard document.
