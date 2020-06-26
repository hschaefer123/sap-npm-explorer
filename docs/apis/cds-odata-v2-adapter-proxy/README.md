# cds-odata-v2-adapter-proxy

OData v2 Adapter Proxy for CDS OData v4 Services

## Build Status

[![Build Status](https://gkecdxodata.jaas-gcp.cloud.sap.corp/buildStatus/icon?job=cds-community%2Fcds-odata-v2-adapter-proxy%2Fmaster)](https://gkecdxodata.jaas-gcp.cloud.sap.corp/job/cds-community/job/cds-odata-v2-adapter-proxy/job/master/)

## Getting Started

- Install: `npm install @sap/cds-odata-v2-adapter-proxy`
- Unit Tests: `npm test`
- Test Server: `npm start`
  - Service: `http://localhost:4004/v2/main`
  - Metadata: `http://localhost:4004/v2/main/$metadata`
  - Data: `http://localhost:4004/v2/main/Header?$expand=Items`

## Usage

### CDS combined backend (Node.js) - integrated

In your existing `@sap/cds` project:

- Run `npm install @sap/cds-odata-v2-adapter-proxy -s`
- Create new file `server.js` in the service folder `srv` of your project: `./srv/server.js`

```
"use strict";

const cds = require("@sap/cds");
const proxy = require("@sap/cds-odata-v2-adapter-proxy");

cds.on("bootstrap", app => app.use(proxy()));

module.exports = cds.server;
```

- Run `cds run` from the project root to start the server:
  - OData v2 service will be available at http://localhost:4004/v2/<service-path>
  - OData v4 service will be available at http://localhost:4004/<service-path>

Note that `@sap/cds` and `express` are peer dependency and needs to be available as module as well.

### CDS combined backend (Node.js) - custom

In your existing `@sap/cds` project:

- Run `npm install @sap/cds-odata-v2-adapter-proxy -s`
- Create new file `index.js` in the service folder `srv` of your project: `./srv/index.js`

```
"use strict";

const express = require("express");
const cds = require("@sap/cds");
const proxy = require("@sap/cds-odata-v2-adapter-proxy");

const host = "0.0.0.0";
const port = process.env.PORT || 4004;

(async () => {
  const app = express();

  // serve odata v4
  await cds
    .connect("db") // ensure database is connected!
    .serve("all")
    .in(app);

  // serve odata v2
  process.env.XS_APP_LOG_LEVEL = "warning";
  app.use(proxy({
    path: "v2",
    port: port
  }));

  // start server
  const server = app.listen(port, host, () => console.info(`app is listing at ${host}:${port}`));
  server.on("error", error => console.error(error.stack));
})();
```

- Run `node srv/index` from the project root to start the server:
  - OData v2 service will be available at http://localhost:4004/v2/<service-path>
  - OData v4 service will be available at http://localhost:4004/<service-path>

Note that `@sap/cds` and `express` are peer dependency and needs to be available as module as well.

### CDS standalone backend (e.g. Java)

In a new Node.js express project:

- Run `npm install @sap/cds -s`
- Run `npm install @sap/cds-odata-v2-adapter-proxy -s`
- Place CDS models in `db` and `srv` model folders
- Create new file `index.js` in the service folder `srv` of the project: `./srv/index.js`

```
"use strict";

const express = require("express");
const http = require("http");

const proxy = require("@sap/cds-odata-v2-adapter-proxy");

const host = "0.0.0.0";
const port = process.env.PORT || 4004;

(async () => {
  const app = express();

  // serve odata v2
  process.env.XS_APP_LOG_LEVEL = "warning";
  app.use(proxy({
    path: "v2",
    target: "http://localhost:8080",
    services: {
      "<odata-v4-service-path>": "<qualified.ServiceName>"
    }
  }));

  // start server
  const server = app.listen(port, host, () => console.info(`app is listing at ${host}:${port}`));
  server.on("error", error => console.error(error.stack));
})();
```

- Make sure, that your CDS models are also available in the project.
  Those reside either in `db` and `srv` folders, or a compiled (untransformed) `srv.json` is provided.
  This can be generated by using the following command:
  ```
  cds srv -s all -o .
  ```
- If not detected automatically, the model path can be set with option `model`
  (especially if `srv.json` option is used).
- Make sure, that all i18n property files reside next to the `srv.json` in a `i18n` or `_i18n` folder, to be detected by localization.
- Run `node srv/index` from the project root to start the server:
  - OData v2 service will be available at http://localhost:4004/v2/<odata-v4-service-path>
  - OData v4 service shall be available at http://localhost:8080/<odata-v4-service-path>
- A deployed version of OData v2 proxy shall have option `target` set to the deployed OData v4 backend URL.
  This can be retrieved from the CF environment using [node-xsenv](https://github.wdf.sap.corp/xs2/node-xsenv) module, e.g.
  from the `destinations` environment variable.

Note that `@sap/cds` and `express` are peer dependency and needs to be available as module as well.

## Cloud Foundry Deployment

When deploying the CDS OData v2 Adapter Proxy to CF, make sure that it has access to the whole CDS model.
Especially, it is the case, that normally the Node.js server is only based on folder `srv` and folder `db` is then missing on CF.

To come around this situation, trigger a `cds build`, that generates a `csn.json` at location `gen/srv/srv/csn.json`.
If your CF deployment of the Node.js backend (incl. CDS OData v2 Adapter Proxy) is then based on folder `gen/srv`,
the CDS models can be found during runtime on Cloud Foundry.

## Documentation

Instantiates an CDS OData v2 Adapter Proxy Express Router for a CDS based OData v4 Server

- **options:** CDS OData v2 Adapter Proxy options
  - **[options.base]:** Base path, under which the service is reachable. Default is ''.
  - **[options.path]:** Path, under which the proxy is reachable. Default is 'v2'.
  - **[options.model]:** CDS service model (path(s) or CSN). Default is 'all'.
  - **[options.port]:** Target port, which points to OData v4 backend port. Default is '4004'.
  - **[options.target]:** Target, which points to OData v4 backend host/port. Default is 'http://localhost:4004'.
  - **[options.services]:** Service mapping, from url path name to service name. If omitted local CDS defaults apply.
  - **[options.standalone]:** Indication, that OData v2 Adapter proxy is a standalone process. Default is 'false'.
  - **[options.mtxEndpoint]:** Endpoint to retrieve MTX metadata for standalone proxy. Default is '/mtx/v1'
  - **[options.ieee754Compatible]:** Edm.Decimal and Edm.Int64 are serialized IEEE754 compatible. Default is 'true'.
  - **[options.pathRewrite]:** Custom path rewrite rules. Default uses 'path' option as rule: { "^/v2": "" }
  - **[options.disableNetworkLog]:** Disable networking logging. Default is 'true'.

Logging is controlled with XSA environment variable `XS_APP_LOG_LEVEL`.
Details can be found at [xs2/node-logging](https://github.wdf.sap.corp/xs2).

## Features

- GET, POST, PUT/PATCH, DELETE
- Batch support
- Actions, Functions
- Analytical Annotations
- Deep Expands/Selects
- JSON format
- Deep Structures
- Data Type Mapping (incl. Date Time)
- IEEE754Compatible
- Messages/Error Handling
- Location Header
- $inlinecount / $count / \$value
- Entity with Parameters
- Stream Support (Octet and Url), Content Disposition
- Multitenancy, Extensibility (proxy in same process only)
- Content-ID
- Draft Support
- Search Support
- Localization
- Tracing
- Logging Correlation
- ETag Support (Concurrency Control)

## OData v2/v4 Delta

http://docs.oasis-open.org/odata/new-in-odata/v4.0/cn01/new-in-odata-v4.0-cn01.html

**Open:**

- $links -> $ref
- KEY(...) -> \$root
- years, months, days, minutes, seconds
- Function Parameters as Query Options