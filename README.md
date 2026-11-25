# Serverless Node Express API

A small **Express** API deployed to **AWS Lambda** with the **Serverless Framework (v3)**.
`serverless-http` wraps the Express app, so all routing happens in Express.

| Method | Path | Response |
|---|---|---|
| GET | `/` | `{"message":"Hello from root!"}` |
| GET | `/hello` | `{"message":"Hello from path!"}` |
| any | anything else | `404 {"error":"Not Found"}` |

## Tech stack

- Node.js + Express 4
- `serverless-http`
- Serverless Framework 3 with `serverless-offline` (local) and `serverless-dotenv-plugin`
- AWS Lambda (`nodejs20.x`) + HTTP API

## Requirements

| Requirement | Version | Notes |
|---|---|---|
| **OS** | **Linux** (Ubuntu / Debian) | Use Linux to install, run and deploy. On Windows, use WSL 2 (Ubuntu); see [Other operating systems](#other-operating-systems) |
| Node.js | 20.17 or newer | Matches the Lambda runtime (`nodejs20.x`). Tested on 20 and 26. Install with [nvm](https://github.com/nvm-sh/nvm): `nvm install 20` |
| npm | Comes with Node.js | |
| AWS account + credentials | Only for deploying | Not needed to run locally |

Serverless Framework v3 is installed locally by `npm ci`; you don't need a global `serverless` install.

### Tested on

| OS | Node.js | Result |
|---|---|---|
| **Linux** (Debian 12, Docker `node:20`) | 20.20 | `npm run dev` serves both routes ✅ |
| macOS 26 (Apple Silicon) | 26.12 | `npm run dev` serves both routes ✅ |

## Run locally

```bash
git clone https://github.com/nawarajsubedi/serverless-node-api.git
cd serverless-node-api
npm ci
npm run dev
```

The API is available at <http://localhost:3000>:

```bash
curl http://localhost:3000/hello
# {"message":"Hello from path!"}
```

`serverless-offline` also opens port 3002 for local Lambda invocation. Running locally needs no AWS credentials and no Serverless login.

### Environment variables (optional)

`serverless-dotenv-plugin` loads a `.env` file from the project root if there is one.
The message `DOTENV: Could not find .env file.` is harmless.

## Deploy to AWS

```bash
npm run deploy       # serverless deploy --stage prod --region ap-northeast-1
```

To deploy you need:

1. AWS credentials, for example from `aws configure` or the `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` environment variables.
2. A Serverless Framework login, because `serverless.yml` sets `org` and `app`: `npx serverless login`.
   To deploy without the Serverless Dashboard, remove the `org:` and `app:` lines from `serverless.yml`.

After deploying, the output shows the HTTP API URL.

## Other operating systems

The supported OS is **Linux**. The commands above were written and tested for a Linux shell.

- **Windows**: install [WSL 2](https://learn.microsoft.com/windows/wsl/install) with Ubuntu, open the Ubuntu terminal, and follow the Linux steps exactly.
- **macOS**: the same commands work in Terminal (tested on macOS 26).

## Project structure

```
handler.js        Express app wrapped with serverless-http
serverless.yml    Service, provider (aws, nodejs20.x), function and plugins
```

## Troubleshooting

- **`TypeError: Cannot redefine property: _serverlessExternalPluginName`**: Serverless v3 can't load the ESM-only
  `serverless-offline` plugin on newer Node versions (Node 20.19+ and 22.12+ load ESM through `require`).
  `npm run dev` already passes `--no-experimental-require-module` to fix this.
  If you call `serverless offline` directly, run it as
  `node --no-experimental-require-module node_modules/serverless/bin/serverless.js offline`.
- **`EADDRINUSE` on port 3000**: another app is using the port. Use `npm run dev -- --httpPort 4000`.
