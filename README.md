# Civic tech agent-bridges toolkit

**Interoperability between civic-tech platforms does not need a standards process. It needs
each platform to be reachable by an agent.**

People increasingly work *through* agents rather than directly on a web interface. A
participation platform that only has a graphical user interface is invisible to that layer: an
agent cannot open a consultation in it, cannot read results out of it, cannot carry a
democratic record from one platform to the next.

The move that fixes this is small and available today. Take the API a platform already has,
wrap it in a documented command-line client, and any agent can drive the platform. Do that for
two platforms and they interoperate, without either of them agreeing to a shared schema first.

This repository is the map: the bridges that exist, how they are licensed, how to build one for
your own platform, and what to change in an API so a bridge is easy to build.

## The bridges

| Platform | Bridge | Talks to | Licence | Status |
|---|---|---|---|---|
| [deliberAIde](https://deliberaide.com) | [`deliberaide-cli`](https://pypi.org/project/deliberaide-cli/) | REST + Co-Pilot WebSocket | proprietary client, open interface | published on PyPI |
| [Pol.is](https://github.com/compdemocracy/polis), [Voxit](https://gitlab.com/voxit) | [`polis-cli`](https://github.com/deliberAIde/polis-cli) | Pol.is HTTP API | Apache-2.0 | offered to the Pol.is and Voxit communities |
| [CONSUL DEMOCRACY](https://github.com/consuldemocracy/consuldemocracy) | [`consul-cli`](https://github.com/deliberAIde/consul-cli) | Rails routes, models, rake tasks, via an operator bridge | Apache-2.0 client, AGPL-3.0 engine | offered to the CONSUL community |
| [Decidim](https://github.com/decidim/decidim) | [`decidim-cli`](https://github.com/deliberAIde/decidim-cli) | GraphQL API | Apache-2.0 client, AGPL-3.0 engine | offered to the Decidim community |

Each bridge is a normal package. Nothing here replaces a platform, forks it, or asks it to
change how it works.

```bash
pip install polis-cli decidim-cli consul-democracy-cli deliberaide-cli
```

## What it looks like in practice

One organiser, one general-purpose agent, three platforms, one democratic record:

1. A citizens' assembly deliberates. The agent asks **deliberAIde** for the recommendation
   package, grounded in the transcript with citations.
2. The room becomes the assembly's final plenary. The agent creates a **Voxit** conversation
   from those recommendations, and participants vote from their phones.
3. The agent reads the vote, classifies what was adopted, what carries conditions and what
   stays contested, and publishes the package into **Munich CONSUL** for wider public input.
4. Later, the agent reconciles all three records without erasing dissent.

No platform was modified. No shared schema was agreed. The bridges did the connecting, and the
agent did the carrying.

## Licensing: why the clients are permissive

Two kinds of code live in these repositories, and they are licensed differently on purpose.

**The clients are Apache-2.0.** A client that speaks HTTP or GraphQL to a running instance
contains none of that platform's source code, so it is an independent work. Licensing it
permissively means a platform can vendor it straight into its own repository, whether that is
AGPL-3.0 (Pol.is, CONSUL, Decidim) or EUPL-1.2 (Voxit). Permissive code can be combined into a
copyleft project; the reverse is blocked. An AGPL client would have made adoption harder for
exactly the communities it is meant for.

**The in-application engines are AGPL-3.0-or-later.** `consul-admin-api` and `decidim-admin-api`
are Rails engines that load into and run as part of an AGPL application. They follow their
host's licence, which is also what those communities would want.

Contributing a bridge upstream does not move its copyright: neither CONSUL nor Decidim asks
contributors for an assignment, so the same code can live in this toolkit and in a platform's
own repository at the same time.

## Build a bridge for your platform

- [Building an agent bridge](docs/building-an-agent-bridge.md) — the shape of a bridge, and how
  to get a first working one out of an existing API in a day using a coding agent and
  [CLI-Anything](https://github.com/HKUDS/CLI-Anything).
- [Designing an API an agent can drive](docs/api-design-for-agent-bridges.md) — what to change
  in an API so a bridge is easy to build and safe to run.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The most useful contribution is a bridge for a platform
that has none yet, or a pull request that takes one of these bridges into its platform's own
repository.

## Licence

Apache-2.0 for this repository. Each bridge carries its own licence; see the table above.
Copyright 2026 deliberAIde.
