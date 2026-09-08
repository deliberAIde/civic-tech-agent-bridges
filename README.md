# Civic tech agent-bridges toolkit

**An open question, explored in working code: now that coding agents can generate a usable CLI
from a well-structured API in about an hour, and general-purpose agents can drive such CLIs
across platforms, what does that mean for how civic tech becomes interoperable?**

Interoperability remains civic tech's unfinished business. Open311 was widely adopted but is no
longer maintained; Germany's OParl standard has not been formally updated since 2018 and is
implemented only partially; the Scottish Government's 2024 market research named the lack of
interoperability between participation tools a defining constraint of the field.

Serious standards work is underway. [Metagov's Interoperable Deliberative
Tools](https://metagov.org/projects/interop) effort is the most important current example, and
Decidim participates in it through its CSV export and import project. That work addresses
shared semantics and portable formats, which is a different problem from the one here.

This repository explores a complementary path. A command-line client makes a platform drivable
by an agent. One agent driving two such clients can move data and orchestrate work between two
platforms without an integration having been built in advance for that particular pair. How far
that goes, which problems it dissolves and which still need standards and coordination, is
exactly what these bridges are for testing.

## The bridges

| Platform | Bridge | Talks to | Licence | Status |
|---|---|---|---|---|
| [deliberAIde](https://deliberaide.com) | [`deliberaide-cli`](https://pypi.org/project/deliberaide-cli/) | REST + Co-Pilot WebSocket | proprietary client, open interface | published on PyPI |
| [Pol.is](https://github.com/compdemocracy/polis), [Voxit](https://gitlab.com/voxit) | [`polis-cli`](https://github.com/deliberAIde/polis-cli) | Pol.is HTTP API | Apache-2.0 | offered to the Pol.is and Voxit communities |
| [CONSUL DEMOCRACY](https://github.com/consuldemocracy/consuldemocracy) | [`consul-cli`](https://github.com/deliberAIde/consul-cli) | Rails routes, models and rake tasks, via an operator bridge | Apache-2.0 client, AGPL-3.0 engine | offered to the CONSUL community |
| [Decidim](https://github.com/decidim/decidim) | [`decidim-cli`](https://github.com/deliberAIde/decidim-cli) | GraphQL API | Apache-2.0 client, AGPL-3.0 engine | offered to the Decidim community |

Each bridge is a normal package that speaks to a platform's own API. None of them forks a
platform, replaces any of its logic, or asks it to change how it works.

```bash
pip install polis-cli decidim-cli consul-democracy-cli deliberaide-cli
```

This is an early-stage, emerging toolkit. It is useful today and openly incomplete: the bridges
cover what their platforms' APIs expose, which differs a lot between platforms and versions.

## Two layers

It helps to separate what an agent does when:

**At build time** an agent writes the bridge, once, from an API description or a codebase. The
result is a deterministic, versioned, reviewable artefact that behaves the same on every run.
Tools like [CLI-Anything](https://github.com/HKUDS/CLI-Anything) (MIT, third-party) make this a
matter of hours rather than weeks.

**At run time** an agent drives the bridges and maps between them: read from one platform,
reshape, write into another. For records that need to be auditable, the agent should emit a
reusable transform script rather than translating ad hoc, so the mapping can be reviewed and
repeated.

Where meaning is genuinely missing at the source, neither approach conjures it. An agent can
flag the gap; closing it is semantic work, and that is where standards efforts do the heavy
lifting.

## What it looks like in practice

One organiser, one general-purpose agent, three platforms, one democratic record:

1. A citizens' assembly deliberates. The agent asks **deliberAIde** for the recommendation
   package, grounded in the transcript with citations.
2. The room becomes the assembly's final plenary. The agent creates a **Voxit** conversation
   from those recommendations, and participants vote from their phones.
3. The agent reads the vote, classifies what was adopted, what carries conditions and what
   stays contested, and publishes the package into **Munich CONSUL** for wider public input.
4. Later, the agent reconciles all three records without erasing dissent.

Each platform was driven through its own API, by one agent, with no integration built in
advance between them.

## Licensing: why the clients are permissive

Two kinds of code live in these repositories, licensed differently on purpose.

**The clients are Apache-2.0.** A client that speaks HTTP or GraphQL to a running instance
contains none of that platform's source code, so it is an independent work. Licensing it
permissively means a platform can vendor it into its own repository, whether that is AGPL-3.0
(Pol.is, CONSUL, Decidim) or EUPL-1.2 (Voxit), since permissive code can be combined into a
copyleft project but not the other way round. The licence was chosen for the platforms'
convenience, not ours.

**The in-application engines are AGPL-3.0-or-later.** `consul-admin-api` and `decidim-admin-api`
are Rails engines that load into and run as part of an AGPL application, so they follow their
host's licence.

Contributing a bridge upstream does not move its copyright: neither CONSUL nor Decidim asks
contributors for an assignment, so the same code can live here and in a platform's own
repository at the same time.

## Build a bridge for your platform

- [Building an agent bridge](docs/building-an-agent-bridge.md) — the shape of a bridge, and how
  to get a working one out of an existing API using a coding agent.
- [Designing an API an agent can drive](docs/api-design-for-agent-bridges.md) — what makes a
  bridge easy to build and safe to run.

## Open questions

These are genuinely open, and worth more than the code:

- What belongs in a standard once agents can bridge formats at low cost?
- What safeguards does responsible agent access need: API scopes, moderation, rate limits, and
  accountability for agent-mediated contributions?
- Where does the semantic work stay irreducible, however good the tooling gets?

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The most useful contributions are a bridge for a
platform that has none yet, a pull request taking one of these bridges into its own platform's
repository, and corrections from people who maintain the APIs involved.

## Licence

Apache-2.0 for this repository. Each bridge carries its own licence; see the table above.
Copyright 2026 deliberAIde.
