---
name: setup
description: Set up Interfere in the current project. Creates the app, installs the SDK, connects GitHub and the deployment platform, and sends existing OpenTelemetry data to Interfere. Use when the user asks to set up, install, or add Interfere to a project.
---

Use the connected `interfere` MCP server. If authentication is required, direct the user to the client's connection flow. Do not ask for passwords or tokens in chat.

The product and the docs call it an app. The MCP calls the same thing a surface: operations such as `createSurface`, `surfaces_list` and `linkCliDestination` take a `surfaceSlug`. Say "app" to the user.

The server exposes `search` and `execute`. Use `search` to read each operation's description and schema before you call it with `execute`. Check the returned HTTP `status` before you use `body`.

## 1. Choose the workspace and the app

- Call `workspaceGate` to list the user's workspaces. Ask which one to use only if more than one is possible.
- Call `surfaces_list`. If an app already matches this project, use it. Otherwise create one with `createSurface`, with the name and framework of this project.
- Read the project to find the framework, the package manager and where the app deploys.

## 2. Install the SDK

- Follow the setup guide for the framework at https://interfere.com/docs. For example, a Vite and React app uses https://interfere.com/docs/sdk/vite.
- Before you reuse an Interfere key found in the environment, `.env` files or CI secrets, check it with `validateKey`. Reuse it only if the outcome is `valid`. Otherwise create a key for the app.
- Store keys in the place the project already uses for secrets. Never commit a secret key. Give only the public key a browser prefix such as `VITE_`.

## 3. Connect GitHub and the deployment platform

- Call `detectIntegrations` to see which integrations the project uses and which are connected.
- For GitHub, and for Vercel or Cloudflare if the app deploys there, get the install URL from `createIntegrationInstallUrl` and open it for the user. The user chooses the repositories and approves access in the browser. Wait for the user, then link the repository with `integrations_linkSurfaceToRepository` and the deployment project with `integrations_linkSurfaceToDestination`.
- If the app deploys anywhere else, such as GitHub Pages, Netlify or a server, link the app with `linkCliDestination`. The build plugin then reports each release.

## 4. Send existing OpenTelemetry data

Skip this step if the project does not export OpenTelemetry.

- Follow https://interfere.com/docs/sdk/otel-ingestion#export-directly-from-a-service, or the Collector section of the same page if the project exports through a Collector.
- Reuse a workspace key only if `validateKey` accepts it. Otherwise create a workspace key with `keys_private_create`, without a `surfaceSlug` and with the `org:telemetry:write` scope. Store it as `INTERFERE_API_KEY` in the server environment. Never commit it.
- Add Interfere as one more OTLP exporter. Keep the existing exporters. Do not add an exporter to a service that the Interfere SDK already covers, because each span then arrives twice.
- Once `serviceActivity` shows spans from a service, link the service to its repository with `setServiceSource`.

## 5. Finish

- Run the project's build or tests if they are quick, so the SDK change compiles.
- Tell the user what you changed, which steps still need them, and how to send a first test event.
- Do not change unrelated code, commit, or deploy unless the user asks.
