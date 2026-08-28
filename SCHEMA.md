# The declaration file - format v0.1

Defines the grammar every clause in the standard tests against. Kept deliberately boring: the
llms.txt lesson is that plain JSON at a fixed, guessable path beats anything clever.

## Location

```
https://<site>/.well-known/data-dignity.json
```

`.well-known/` is the existing IETF-registered convention (RFC 8615) for exactly this kind of
site-level declaration - `security.txt` lives here, so does the pattern this borrows. Reusing
an existing convention instead of inventing a new path is deliberate: an agent (or a checker)
already knows to look there.

## Shape

This is an illustrative example, not any real organization's actual declaration:

```json
{
  "version": "0.1",
  "declared": "2026-01-01T00:00:00Z",
  "retention": {
    "content": "none",
    "identity": ["support-ticket"],
    "notes": "Support tickets are kept for 90 days to resolve disputes. Nothing else is stored."
  },
  "third_parties": [
    { "name": "Example CDN Co", "purpose": "content delivery", "data_seen": ["ip"] }
  ],
  "jurisdiction": ["example-region"],
  "deletion": {
    "handle": "https://example.com/api/agent-delete",
    "method": "POST",
    "covers": "the identity records named above; content is never stored to begin with"
  },
  "receipt_support": false
}
```

### Field notes

- `retention` - what's kept, split into `content` (what an interaction produced) and `identity`
  (anything kept about who the interaction was with, and why). Most real sites will have SOME
  identity-adjacent retention for legitimate reasons (support, fraud prevention, legal
  requirements) even when they store zero content - the split rewards declaring that honestly
  rather than a blanket "we retain nothing" that a check would falsify.
- `third_parties` - every origin that sees agent-delivered data, named, with what it sees. An
  empty array is a valid, gradeable answer.
- `jurisdiction` - where the data is processed and stored, plain strings, region codes
  preferred where they're independently checkable.
- `deletion` - a real, callable handle. The whole point is machine-callable, not a support
  ticket paragraph.
- `receipt_support` - whether the site supports per-interaction receipts. `false` is an honest,
  gradeable answer, not a failure by itself - this is the newest and most demanding part of the
  standard, and few sites are expected to support it early.

## Open questions

- Signing/integrity (should the file be signed so it can't be tampered with in transit)?
  Flagged, not designed - v0.1 assumes plain HTTPS transport is sufficient for a first version,
  the same trust model most similar declaration files use.
- Versioning and migration once a v0.2 exists. Not needed until there is one.

## Status

v0.1 draft, published for comment alongside the main standard document.
