# Uniqorn: Context, Philosophy, and Boundaries

Guidance for an AI agent that will **use Uniqorn as a backend**: building on it, integrating
with it, or deciding whether it fits a problem.

This is the *why and the shape*. For the endpoint-writing API surface (classes, validators,
code skeleton, gotchas), read [`BACKEND.md`](BACKEND.md) alongside this file. For exposing live
endpoints as agent tools, read [`MCP.md`](MCP.md).

---

## 1. What Aeonics is

**Aeonics** is a zero-dependency, pure-Java, plugin-based application framework. Not "few
dependencies" — zero. No Maven, no Gradle, no Spring, no Jackson, no Netty. It is built with
plain `javac` and `jar`, targets Java 11 source, and uses Java 9+ modules (`ModuleLayer` +
`ServiceLoader`) to load plugins.

Everything it needs, it owns: the HTTP/1.1 + WebSocket server is hand-rolled on NIO, the JSON
layer is hand-rolled, JWT/OIDC is hand-rolled, the Git smart-HTTP server is written from
scratch, and runtime Java compilation goes through the JDK's own `javac` API.

Core abstractions, in the order you meet them:

| Concept | Role |
|---|---|
| `Entity` / `Template` / `Item` | The universal object model. Everything configurable is an Entity built from a Template. |
| `Registry` | Category-keyed store of live entities. |
| `Manager` | Singleton cross-cutting services: Config, Logger, Executor, Scheduler, Security, Network, Snapshot. |
| `Data` | Zero-dep JSON-like value container (value / list / map). The lingua franca. |
| `Step` / flow | Event and processing pipeline. |

**Aeonics is the framework. You almost certainly will not touch it.** It is the layer Uniqorn
is built on, and it is deliberately not the layer users are invited into.

## 2. What Uniqorn is

**Uniqorn** is the productized SaaS on top of Aeonics, and it collapses to one sentence:

> Write a Java file. `git push`. It is a live REST endpoint, automatically exposed to AI agents
> over MCP.

There is no container to build, no pipeline to configure, no YAML, no deploy step. **The push
*is* the deployment.** A push compiles the file, publishes the endpoint, and returns
machine-readable per-file feedback on the same connection. Compile-to-live is ~87 ms; the
end-to-end loop is seconds.

Components:

- **uniqorn.instance** — the per-tenant unit (Docker container): admin panel, built-in Git
  server, local SQLite, object storage, the endpoints themselves.
- **uniqorn.gateway** — TLS termination, certbot/Let's Encrypt, connection pooling.
- **uniqorn.manager** — control plane: signup, Stripe billing, Docker provisioning, lifecycle
  crons.

Scale of the whole thing: the entire release is **1.2 MB**, frontend to backend. The
user-facing API is ~10 classes, and it serves ~15k rps.

## 3. Philosophy

These are not decorative values. They predict what Uniqorn will and will not do, so an agent
that internalizes them will guess right about the platform far more often.

**Remove, don't hide.** The industry's two moves when facing complexity are to hide it (a
framework, an abstraction, an operator, a control plane) or to remove it. Hiding is the better
business model; removing is what earns trust. Docker and Kubernetes are hide-the-mess
philosophy, and Uniqorn rejects that. When Uniqorn faces a choice between wrapping something
and deleting the need for it, it deletes.

**The part you actually need is small enough to own.** Read the RFCs and you find the leading
implementations already skip half the spec and nobody misses it. That observation is the whole
zero-dependency thesis: it is not Not-Invented-Here, it is a measured claim that the necessary
subset is small. Zero dependencies is the *single decision*; supply-chain safety, security
surface, sovereignty, simplicity, and agent-friendliness are all downstream consequences of it.

**Explicit over implicit.** No convention-based discovery, no auto-registration, no "it happens
because the file exists," no annotation scanning. Registration is an explicit, visible,
diff-readable list. Behavior triggered by mere existence is invisible and cannot be selectively
controlled. If you are tempted to make something work automatically because it is *there*,
that is the wrong instinct here — propose the explicit variant.

**Sovereignty is control, not geography.** "Your servers, your rules" is jurisdiction-agnostic.
Do **not** frame Uniqorn as an EU-only or GDPR product; residency and CLOUD Act are a quiet FAQ
line, not the pitch. The real claim is that exit is a `git clone` and it runs on your own
hardware anywhere.

**Bounded surface serves humans and agents identically.** The ~10-class API, the one-file
endpoint, and the closed action space were designed for human legibility and turn out to be
exactly what makes the platform agent-native. A model cannot hallucinate an import that does
not exist when the surface is small enough to hold entirely in context.

**Everything is a decision.** Java was chosen because it forces choices to be visible rather
than magical. Uniqorn inherits that taste. Prefer the boring, explicit, inspectable path.

## 4. Goals

- Make the code→push→live loop the shortest possible: **push a file, ship an API.** The push is
  the irreducible human act; AI writes, human pushes.
- Keep the user-facing surface small enough to learn in one sitting and hold in one context
  window.
- Be genuinely agent-native at the endpoint layer: bounded single-file context, closed and
  enforced action space, short machine-readable feedback loop, endpoint metadata that doubles
  as an MCP tool schema.
- Zero lock-in. Exit is a `git clone`. Self-hosting is a first-class deployment, not a
  downgrade.
- Predictable flat billing that replaces an infrastructure stack, not a metered surprise.
- Every push is a version; any version restores instantly.

## 5. Non-goals — read this section twice

Most integration mistakes come from assuming Uniqorn wants to be something it has explicitly
refused to be.

**It is not a general-purpose application framework, and does not want to be.** If the logic
does not fit the one-file endpoint model, the correct answer is *"use Spring. Genuinely."* Do
not try to bend Uniqorn into an application server. Suggest a better structure and stop.

**There is no shared code across endpoints.** No cross-file imports, no common library module,
no utility package. One file is one endpoint, completely standalone. Share state via
`State.global()` or call across endpoints with `Api.chain(url, method, data)`. This is a
positioning constraint, not an oversight — do not design around it by generating a "utils"
endpoint.

**The sandbox is not a security boundary.** This matters and the public docs overstate it. The
restriction is a **type check on compiled bytecode**: after `javac` succeeds but before the
class is loaded, the constant pool is read to collect every class the bytecode actually
references, and each is matched against a denied-package prefix list and a denied-class set.
A violation fails the deploy with `422 Use of restricted type: <class>`. It is enforced **only
on the hosted trial and personal tiers** — Team, Enterprise, custom, and self-hosted instances
have no restriction at all. Its real value is as a *generation constraint* and a UX guardrail
that steers tier-1 users into the intended model. The actual isolation is the per-instance JVM
and container. Once user code executes, it is inside the trust boundary. Consequences for you:
  - Do not describe endpoints as "safe by construction" or "blocked at compile time" as if that
    were a security guarantee.
  - **Identifier names are never matched.** The check sees only fully-qualified type names from
    the constant pool, so a variable called `paymentMethod`, a comment mentioning `URL`, or a
    call to `getClass()` will not trip it. Write idiomatic Java.
  - It can still flag a type you never named, because the constant pool includes types from the
    signatures of members you reference. `java.io.Serializable` in particular is broad. The
    error names the offending class — read it literally.
  - Only the **first** offending class is reported, and which one is nondeterministic when
    several are present. Fix, re-push, repeat.
  - Runaway compute and infinite loops are deliberately **out of scope**: they are self-harm
    within one tenant's own instance, with no lateral movement.

**It is not built for world scale.** The target is on-prem standalone or small on-site clusters
— roughly a dozen instances, not a global fleet. Availability and backups are delegated to the
VPS provider. Do not propose architectures that assume multi-region orchestration.

**Test coverage is handled independently.** The internal framework test suite holds a zero-dependency 
harness and core suites but this is not available for developers.
For your own work, the push output remains the practical oracle — push, read the JSON line, call the
URI, verify the response.

**The frontend toolbox stays minimal.** `aeonics.frontend` deliberately has no reactivity, no
route guards, no component library. Adding them would make it the framework Aeonics does not
want to own. If a project outgrows it, graduate to React or Vue — that is the sanctioned path,
not a failure.

**No build system will be added.** No Maven, no Gradle, no npm, no bundler, no transpiler.

**`.returns()` is prose, not a schema.** It documents the response for humans and agents; it
does not validate or generate types. Do not treat it as a contract the runtime enforces.

## 6. Target repo structure

The user's repo is cloned from and pushed to the instance's built-in Git server. Only two
top-level folders carry meaning.

```
/
├── README.md          # allowed, never processed
├── src/               # Java endpoints — compiled and published on push
│   ├── hello/         # workspace = URL prefix
│   │   ├── world.java
│   │   └── greet.java
│   └── billing/
│       └── invoice.java
└── www/               # static assets — extracted and served on push
    ├── index.html
    ├── app.js
    └── assets/
        └── logo.svg
```

### `src/` — exactly `/src/{workspace}/{name}.java`

- **Only `.java` files.** A `.md`, `.json`, or `.txt` under `src/` is silently ignored.
- **Exactly two path components** below `src/`. No subfolders inside a workspace, and no
  `.java` sitting directly in `src/`. `/src/a/b/c.java` is ignored.
- **`{workspace}`** is the folder name. It becomes the URL prefix and is created automatically
  on first push (subject to a plan quota). Charset is `[A-Za-z0-9-_.]` only.
- **`{name}.java`** — the file name is **irrelevant to the URL**. It is just a name.
- **The URL comes from the code**, not the path: `new Api("/api/example", "GET")`.

  Final URL = global prefix (default `/upi`) + `/{workspace}` + declared path.

  So `/src/hello/world.java` declaring `new Api("/world", "GET")` serves `GET /upi/hello/world`.

- The `package` declaration is stripped before compilation and a fixed import block is
  prepended — which is why `import uniqorn.*;` works and why a `package` line is pointless.
- Delete the file → the endpoint is unpublished. Change the file → it recompiles only if the
  SHA changed. **A compile failure on update keeps the previous version live.**

### `www/` — anything, any depth

- Any extension, any nesting. Extracted to instance storage on push; deleting a file removes it
  from the server. Only directory entries and `..` traversal are rejected.
- Served as a **404 fallback**, not a mounted prefix: a request that matches no endpoint falls
  through to static serving.
- `/` and any trailing-slash URL resolve to `index.html`. `/foo` falls back to `/foo.html`. A
  directory containing `index.html` gets a 301 to its trailing-slash form. A final miss serves
  `/404.html`.
- **Reserved prefixes bypass static serving entirely** and 404 normally: `/ae`, `/oauth`,
  `/panel`, `/.well-known`. Do not put assets there.
- No templating. Bytes are returned verbatim, MIME guessed from the path.

### Everything else

Files outside `src/` and `www/` are accepted by Git and **never processed**. Keep your README,
notes, and tooling wherever you like.

### Push feedback

Only the **default branch** matters — a push with no update to it deploys nothing and says so.
Each processed file returns one machine-readable line, then a summary:

```
remote: [@Uniqorn] {"file": "/src/hello/world.java", "uri": "GET /upi/hello/world", "status": "updated", "error": null, "info": null}
remote: ---------------
remote: Done: created=0 updated=1 removed=0 ignored=0 error=0 in 133ms
remote: ---------------
```

`status` is one of `created` / `updated` / `removed` / `ignored` / `error`. Parse this. On a
compile error, the `error` field carries real `javac` diagnostics — use them to fix and push
again. On `ignored`, the `info` field tells you which layout rule you broke.

## 7. Working style when building on Uniqorn

- **Read the push output.** It is the test oracle. There is no test framework — the loop is
  push, read the JSON line, call the URI, verify the response.
- **Keep endpoints at 10–20 lines of business logic.** The framework owns security, storage,
  networking, compilation, and deployment. You own the intent. If it does not fit, say so
  rather than inventing scaffolding.
- **Watch for runtime coupling the one-file model hides.** `State.global()` and `Api.chain()`
  create dependencies between endpoints that are invisible when reviewing a single file. Note
  them explicitly.
- **`Api.atomic()` is instance-wide** — it locks every endpoint, not just yours. Keep blocks
  short.
- **`State` does not survive a reboot**, and `State.local` is orphaned on redeploy. Use
  `Api.storage()` or `Api.database()` for anything durable.
- **Document every endpoint fully** — `.summary()`, `.description()`, `.parameter()` with a
  description, `.returns()`. This is not politeness: it is what becomes the MCP tool schema
  that lets agents discover and call the endpoint.
- **Be honest in generated copy.** No em-dashes in user-facing marketing text, no overclaiming
  the sandbox, no "safe by construction."
