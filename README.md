# The Data Dignity Standard™

*[Version française](README.fr.md)*

**v0.1 - Draft for public comment**

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

## The declaration file

Any site can publish a small JSON file at:

```
https://<site>/.well-known/data-dignity.json
```

describing what it retains, who else sees agent-delivered data, where it's processed, and how
to request deletion. (`.well-known/` is the existing, IETF-registered convention used for this
kind of site-level declaration - the same pattern `security.txt` already uses.) Full format:
`SCHEMA.md` in this repository.

## The clauses

This first version covers five clauses, the site-side half of the standard. Further
clauses - covering what a dignity-respecting AI agent itself must do, plus stronger enforcement
layers - are planned for later versions; they are not omitted by oversight.

### 1 - Declared retention, verified, not trusted
The site states what it retains from an agent interaction, for how long, and why. Verified by
checking real, marked test transactions for reappearance - not by reading the declaration and
taking it on faith.

### 2 - No undisclosed third parties
Every third party that sees agent-delivered data is named. Observed network traffic during a
real agent interaction is checked against the declared list.

### 3 - Jurisdiction, stated plainly
Where the data is processed and stored is named in the declaration, not left for a user to
guess or discover after the fact.

### 4 - A working deletion path
A real, machine-callable way to request deletion of what an interaction left behind - and
plain honesty about anything a site is legally required to keep, and why.

### 5 - The receipt
Per interaction, a site can return a short receipt: what was received, what's retained and
until when, who else saw it, and the deletion path for that specific interaction. This is the
newest and most demanding clause, and the one we expect very few sites to support at first -
which is exactly why it exists as its own graded line rather than a requirement.

## Grading

Each clause is scored, not pass/fail - a site with real gaps still has a ladder to climb, and a
score can be re-checked over time as a site changes. Composite and per-clause grades are both
reported.

## Status

This is a v0.1 draft, published for comment. ArrowMem intends to publish its own declaration
file and score against this standard before asking anyone else to. A free, open reference
checker is planned to follow shortly after the standard itself.

**Feedback and questions:** open an issue on this repository, or email git@arrowmem.ca

**License:** this document is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- see LICENSE. "Data Dignity Standard" is a trademark of ArrowMem (application pending); the
trademark is a separate matter from this license and is not granted by it.

The ArrowMem team
