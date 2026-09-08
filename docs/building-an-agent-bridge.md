# Building an agent bridge

A bridge is a command-line client for one platform, shaped so that an AI agent can drive it
without a wrapper. This is what the four existing bridges have in common and how to get a
working one for your platform.

## The shape

```
<tool> profile add <name> <base-url> --auth <method>   # name an instance once
<tool> --json <group> <operation> [options]            # one command per API operation
<tool> --json export <resource> --out <path>           # bulk read
```

Three properties make it usable by an agent:

**Every command answers in one envelope shape.** With `--json`, the output is a single object
that says what was requested, what came back and whether it worked. An agent never parses prose,
never scrapes a table, and never has to tell an error message from a result.

```json
{"ok": true, "operation": "convo.create", "request": {...}, "response": {"conversation_id": "6iker7ivbh"}}
```

**Instances are named, credentials are not passed on the command line.** A profile holds the
base URL and how to authenticate; the secret comes from the environment or a credential store.
An agent then writes `-p munich` instead of assembling a URL and a token into a command that
would end up in shell history.

**The tool can describe itself.** A command that lists the operations available against *this*
installation, because forks differ. `deliberaide-cli` keeps a generated route manifest;
`consul-cli` discovers the installed routes and models from the running application. An agent
that can ask "what can I do here" recovers from a version mismatch instead of guessing.

## Getting the first version out

**If the platform has an OpenAPI or GraphQL schema, generate.** Point a coding agent at the
schema and have it emit one command per operation, plus the profile and envelope layer by hand
once. `polis-cli` and `deliberaide-cli` are built this way: their command surfaces are
regenerated when the upstream API changes, so drift is a rebuild rather than a rewrite.

**If it does not, wrap what exists.** [CLI-Anything](https://github.com/HKUDS/CLI-Anything) (MIT)
turns an existing software surface into an agent-native CLI and is a reasonable starting point.
`consul-cli` went further because CONSUL does not expose a complete operator API: it pairs a
Python client with a small token-authenticated Rails engine mounted in the application, which
reaches the platform's own models, routes and rake tasks rather than reimplementing them. That
engine is AGPL-3.0, like its host.

**Budget about a day for a first useful version, then weeks of edges.** In one timed run,
building a read-and-write Decidim CLI this way took 49 minutes to a working, tested state. What
takes the remaining time is headless authentication, pagination, schema differences between
versions, and the behaviours you only discover by running against a real installation. The
repository tells you what exists; only a live server tells you how to call it.

## Rules worth keeping

- **Do not fork the platform.** A bridge that only works against a patched installation is hard
  for anyone else to adopt. Where in-application code is unavoidable, make it a mountable engine
  or plugin under the platform's own licence, and keep it as small as it can be.
- **Never invent domain behaviour.** A bridge calls the platform's own logic. If the CLI
  implements a validation the platform does not have, the two will disagree eventually, and the
  platform is right.
- **Test against a real installation.** Mocks confirm you understood the documentation. A
  running instance tells you what the API actually does, which is the whole point of the bridge.
- **Make every write reversible or dry-runnable.** Agents retry. `--dry-run` that prints the
  exact request, and idempotent behaviour where the API allows it.
- **Keep secrets out of the repository and out of arguments.** Environment variables and
  profiles, never a flag value.

## Then offer it upstream

A bridge maintained by one organisation is a dependency. A bridge in the platform's own
repository is infrastructure. Licence the client permissively so the platform can adopt it
without a licence conflict, and open the pull request.

See [Designing an API an agent can drive](api-design-for-agent-bridges.md) for the other half:
which API choices made the bridges easy or hard to build.
