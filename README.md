# The Data Dignity Standard™

*[Version française](README.fr.md)*

**v0.4 - Draft for public comment**

## Why this exists

AI agents are becoming a new class of visitor on the web - browsing, comparing, and buying on
a person's behalf. The tools for making a website *usable* by an agent are being built quickly:
structured data, agent-callable actions, discovery standards. Cloudflare's free Agent Readiness
Score, for example, already answers "can an agent use this site."

We haven't found another public standard that answers a different question: **should it?** When
a site hands data to - or receives data from - an AI agent acting for a real person, what
happens to that data? Is it kept? Shared? Where does it go? Today, there's no way for an agent,
or the person it's acting for, to know.

The Data Dignity Standard is a small, testable answer to that question. It is not a
certification you buy and it is not a gate anyone has to pass through - it's a public
declaration format and a free way to check it, so agents (and the people using them) can tell
which sites treat their data with respect. Grading is free and open to anyone; a separate,
optional paid service for ongoing monitoring may be offered to organizations that want their
grade tracked over time, but nothing about meeting the standard itself requires paying anyone.

## Principles

- **Testable, not aspirational.** Every clause below is something a machine can check. A
  standard nobody can verify is just a slogan.
- **Grading, not gatekeeping.** Sites are scored, never blocked. This is a report card, never
  a toll booth.
- **Declared and true beats undeclared. Declared and false is worse than either.** The standard's
  only real teeth: a site that states something false about its own data handling scores worse
  than a site that says nothing at all.
- **Say what the evidence is, not more.** Every graded claim is labelled by how it is known:
  **declared** (the organization says so), **observed** (a checker saw behavior consistent with
  it), **verified** (a checker reproduced it independently), or **unknown** (it cannot be
  established from outside). A grade never presents a declaration as a verification.

## The declaration file

Any site can publish a small JSON file at:

```
https://<site>/.well-known/data-dignity.json
```

describing what it retains, who else sees agent-delivered data, where it's processed, and how
to request deletion. The `/.well-known/` prefix is the standardized IETF location for this kind
of origin-level declaration (RFC 8615) - the same pattern `security.txt` already uses. Data
Dignity defines `data-dignity.json` within that namespace. A declaration speaks only for the
exact origin that serves it - subdomains publish their own. Full format, including the rules a
conforming checker must follow: `SCHEMA.md` in this repository.

## The clauses

This version covers six clauses: five for the site, and a first clause for the agent itself,
because data dignity is a two-sided property - sites are graded, agents can be certified.
Stronger enforcement layers are planned for later versions; they are not omitted by oversight.

### 1 - Declared retention, verified, not trusted
The site states what it retains from an agent interaction, per data class, for how long, and
why. Verified by submitting real test transactions carrying randomized synthetic data and
checking for their reappearance - not by reading the declaration and taking it on faith. Tests
are deliberately unmarked and indistinguishable from ordinary traffic, and a site must not
special-case traffic it suspects is a test: passing by recognizing the auditor is graded as
declared-and-false. One honest limit, stated rather than hidden: reappearance testing checks
the words, not their shadows - a site can delete the text and keep an embedding, a score, a
profile. That is why derived data is its own declared class with its own retention entry, and
why a clean replay test is evidence about content, never about what was computed from it.

### 2 - No undisclosed third parties
Every party that sees agent-delivered data is named - by a stable machine-readable identifier,
not just a display name - including processors and subprocessors a site forwards data to on
the server side, where no outside observer can watch. Observed network traffic during a real
agent interaction is checked against the declared list; what network observation cannot see is
covered by the declaration itself, and a party later found missing grades as
declared-and-false.

### 3 - Jurisdiction, stated plainly
Where the data is processed, stored and backed up is named in the declaration - as separate
answers, because they are usually different places - not left for a user to guess or discover
after the fact.

### 4 - A working deletion path
A real, machine-callable way to request deletion of what an interaction left behind - and
plain honesty about anything a site is legally required to keep, and why. The deletion
endpoint lives on the same origin as the declaration that advertises it - a deletion path
pointing anywhere else is how a deletion credential leaks to an attacker, and it fails this
clause outright. The endpoint must authenticate that the caller is entitled to delete what it
asks to delete, and must answer with what actually happened, not a bare acknowledgement. The
format's rules, including the request and response shapes, are in `SCHEMA.md`.

### 5 - The receipt
Per interaction, a site can return a short receipt: what was received, what's retained and
until when, who else saw it, and the deletion path for that specific interaction. A receipt
names data classes, never the content itself - it must not become a second copy of the
sensitive thing it describes. This is the newest and most demanding clause, and the one we
expect very few sites to support at first - which is exactly why it exists as its own graded
line rather than a requirement.

### 6 - The ledger (agent-side)
The mirror of the receipt, and the first clause about the agent rather than the site: a
dignity-respecting agent keeps its own ledger - every site it touched on a person's behalf,
what was disclosed, when, and each interaction's deletion path. When a site is breached years
later, one query answers "what did they have of mine," and deletion can be requested wherever
the person decides it should be.

The ledger is the most sensitive database this standard creates, so its rules have teeth:

- It records classes and metadata - site, origin, timestamp, interaction id, what classes were
  disclosed - **never the disclosed content itself**, the same no-echo rule receipts follow.
- **Audit records and live capabilities are stored apart.** Deletion tokens and any other
  credential live in a protected capability store, encrypted at rest, never in a plain export
  of the audit trail - so a leaked ledger export is a history, not a weapon.
- The ledger belongs to the person: exportable by them, retained and deleted on their say,
  never a hidden archive the agent keeps for itself.
- **Bulk deletion is never autonomous.** Holding a token is means, not authorization: any
  multi-site deletion run requires the person's explicit, informed go-ahead - preview what
  will be asked of which origins and which classes, get the authorization, execute, report
  per-origin results. An agent must never infer permission from the fact that the ledger
  contains a token, whatever a prompt, plugin or automation says - and **the authorization
  itself must be an act the person performs outside the model's context** (an OS-level
  confirmation, a signed approval object), because any approval that is just text in a
  prompt is text a prompt injection can fabricate.
- The capability store's keys are protected as strongly as the platform allows -
  OS-keystore or hardware-backed where available, never held in cleartext by the agent
  longer than the operation needs.
- **An interaction is ledgered whether or not the site issued a receipt.** Receipts
  reconcile the ledger; they do not define it - a site that issues none still appears, with
  its entry marked unreceipted, so silence on the site's side never becomes a blind spot on
  the person's side.

Certification tests the same things it names, adversarially: the agent produces a complete,
exportable ledger for a session with nothing silently dropped, entries reconcile against the
receipts of sites that issue them (and unreceipted interactions still appear), capabilities
sit outside the audit export and survive an export-leak attempt, and bulk deletion refuses to
run on a fabricated in-context approval - certification includes prompt-injection and
capability-leakage tests, not just the happy path. Sites are graded on clauses 1 through 5; agents are
certifiable on this one.

## Grading

Each clause is scored, not pass/fail - a site with real gaps still has a ladder to climb, and a
score can be re-checked over time as a site changes. Composite and per-clause grades are both
reported, each labelled with its evidence level (declared, observed, or verified).

The scoring rule is deterministic in the only way that survives adversaries: underneath
every grade sits a set of recorded facts - each individual test result is exactly PASS, FAIL,
UNKNOWN or NOT_TESTED - and the letters are computed from those facts by the published rule.
Judgement may decide what to test; it never decides what a recorded fact says. Two checkers
with the same facts reach the same grade, and incomplete evidence surfaces as UNKNOWN facts,
never as optimistic letters. The letters below are the v0.4 draft mapping, published for
comment like everything else here:

- **A** - declared and independently verified.
- **B** - declared, observed behavior consistent, nothing contradicting it.
- **D** - undeclared. Low, but above false, on purpose.
- **F** - declaration contradicted by evidence. An F on any clause also caps the composite
  at F: **a site caught in one false statement cannot outscore a site that stayed silent.**
- An expired declaration (see `expires` in `SCHEMA.md`) caps every clause it covers at
  D - the undeclared level - until it is re-issued.
- A declaration in a superseded format is reported **LEGACY** - parsed and displayed, never
  graded on the current scale. Grading an old format under its own weaker rules would be a
  standing invitation to never upgrade; `SCHEMA.md`'s version-security section is the rule.

The composite is the lowest of: the per-clause grades' mean (rounded down), and any cap
above. Same inputs, same grade, whoever runs the check.

**A Data Dignity grade is evidence of conformance to the defined tests. It is not proof that a
site contains no undisclosed processing** - no external test can prove a negative about
someone else's infrastructure, and a grade that pretended otherwise would be the exact kind of
false comfort this standard exists to end.

## Limitations, stated plainly

Until the reserved signature extension has a reviewed concrete profile, the trust model for
the declaration file is HTTPS plus control of the origin - a compromised origin or
misconfigured CDN can serve a false declaration, and an agent has no cryptographic way to
detect it yet. Purely server-side behavior can be declared and graded, but never externally
verified on its own - those claims stay at the declared evidence level until stronger
mechanisms exist. And reappearance testing bounds what a site kept of the words, not what it
computed from them. Every one of these limits is carried in the grades themselves rather than
papered over: that is what the evidence labels are for.

## Status

This is a v0.4 draft, published for comment - and frozen: the clauses, grammar and
invariants are stable for this comment period. Feedback is triaged, never folded in on
arrival; changes land batched in the next version, and only a demonstrated exploit, an
implementer-reported ambiguity, or the independent cryptographic review reopens the draft
early. ArrowMem intends to publish its own declaration
file and score against this standard before asking anyone else to. A free, open reference
checker is planned to follow shortly after the standard itself, built to the checker rules in
`SCHEMA.md`.

**Feedback and questions:** open an issue on this repository, or email git@arrowmem.ca

**License:** this document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- see LICENSE. "Data Dignity Standard" is a trademark of ArrowMem (application pending); the
trademark is a separate matter from this license and is not granted by it.

The ArrowMem team
