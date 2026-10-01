# Interfere plugin

Investigate production errors, sessions, and releases from Claude Code, Cursor, or Codex. The plugin connects to `https://mcp.interfere.com/mcp` and includes an investigation skill.

An Interfere account with access to the relevant workspace is required. Sign in through your client's connection flow and choose your workspace. Access follows your existing permissions. The connection can read data and perform changes you request; it is not read-only. No API key belongs in these files.

## Install

The installable plugin is in `plugins/interfere`. Using the plugin requires no dependency installation or build step.

### Claude Code

Add the marketplace, then install the plugin:

```text
/plugin marketplace add interfere-inc/plugins
/plugin install interfere@interfere-plugin-marketplace
```

Open `/mcp` to connect your Interfere account.

### Cursor

Clone the repository and copy the plugin into your local plugins directory:

```sh
git clone https://github.com/interfere-inc/plugins.git interfere-plugins
mkdir -p ~/.cursor/plugins/local
cp -R interfere-plugins/plugins/interfere ~/.cursor/plugins/local/interfere
```

Reload Cursor and open Customize to connect the Interfere MCP server. Managed installations may require your administrator to allow local plugin imports.

### Codex

Register the marketplace and install its plugin:

```sh
codex plugin marketplace add https://github.com/interfere-inc/plugins.git
codex plugin add interfere@interfere-plugin-marketplace
```

Restart your session and complete the Interfere connection flow when prompted. The package also includes marketplace metadata for supported Codex desktop clients.

## Use Interfere

Try asking:

- "Investigate recent errors in my Interfere workspace."
- "Trace this problem to its release and supporting evidence."
- "Find the sessions affected by this problem."

The server exposes `search` to discover available operations and `execute` to call them. The bundled skill helps the agent select the workspace, inspect the current API contract, and distinguish evidence from hypotheses. It does not automatically resolve problems or change settings during an investigation.

If sign-in fails, reconnect through your client. A permission error requires the relevant workspace access; retrying with another endpoint will not grant it.

For help, contact [Interfere support](mailto:support@interfere.com). See our [privacy policy](https://interfere.com/legal/privacy-policy) and [terms of service](https://interfere.com/legal/terms-of-service).
