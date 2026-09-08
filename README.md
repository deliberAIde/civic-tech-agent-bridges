# Civic tech agent-bridges toolkit

Open-source command-line clients that let an AI agent operate a civic-tech platform through the
platform's own API.

One agent driving two of these bridges moves data and orchestrates work between two platforms,
without an integration having been built in advance for that pair. That is the whole idea, and
it works today.

## Why

Civic tech platforms have never worked well together. Every organisation that wanted two tools to
cooperate has paid for a bespoke integration, and most of those integrations died with the
project that funded them. The field ends up as a set of capable, isolated tools: a city can run
its deliberation in one and its vote in another, and still carry the record between them by hand.

What changed is the cost of a client. A complete, tested command-line client for a platform's API
used to be weeks of work that nobody would fund for a single integration. With a coding agent it
is hours. That makes it reasonable for every platform to have one — and once two platforms do, an
agent can operate both, so the integration nobody would have paid to build no longer needs
building.

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
logic, or requires it to change. Early-stage and openly incomplete: a bridge covers what its
platform's API exposes, which differs a lot between platforms and versions.

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
With tools like [CLI-Anything](https://github.com/HKUDS/CLI-Anything) (MIT, third-party) this is
hours of work, not weeks. In one timed run, a read-and-write Decidim CLI reached a working,
tested state in 49 minutes.

**Run time.** An agent drives the bridges and maps between them: read from one platform,
reshape, write into another. For records that have to be auditable, the agent emits a reusable
transform script rather than translating ad hoc, so the mapping can be reviewed and replayed.

Where meaning is genuinely absent at the source, no tooling conjures it. An agent can flag the
gap; closing it is semantic work.

## The goal

Not a toolkit maintained by one company. The goal is that every civic-tech platform has an open
bridge, maintained by the community that maintains the platform, the way client libraries work
everywhere else in software. Then interoperability stops being a project that has to be funded
each time and becomes a property: any two platforms with bridges can be composed, by an agent or
by a plain script, without their communities having to coordinate with each other first.

These four are a starting set, and evidence that the pattern holds across very different
architectures: a GraphQL API, a REST API, and a Rails application with no complete operator API
of its own. We would rather each bridge ended up in its own platform's repository than in ours.

## Where this sits next to standards work

Standards efforts solve a different problem: shared semantics and portable formats, so that
records mean the same thing across tools. [Metagov's Interoperable Deliberative
Tools](https://metagov.org/projects/interop) is the current reference point, and Decidim
participates in it. Bridges solve reachability: making a platform operable by an agent right
now, on the API it already has. The two compose well, and neither removes the need for the
other.

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
repository at the same time.

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
