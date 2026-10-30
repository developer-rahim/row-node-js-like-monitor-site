# Uptime Monitor — Node.js (zero-framework)

A self-hosted **uptime monitoring service** written in **plain Node.js with no web framework**.
Instead of Express or Fastify, the project builds its own HTTP layer on Node's native `http`
module — including a custom router, a file-based JSON CRUD store, a background health-check
worker, and Twilio SMS notifications when a monitored service goes down.

It is both a working monitor and a study in how the Node.js HTTP layer actually works under the hood.

---

## Highlights

- **No framework** — request handling, routing and response writing are done directly against
  Node's `http` module, so every layer is visible.
- **Custom router** — URLs are parsed manually, trimmed, matched against a route table, and
  dispatched with a not-found handler for unmatched paths.
- **Health-check worker** — a background loop probes each registered URL, records `up` / `down`
  state, and triggers an alert on state change.
- **Twilio SMS alerts** — downtime notifications sent through the Twilio REST API using
  `https` and `querystring` directly.
- **File-based CRUD** — monitor records persist as JSON on disk, so there is no external database
  dependency.
- **Environment-aware configuration** — separate `local` and `production` profiles selected via
  `NODE_ENV`.

---

## Project Structure

```
row_node_project/
├── index.js                 # Entry point — bootstraps server + worker
├── data_crud/
│   ├── data.js              # JSON file CRUD (create / read / update / delete)
│   ├── server.js            # HTTP server bootstrap
│   └── worker.js            # Health-check worker loop
├── helper/
│   ├── handle_req_res.js    # Request parsing, routing dispatch, response writing
│   ├── env_setup.js         # Environment configuration
│   ├── notification.js      # Twilio SMS sender
│   └── utiles.js            # Shared utilities (parseJson, etc.)
├── routes/
│   ├── routes_name/         # Route name -> handler mapping
│   └── route_handler/       # Individual route handlers + not_found_handler
└── type_script/             # TypeScript declarations
```

Supporting folders at the repository root mirror this layout for experimentation:
`module/http_module`, `module/event_module`, `project_structure/lib` and `steam_buffer`.

---

## How It Works

### Entry point

```js
const http = require('http');
const { handleReqRes } = require('./helper/handle_req_res');
const environment = require('./helper/env_setup');
const crudFile = require('./data_crud/data');
const twilio = require('./helper/notification');
const server = require('./data_crud/server');
const worker = require('./data_crud/worker');

const app = {};
app._init = () => {
    server.init();
    worker.init();
};
app._init();
```

### Request handling

```js
const parseUrl = new URL(req.url, `http://${req.headers.host}`);
const pathName = parseUrl.pathname;
const trimedPath = pathName.replace(/^\/+|\/+$/g, '');
const reqMethod = req.method.toLowerCase();
```

Query parameters are read from `parseUrl.searchParams`, and headers come straight off
`req.headers`.

### Data storage

`data_crud/data.js` writes monitor records as JSON files under a base directory, exposing
`create`, `read`, `update` and `delete` operations with a callback-style API.

---

## Getting Started

### Prerequisites

- Node.js `>= 16`

### Setup

```bash
git clone https://github.com/developer-rahim/row-node-js-like-monitor-site.git
cd row-node-js-like-monitor-site
cd row_node_project
npm install
```

### Run

```bash
# local (port 3000)
npm start

# production (port 5001)
npm run production
```

---

## ⚠️ Security Notice

This repository stores secrets directly in `helper/env_setup.js`. **Before running this project,
replace those hardcoded values with environment variables:**

```js
const twilio = {
    messagingServiceSID: process.env.TWILIO_MESSAGING_SERVICE_SID,
    accountSID: process.env.TWILIO_ACCOUNT_SID,
    accountToken: process.env.TWILIO_AUTH_TOKEN,
};
```

The credentials committed to this repository should be treated as compromised and **rotated in the
Twilio console**. The same applies to any `secretKey` values stored in that file.

---

## License

Released under the MIT License.
