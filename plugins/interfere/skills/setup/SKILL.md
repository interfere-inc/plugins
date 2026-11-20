---
name: setup
description: Set up Interfere in the current project. Creates the app, installs the SDK, connects GitHub and the deployment platform, and sends existing OpenTelemetry data to Interfere. Use when the user asks to set up, install, or add Interfere to a project.
---

Use the connected `interfere` MCP server. If authentication is required, direct the user to the client's connection flow. Do not ask for passwords or tokens in chat.

The product and the docs call it an app. The MCP calls the same thing a surface: operations such as `createSurface`, `surfaces_list` and `linkCliDestination` take a `surfaceSlug`. Say "app" to the user.

The server exposes `search` and `execute`. Use `search` to read each operation's description and schema before you call it with `execute`. Check the returned HTTP `status` before you use `body`.

Do not retry a write after a timeout or an unclear result. Read the current state first, for example with `surfaces_list`, and reuse the same `idempotencyKey` when the operation takes one, so that you do not create a second app or key. A 403 means that the user lacks a permission. Tell the user which step needs a workspace admin, and continue with the other steps.

## 1. Choose the workspace and the apps

- Call `workspaceGate` to list the user's workspaces. Ask which one to use only if more than one is possible.
- Read the project to find the framework, the package manager and where each part deploys.
- In a monorepo, ask which packages to set up. Create one app for each deployed frontend.
- A backend service without a browser SDK, or a service in another language, needs no app. Set it up through step 4 only.
- Call `surfaces_list`. If an app already matches a package, use it. Otherwise create one with `createSurface`, with the name and framework of that package.
- `createSurface` returns the public key of the app. Store it at once, as step 2 describes. The secret key comes from step 2, not from the app.

## 2. Install the SDK

- Follow the setup guide for the framework at https://interfere.com/docs. For example, a Vite and React app uses https://interfere.com/docs/sdk/vite.
- Before you reuse an Interfere key found in the environment, `.env` files or CI secrets, check it with `validateKey`. Reuse it only if the outcome is `valid`.
- The build needs a workspace secret key with the `release:write` scope, stored as `INTERFERE_API_KEY`. One key publishes releases for every app in the workspace. Do not create a key for an app.
- If the project has no valid secret key, call `keys_private_list` and keep the keys that have no `surfaceId`. Read the scopes of each key with `keys_private_get`. Show the user the name, the last 4 characters and the scopes of each key, then ask the user to choose:
  - **Use an existing key.** Interfere shows a secret only when the key is created, so the user adds it to `.env.local` or to the project's secret manager. Never ask the user to paste a secret into the chat. When the user confirms, check the key with `validateKey`.
  - **Create a key for this project.** Call `keys_private_create` without a `surfaceSlug`, with the `release:write` scope only. Name the key after the project and where it builds, for example `my-app (Vercel build)`. Write the returned secret straight into the project's secret store as `INTERFERE_API_KEY`, and do not show it.
- Recommend a new key when every listed key has more scopes than `release:write`. The build secret goes to CI and to the deployment platform, and a broader key there can change members, billing and other keys.
- Store keys in the place the project already uses for secrets. Never commit a secret key. Give only the public key a browser prefix such as `VITE_`.
- The build needs the same keys where it runs in CI and on the deployment platform, not only in local files. You cannot set those values. List each variable that the user must add there, with its name and where it goes, but not the secret value.

## 3. Connect GitHub and the deployment platform

- Call `detectIntegrations` to see which integrations the project uses and which are connected.
- For GitHub, and for Vercel or Cloudflare if the app deploys there, get the install URL from `createIntegrationInstallUrl` and open it for the user. The user chooses the repositories and approves access in the browser. Wait for the user, then link the repository with `integrations_linkSurfaceToRepository` and the deployment project with `integrations_linkSurfaceToDestination`.
- In a monorepo, set `workingDirectory` to the package folder of the app when you link the repository.
- If the app deploys anywhere else, such as GitHub Pages, Netlify or a server, link the app with `linkCliDestination`. The build plugin then reports each release.

## 4. Send existing OpenTelemetry data

Use this step for a backend service, an OpenTelemetry Collector, or a project that already exports OpenTelemetry. Skip it if the project exports no OpenTelemetry data.

- Follow https://interfere.com/docs/sdk/otel-ingestion#export-directly-from-a-service, or the Collector section of the same page if the project exports through a Collector.
- This step uses a workspace telemetry key, which is different from the secret key of an app. Name its variable `INTERFERE_TELEMETRY_KEY`, so that it does not replace the `INTERFERE_API_KEY` of an app. The guide uses `INTERFERE_API_KEY`, so use `INTERFERE_TELEMETRY_KEY` in its place in the exporter header, for example `Authorization=Bearer%20${INTERFERE_TELEMETRY_KEY}`, or `${env:INTERFERE_TELEMETRY_KEY}` in a Collector configuration.
- Reuse a workspace key only if `validateKey` accepts it. Otherwise create a workspace key with `keys_private_create`, without a `surfaceSlug` and with the `org:telemetry:write` scope. Store it as `INTERFERE_TELEMETRY_KEY` in the environment of the service or the Collector. Never commit it.
- Add Interfere as one more OTLP exporter. Keep the existing exporters. Do not add an exporter to a service that the Interfere SDK already covers, because each span then arrives twice.
- Once `serviceActivity` shows spans from a service, link the service to its repository with `setServiceSource`.

## 5. Finish

- Run the project's build or tests if they are quick, so the SDK change compiles.
- Tell the user what you changed, which steps still need them, and how to send a first test event.
- After the user sends a test event, check `surfaceActivity` and tell the user whether the event arrived.
- Point the user to the consent guide (https://interfere.com/docs/sdk/consent) and the replay masking guide (https://interfere.com/docs/sdk/replay-privacy). Do not change those settings yourself.
- Do not change unrelated code, commit, or deploy unless the user asks.
