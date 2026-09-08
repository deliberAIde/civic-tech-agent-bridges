# Civic tech agent-bridges toolkit

Open-source command-line clients that let an AI agent operate a civic-tech platform through the
platform's own API. One agent driving two of these bridges moves data and orchestrates work
between two platforms, with no shared standard and no integration built in advance for that
pair.

## Why

Interoperability remains civic tech's unfinished business. Open311 was widely adopted but is no
longer maintained. Germany's OParl standard has not been updated since 2018 and is implemented
only partially. The Scottish Government's 2024 market research named the lack of interoperability
between participation tools a defining constraint of the field. Serious standards work is
underway: [Metagov's Interoperable Deliberative Tools](https://metagov.org/projects/interop)
effort, in which Decidim participates, and Decidim's own CSV export and import work are the most
visible current examples.

Two things have changed. Coding agents can now generate a working CLI from any well-structured
API in under an hour. And general-purpose agents can drive such CLIs to move data and
orchestrate workflows across platforms. A platform with a CLI is operable by an agent; two
platforms with CLIs can be made to work together by an agent; and the CLI itself is a
deterministic, reviewable artefact that is cheap to build and cheap to keep current.

That does not replace standards work. The agent layer dissolves the problems of format and
transport: an agent reads both schemas and maps between them. It does not dissolve shared
semantics, provenance, or the safeguards that responsible agent access requires. Those still
need standards and coordination, and the bridges are built to sit next to that work rather than
in place of it.

## The bridges

| Platform | Bridge | Talks to | Licence |
|---|---|---|---|
| [deliberAIde](https://deliberaide.com) | [`deliberaide-cli`](https://pypi.org/project/deliberaide-cli/) | REST + Co-Pilot WebSocket | proprietary client, open interface |
| [Pol.is](https://github.com/compdemocracy/polis), [Voxit](https://gitlab.com/voxit) | [`polis-cli`](https://github.com/deliberAIde/polis-cli) | Pol.is HTTP API | Apache-2.0 |
| [CONSUL DEMOCRACY](https://github.com/consuldemocracy/consuldemocracy) | [`consul-cli`](https://github.com/deliberAIde/consul-cli) | Rails routes, models and rake tasks, via an operator bridge | Apache-2.0 client, AGPL-3.0 engine |
| [Decidim](https://github.com/decidim/decidim) | [`decidim-cli`](https://github.com/deliberAIde/decidim-cli) | GraphQL API | Apache-2.0 client, AGPL-3.0 engine |

```bash
pip install polis-cli decidim-cli consul-democracy-cli deliberaide-cli
```

Each bridge speaks to a platform's own API. None of them forks a platform, replaces any of its
logic, or requires it to change. Together with [CLI-Anything](https://github.com/HKUDS/CLI-Anything),
a third-party open-source tool that generates such clients, and an API-design guide for
agent-ready platforms, they form an emerging and openly incomplete toolkit: a bridge covers what
its platform's API exposes, which differs a lot between platforms and versions.

deliberAIde itself is not yet open source; it is an early-stage platform in its pilot phase. The
bridges and this toolkit are.

## In practice

Each bridge exposes its platform's own lifecycle as commands, with named profiles for instances
and `--json` on everything:

```bash
decidim --json -p city  process create "Mobility plan 2030"
polis   --json -p vote  convo create "Final vote on the recommendations"
polis   --json -p vote  seed <conversation-id> statements.txt
consul  --json -p city  proposals list --where '{"projekt_phase_id": 4}'
```

An operator can run these by hand, which is what keeps them debuggable. An agent can chain them,
which is what makes platforms interoperate: because every command answers in the same envelope
shape, moving between two platforms needs no adapter written in advance for that pair.

Typical shapes this enables:

- take the output of a deliberation on one platform and open a vote on it on another;
- publish the result of a decision into a public participation portal, then read the public
  response back for analysis;
- run one reporting query across several installations of different platforms;
- move a process between instances, or between platforms, with the transformation included.

A worked example: a deliberation tool holds the reasoning and its evidence, a voting tool runs
the decision, and a participation portal opens the outcome to a wider public. Three platforms,
three bridges, one agent carrying the record from one to the next and reconciling them at the
end, with nothing built in advance between them.

## Two layers

**Build time.** An agent writes the bridge once, from an API description or a codebase. The
result is a deterministic, versioned, reviewable artefact that behaves identically on every run.
With a tool like CLI-Anything this takes under an hour for a first working version; in one timed
run, a read-and-write Decidim CLI reached a working, tested state in 49 minutes.

**Run time.** An agent drives the bridges and converts between them: fetch from one platform,
reshape, write into another. No shared format is needed, because the agent reads both schemas
and maps the fields. For audit-grade records the agent emits a reusable transform script rather
than translating ad hoc, so intermediate files and logs form the audit trail and the mapping can
be reviewed and replayed.

Where meaning is missing at the source, no tooling and no standard conjures it. The agent's job
there is to flag the gap.

## What still needs standards and coordination

- **Shared semantics.** A format can be mapped; a concept that one platform records and another
  does not cannot be translated into existence. This is the work standards efforts do.
- **Provenance.** Which platform, which version, which operator, which transformation. Bridges
  keep this visible in their envelopes and transform scripts; a shared vocabulary for it would
  be better.
- **Safeguards for agent access.** API scopes, moderation, rate limits, and accountability for
  agent-mediated contributions. Decidim's action logging for API users is an existing answer
  worth copying.

## Licensing: why the clients are permissive

Two kinds of code live in these repositories, licensed differently on purpose.

**The clients are Apache-2.0.** A client that speaks HTTP or GraphQL to a running instance
contains none of that platform's source code, so it is an independent work. Permissive licensing
means a platform can vendor it into its own repository, whether that is AGPL-3.0 (Pol.is,
CONSUL, Decidim) or EUPL-1.2 (Voxit), since permissive code can be combined into a copyleft
project but not the other way round. The licence was chosen for the platforms' convenience.

**The in-application engines are AGPL-3.0-or-later.** `consul-admin-api` and `decidim-admin-api`
are Rails engines that load into and run as part of an AGPL application, so they follow their
host's licence.

Contributing a bridge upstream does not move its copyright. Neither CONSUL nor Decidim asks
contributors for an assignment, so the same code can live here and in a platform's own
repository at the same time. We would rather each bridge ended up in its own platform's
repository than in ours.

## Build a bridge for your platform

- [Building an agent bridge](docs/building-an-agent-bridge.md) — the shape of a bridge, and how
  to get a working one out of an existing API with a coding agent.
- [Designing an API an agent can drive](docs/api-design-for-agent-bridges.md) — which API
  choices made these bridges easy or hard to build.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Most useful: a bridge for a platform that has none yet, a
pull request taking one of these into its own platform's repository, and corrections from the
people who maintain the APIs involved.

## Licence

Apache-2.0 for this repository. Each bridge carries its own licence; see the table above.
Copyright 2026 deliberAIde.
