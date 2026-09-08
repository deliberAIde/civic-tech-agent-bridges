# Designing an API an agent can drive

Notes from building bridges for four participation platforms. None of this is exotic; it is
mostly ordinary good API practice, with a bias toward the things that break agents specifically.

## The short version

An agent is a client that has never read your documentation the way a human would, cannot see
your interface, and will not guess correctly twice. It needs three things from an API: a way to
find out what operations exist, a predictable shape for every response, and errors that say what
to do next.

## What makes a bridge easy to build

**A machine-readable description of the surface.** An OpenAPI or GraphQL schema turns most of a
bridge into generated code. `polis-cli` and `deliberaide-cli` both generate their command layer
from the API description, so a new endpoint upstream becomes a new command without hand-written
glue. Where no description exists, the bridge has to hand-roll each operation, which is where the
work and the drift come from.

**Stable operation names.** If an operation is identified by a name that survives refactors, the
bridge can map one command to one operation and regenerate safely. If operations are identified
only by URL shape, every route change silently breaks clients.

**Nouns that match the domain.** A bridge is easiest when the API exposes the same objects the
platform's own users talk about: a process, a phase, a proposal, a conversation. Bridges that
have to assemble a domain object out of four generic endpoints are brittle and hard to explain.

**One way to authenticate, and it works headlessly.** Token or client-credentials authentication
that a machine can obtain without a browser. Interactive-only login pushes bridges into
scripted form posts and session cookies, which is what `consul-cli` had to do for CONSUL and it
is by far the most fragile part of that bridge.

## What makes a bridge safe to run

**Say what an operation did.** Return the created or changed object, with its identifier, not
just `204 No Content`. An agent that has to re-query to find out what it just made will
sometimes find the wrong thing.

**Fail with a reason, not a status.** `422` with a body that names the field and the constraint
lets an agent correct itself. `500 Internal Server Error` with an HTML page ends the run. This
matters more for agents than for people, because a person reads the logs and an agent reads the
response.

**Make writes idempotent, or make repeats detectable.** Agents retry. An idempotency key, or a
uniqueness constraint that returns the existing object rather than a duplicate, prevents an
interrupted run from leaving two of everything.

**Be honest about asynchronous work.** If a request starts a job, return a handle and a way to
poll it. Bridges that have to guess at completion by sleeping produce demos that work in
rehearsal and fail on stage.

**Do not tie a long-running job to the client's connection.** If the client disconnects and the
work is discarded with no trace, every network hiccup costs the user their result. Persist the
run, let a reconnecting client find it.

## What the bridge should offer the agent

These are conventions of the CLI, not of your API, but they are what turns a client into
something an agent can use without a wrapper.

- **`--json` on everything**, with one envelope shape: what was requested, what came back, and
  whether it succeeded. An agent should never have to parse prose.
- **Named profiles** for instances, holding the base URL and the credential reference, so a
  command names the target instead of repeating connection details.
- **Secrets from the environment**, never as flag values that end up in shell history and
  process listings.
- **Exit codes that mean something**, so a script can branch without reading output.
- **A `--dry-run` for anything that writes**, printing exactly what would be sent.
- **Discovery built in**: a command that lists the operations available against *this*
  installation, since forks differ.

## A note on MCP

The same API can be exposed to agents as an MCP server instead of, or in addition to, a CLI. The
design advice above does not change: an MCP tool needs the same discoverability, the same
predictable responses and the same honest errors. A CLI is the cheaper first step, works with
every agent and shell, and is easy for a human operator to run when something has gone wrong.
Which surface wins is not settled, and platforms that keep their API in the shape described here
can offer either.
