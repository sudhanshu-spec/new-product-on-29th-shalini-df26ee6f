# Node.js Tutorial: Main Branch Changes

A minimal, dependency-free Node.js HTTP server written for beginners. It demonstrates one endpoint, `GET /hello`, and answers every other request with a `404` catch-all. The server is built on Node.js's built-in `node:http` module — no framework, no middleware and no dependencies — and the whole application is one readable file, because readability is a requirement of this project rather than a side effect of it.

## Prerequisites

- **Node.js v18.0.0 or newer.**
- **`curl`, for the testing examples below** — the Testing and Expected-output sections issue `curl` commands. It ships by default on macOS, Windows 10 and later, and most Linux distributions, and any HTTP client may be used in its place. The server itself needs nothing beyond Node.js.

The version floor lives in two places and the two must agree: `"engines": { "node": ">=18.0.0" }` in `package.json`, and this section. npm treats `engines` as advisory, so treat this section as the promise you can rely on.

Check the version you have:

```bash
node --version
```

```text
v22.23.4
```

Node.js 18 reached end-of-life on 2025-04-30 and Node.js 20 on 2026-04-30, so Node 22 — in maintenance until 2027-04-30 — or the current LTS release is the better choice today. The declared floor still runs the tutorial unchanged.

## Installation

None beyond Node.js itself. The server uses only Node.js built-in modules, so there is no `npm install` step, no `node_modules` directory and no lockfile. The project is three files, and they run as they are.

## Execution

Start the server from the project root:

```bash
node server.js
```

The same server starts through the `start` script declared in `package.json`:

```bash
npm start
```

Either way it prints:

```text
Server listening on port 3000
```

On the `npm start` path, npm prints its own banner lines first, so a fresh start shows:

```text
> nodejs-tutorial-hello-world@1.0.0 start
> node server.js

Server listening on port 3000
```

The process then stays in the foreground and serves requests until you stop it (see Shutting down). It writes nothing to disk while it runs.

## Configuration

The server reads one environment variable, `PORT`.

| Property | Value |
| --- | --- |
| Default | `3000`, used when `PORT` is unset or empty |
| Accepted values | Digits only, `1024` to `65535` inclusive |
| Leading zeros | Accepted — `03000` is port 3000 |
| Rejected | Anything else: a sign, whitespace, a decimal point or any other non-digit character |

Examples:

```bash
PORT=4000 node server.js     # Server listening on port 4000
PORT=4000 npm start          # npm's banner lines, then Server listening on port 4000
```

An invalid value is reported on stderr and the process exits with status `1` — no port is bound and no startup line is printed:

```bash
PORT=abc node server.js; echo $?
```

```text
Invalid PORT value "abc": PORT must be an integer between 1024 and 65535. Unset PORT to use the default 3000.
1
```

```bash
PORT=80 node server.js; echo $?
```

```text
Port 80 is below the minimum: ports 1-1023 are privileged and require elevated access. Set PORT to a value between 1024 and 65535, or unset PORT to use the default 3000.
1
```

```bash
PORT=70000 node server.js; echo $?
```

```text
Port 70000 is above the maximum: PORT must be between 1024 and 65535. Unset PORT to use the default 3000.
1
```

Ports 1-1023 are privileged, so the server refuses them through its own validation before it asks the operating system to bind anything.

A port that another process already holds fails the same way:

```bash
node server.js; echo $?
```

```text
Port 3000 is already in use (EADDRINUSE). Stop the process using that port, or set PORT to a free port.
1
```

## Testing

With the server running on the default port, these four commands exercise the whole contract:

```bash
curl http://localhost:3000/hello            # Hello world
curl -i http://localhost:3000/hello         # 200, Content-Type: text/plain, Content-Length: 11
curl http://localhost:3000/invalid          # Not Found, status 404
curl -X POST http://localhost:3000/hello    # Not Found, status 404
```

The third and fourth commands both answer **404**: `/invalid` is not a route, and `POST /hello` matches the path but not the method, so the catch-all answers it. Add `-i` to either command to see the status line; the full responses are shown in Expected output examples.

## Expected output examples

A terminal running the server, with three requests issued from a second terminal:

```bash
curl -i http://localhost:3000/hello
curl -i http://localhost:3000/invalid
curl -i -X POST http://localhost:3000/hello
```

```text
Server listening on port 3000
2026-10-29T13:41:17.742Z GET /hello
2026-10-29T13:41:17.751Z GET /invalid
2026-10-29T13:41:17.759Z POST /hello
```

Every request produces exactly one line, including the requests answered with 404. Each line has three fields separated by single spaces:

1. an ISO-8601 UTC timestamp from `new Date().toISOString()` — millisecond precision, trailing `Z`;
2. the HTTP method exactly as received;
3. the request path the router matched — the request target cut at the first `?`, so `GET /hello?name=test` logs `GET /hello`.

The responses, header for header:

```bash
curl -i http://localhost:3000/hello
```

```text
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 11
Date: Thu, 29 Oct 2026 13:41:17 GMT
Connection: keep-alive
Keep-Alive: timeout=5

Hello world
```

```bash
curl -i http://localhost:3000/invalid
```

```text
HTTP/1.1 404 Not Found
Content-Type: text/plain
Content-Length: 9
Date: Thu, 29 Oct 2026 13:41:17 GMT
Connection: keep-alive
Keep-Alive: timeout=5

Not Found
```

The application sets `Content-Type: text/plain` (with no `charset` parameter) and `Content-Length`. `Date`, `Connection` and `Keep-Alive` are added by Node.js itself and are not part of the contract.

## Shutting down

Press `CTRL+C` — SIGINT, the only signal the server handles. It prints:

```text
Shutting down server on port 3000
```

then stops accepting connections, closes its connections cleanly and exits with status `0`. The port in the line is the port in use, so with `PORT=4000` the line reads `Shutting down server on port 4000`.

The wait is bounded: the server gives in-flight requests up to one second to finish, then closes whatever is still connected and exits with status `0` all the same, so a client that opens a connection and never finishes its request cannot keep the process running. Pressing `CTRL+C` a second time during that second stops the server immediately, without a second shutdown line.

A complete session therefore looks like this:

```text
Server listening on port 3000
2026-10-29T13:41:17.742Z GET /hello
Shutting down server on port 3000
```

## Project structure

```text
server.js      Main application entry point: the complete HTTP server
package.json   NPM project metadata (name, version, description, main, start script, engines, keywords)
README.md      Execution and usage instructions (this file)
```

## Endpoint reference

| Method | Path | Status | Body |
| --- | --- | --- | --- |
| `GET` | `/hello` | `200` | `Hello world` |
| Any | Any (`*` catch-all) | `404` | `Not Found` |

Row 2 is the catch-all: it answers every request that row 1 does not match — any method on any path, so `GET /invalid` (a `GET` on an unknown path) and `POST /hello` (another method on `/hello`) both return `404`. A `HEAD` request carries the status and headers of its routing decision but no body bytes, because Node.js suppresses the body of every `HEAD` response: `curl -I http://localhost:3000/hello` therefore shows `404`, `Content-Type: text/plain` and `Content-Length: 9` with no `Not Found`.

Both responses carry `Content-Type: text/plain`. `Hello world` is 11 bytes and `Not Found` is 9 bytes, neither with a trailing newline. Requests are logged before routing, one line each, in the format `<ISO-8601 timestamp> <HTTP method> <request path>`, where the timestamps are UTC.
