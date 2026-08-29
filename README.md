# The Data Dignity Standard™

*[Version française](README.fr.md)*

**v0.2 - Draft for public comment**

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

This version covers five clauses, the site-side half of the standard. Further
clauses - covering what a dignity-respecting AI agent itself must do, plus stronger enforcement
layers - are planned for later versions; they are not omitted by oversight.

### 1 - Declared retention, verified, not trusted
The site states what it retains from an agent interaction, for how long, and why. Verified by
submitting real test transactions carrying randomized synthetic data and checking for their
reappearance - not by reading the declaration and taking it on faith. Tests are deliberately
unmarked and indistinguishable from ordinary traffic, and a site must not special-case traffic
it suspects is a test: passing by recognizing the auditor is graded as declared-and-false.

### 2 - No undisclosed third parties
Every party that sees agent-delivered data is named - including processors and subprocessors a
site forwards data to on the server side, where no outside observer can watch. Observed network
traffic during a real agent interaction is checked against the declared list; what network
observation cannot see is covered by the declaration itself, and a named party later found
missing grades as declared-and-false.

### 3 - Jurisdiction, stated plainly
Where the data is processed, stored and backed up is named in the declaration - as separate
answers, because they are usually different places - not left for a user to guess or discover
after the fact.

### 4 - A working deletion path
A real, machine-callable way to request deletion of what an interaction left behind - and
plain honesty about anything a site is legally required to keep, and why. The deletion
endpoint must authenticate that the caller is entitled to delete what it asks to delete, and
must answer with what actually happened, not a bare acknowledgement. The format's rules for
this are in `SCHEMA.md`.

### 5 - The receipt
Per interaction, a site can return a short receipt: what was received, what's retained and
until when, who else saw it, and the deletion path for that specific interaction. A receipt
names data classes, never the content itself - it must not become a second copy of the
sensitive thing it describes. This is the newest and most demanding clause, and the one we
expect very few sites to support at first - which is exactly why it exists as its own graded
line rather than a requirement.

## Grading

Each clause is scored, not pass/fail - a site with real gaps still has a ladder to climb, and a
score can be re-checked over time as a site changes. Composite and per-clause grades are both
reported, each labelled with its evidence level (declared, observed, or verified).

The scoring rule is deterministic, so two independent checkers reach the same grade:

- A clause that is declared and independently verified grades highest.
- A clause that is declared, with observed behavior consistent and nothing contradicting it,
  grades in the middle.
- A clause that is undeclared grades low - but above false.
- A clause whose declaration is contradicted by evidence grades F, and caps the composite:
  **a site caught in one false statement cannot outscore a site that stayed silent.**
- An expired declaration (see `expires` in `SCHEMA.md`) is graded as stale: no clause it
  covers can grade above the undeclared level until it is re-issued.

**A Data Dignity grade is evidence of conformance to the defined tests. It is not proof that a
site contains no undisclosed processing** - no external test can prove a negative about
someone else's infrastructure, and a grade that pretended otherwise would be the exact kind of
false comfort this standard exists to end.

## Status

This is a v0.2 draft, published for comment. ArrowMem intends to publish its own declaration
file and score against this standard before asking anyone else to. A free, open reference
checker is planned to follow shortly after the standard itself, built to the checker rules in
`SCHEMA.md`.

**Feedback and questions:** open an issue on this repository, or email git@arrowmem.ca

**License:** this document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- see LICENSE. "Data Dignity Standard" is a trademark of ArrowMem (application pending); the
trademark is a separate matter from this license and is not granted by it.

The ArrowMem team
