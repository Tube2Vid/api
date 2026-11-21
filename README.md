# Tube2Vid API

[![CI](https://github.com/Tube2Vid/api/actions/workflows/ci.yml/badge.svg)](https://github.com/Tube2Vid/api/actions/workflows/ci.yml)
[![OpenAPI 3.1](https://img.shields.io/badge/OpenAPI-3.1-6ba539.svg)](openapi.yaml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

The OpenAPI 3.1 spec for the Tube2Vid REST API: a video downloader API for YouTube, TikTok, Instagram, X, Reddit and 1,800+ other sites. Submit a URL, poll the job, and fetch the MP4, MP3 or image through a short-lived signed link. Use [`openapi.yaml`](openapi.yaml) to generate a client in any language, import it into Postman or Insomnia, or read the rendered reference on [GitHub Pages](https://tube2vid.github.io/api/). The canonical, always-current reference is the [Tube2Vid API documentation](https://tube2vid.com/docs).

Base URL: `https://tube2vid.com`, routes under `/v1`. Downloads are asynchronous: submit, poll, then fetch.

## Authentication

[Get an API key](https://tube2vid.com/pricing) and send it as a bearer token. New keys get the `jobs:write`, `jobs:read` and `usage:read` scopes by default.

```bash
export TUBE2VID_API_KEY=tv_live_...
```

## Download a file

Downloads are asynchronous: submit, poll, then fetch.

```bash
# 1. Submit. Reserves credits and returns a job id.
curl -s https://tube2vid.com/v1/jobs \
  -H "authorization: Bearer $TUBE2VID_API_KEY" \
  -H "content-type: application/json" \
  -d '{"url":"https://www.youtube.com/watch?v=dQw4w9WgXcQ","mode":"auto","format":"mp4","quality":"best"}'
# {"jobId":"3f0c...","status":"queued","platform":"youtube","estimatedCredits":1}

# 2. Poll until status is "completed" or "failed".
curl -s https://tube2vid.com/v1/jobs/$JOB_ID -H "authorization: Bearer $TUBE2VID_API_KEY"

# 3. Fetch. The API redirects to a short-lived signed URL; -L follows it and -OJ keeps the file name.
curl -L -OJ https://tube2vid.com/v1/jobs/$JOB_ID/download -H "authorization: Bearer $TUBE2VID_API_KEY"
```

For audio only, send `"mode":"audio","format":"mp3"`. The job's event log is at `GET /v1/jobs/{jobId}/events`.

## Account

```bash
curl -s https://tube2vid.com/v1/usage -H "authorization: Bearer $TUBE2VID_API_KEY"
# {"creditBalance":4200,"jobsTotal":12,"jobsCompleted":11,"jobsFailed":1}

# Needs the billing:write scope. Open checkoutUrl in a browser to pay.
curl -s https://tube2vid.com/v1/billing/checkout \
  -H "authorization: Bearer $TUBE2VID_API_KEY" \
  -H "content-type: application/json" \
  -d '{"credits":5000}'
```

## Public endpoints (no key)

```bash
curl -s https://tube2vid.com/v1/platforms             # first-class platforms and their providers
curl -s https://tube2vid.com/v1/platforms/health      # live status per platform
curl -s "https://tube2vid.com/v1/status/history?limit=20&platform=youtube"
curl -s https://tube2vid.com/v1/ops/metrics
curl -s https://tube2vid.com/healthz
```

`POST /v1/public/jobs` is the anonymous flow the website uses. It needs a solved work challenge from `GET /v1/challenges/pow` and a Cloudflare Turnstile token, so scripts should use `POST /v1/jobs` with a key.

## Errors

Errors are JSON with an `error` message and sometimes a `code`:

| Status | Meaning |
| --- | --- |
| 400 | Invalid body (`error` lists the field errors) or `unsupported_platform` |
| 401 | Missing or invalid bearer token |
| 402 | Not enough credits; `creditBalance` and `requiredCredits` are included |
| 403 | The key lacks the scope, or the job is not yours |
| 404 | Unknown job |
| 409 | The job has not completed yet |
| 410 | The file has expired |
| 502 | The job could not be queued; retry |

A job that fails reports `status: "failed"` with an `errorCode` and `errorMessage`, and its reserved credits are refunded.

## SDKs and tools

Official clients that wrap this API:

| Language | Repository | Install |
| --- | --- | --- |
| TypeScript / JavaScript | [Tube2Vid/sdk](https://github.com/Tube2Vid/sdk) | `npm install @tube2vid/sdk` |
| Python | [Tube2Vid/sdk-python](https://github.com/Tube2Vid/sdk-python) | `pip install tube2vid` |
| Go | [Tube2Vid/sdk-go](https://github.com/Tube2Vid/sdk-go) | `go get github.com/Tube2Vid/sdk-go` |
| Java | [Tube2Vid/sdk-java](https://github.com/Tube2Vid/sdk-java) | `com.tube2vid:tube2vid-sdk` |
| C# / .NET | [Tube2Vid/sdk-dotnet](https://github.com/Tube2Vid/sdk-dotnet) | `dotnet add package Tube2Vid` |
| PHP | [Tube2Vid/sdk-php](https://github.com/Tube2Vid/sdk-php) | `composer require tube2vid/sdk` |
| Ruby | [Tube2Vid/sdk-ruby](https://github.com/Tube2Vid/sdk-ruby) | `gem install tube2vid` |
| Rust | [Tube2Vid/sdk-rust](https://github.com/Tube2Vid/sdk-rust) | `cargo add tube2vid` |
| Swift | [Tube2Vid/sdk-swift](https://github.com/Tube2Vid/sdk-swift) | Swift Package Manager, package `Tube2Vid` |
| Command line | [Tube2Vid/cli](https://github.com/Tube2Vid/cli) | `npm install -g @tube2vid/cli` |
| AI tools (MCP) | [Tube2Vid/mcp](https://github.com/Tube2Vid/mcp) | `npx -y @tube2vid/mcp` |

## Links

- [Tube2Vid](https://tube2vid.com): the online video downloader this API powers
- [API documentation](https://tube2vid.com/docs)
- [Pricing and API keys](https://tube2vid.com/pricing)
- [Live platform status](https://tube2vid.com/status)

## Contributing

The spec mirrors the production API, so changes start there; issues about wrong or missing schemas are welcome. Lint before opening a pull request:

```bash
npx @redocly/cli lint openapi.yaml
```

## License

[MIT](LICENSE), Copyright (c) 2026 Tube2Vid.
