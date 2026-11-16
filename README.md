<p align="center">
  <img src="assets/duobox-icon.svg" alt="Duobox" width="72" height="72">
</p>

# Duobox MCP server

The official MCP server for [Duobox](https://duobox.io), business email on your own domain. Connect ChatGPT, Claude, Cursor or any MCP client to your company email: your agent reads mail, writes and sends it, and manages the team, with the permissions you choose.

```
https://mcp.duobox.io/v1/mcp
```

It is a hosted server. There is nothing to install and there is no API key: the agent signs in over OAuth with a Duobox mail account, the same address and password you open your inbox with.

Full page: [duobox.io/en/mcp](https://duobox.io/en/mcp)

## Connect it

When the client asks you to sign in, use your Duobox mail account.

**Claude.** Open *Customize*, then *Connectors*, and add a connector. Paste the address. Leave sign-in on so it asks for your account.

**ChatGPT.** Open *Plugins* and add an MCP server. Paste the address and choose OAuth.

**Claude Code.** Run this in the project folder. It lands in `.mcp.json`.

```sh
claude mcp add --transport http duobox https://mcp.duobox.io/v1/mcp
```

**Cursor.** Open *Settings*, then *MCP*, and add a server. Or paste this into `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "duobox": {
      "url": "https://mcp.duobox.io/v1/mcp"
    }
  }
}
```

**VS Code.** In the command palette, *MCP: Add Server*. Or paste this into `.vscode/mcp.json`:

```json
{
  "servers": {
    "duobox": {
      "type": "http",
      "url": "https://mcp.duobox.io/v1/mcp"
    }
  }
}
```

**Windsurf.** In Cascade, open *Manage MCPs*. Or paste this into `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "duobox": {
      "serverUrl": "https://mcp.duobox.io/v1/mcp"
    }
  }
}
```

**Gemini CLI.** It lands in `~/.gemini/settings.json`.

```sh
gemini mcp add --transport http duobox https://mcp.duobox.io/v1/mcp
```

**Codex.** It lands in `~/.codex/config.toml`.

```sh
codex mcp add duobox --url https://mcp.duobox.io/v1/mcp
```

**Other clients.** If it asks for an address, paste the server's. If it asks for a file, use the Cursor one. If it asks for the type, it's `http`.

The same steps are in Duobox mail, under *Settings → MCP*.

## Permissions

When you connect a client, Duobox shows what it asks for. Everything starts checked: take off what you don't want.

| Permission | Scope | What it allows |
| --- | --- | --- |
| Read your mail | `mail:read` | See the emails you receive. |
| Save drafts | `mail:draft` | Leave them ready, without sending. |
| Send mail | `mail:send` | Send from your addresses. |
| Manage the company | `company` | The team, the plan and the inboxes. |

- Sending mail includes saving drafts, and saving drafts includes reading.
- If a tool needs a permission you took off, the client is asked for it again.
- *Manage the company* doesn't replace your role: if you don't administer the company, those tools answer no.
- In Duobox mail, *Agents → Connections* lists every connected app, since when, and the last time it came in. *Cut off access* shuts it out right away.

## The agent proposes, you send

`send_message` doesn't send by default: it leaves the email as the agent's draft and returns the preview, exactly as it would go out. `confirm_message` sends it untouched.

- If you asked the agent not to check with you, `send_message` with `auto` sends it in the same step.
- Each inbox decides whether the agent can send on its own: *Send without your OK*, under *Settings → MCP*. On new inboxes it starts off, and the proposal waits for a person to send it.
- What the agent proposes sits in Drafts, marked *Waiting for your OK*. What it did is in *Agents → Activity*.

## Tools

35 tools, grouped by task. Each one carries the MCP annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`) so the client can decide when to ask you.

**Reading mail**

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_my_mailboxes` | `mail:read` | The inboxes your account can open. |
| `list_messages` | `mail:read` | Lists an inbox by folder, with search and filters by alias, date and thread owner. |
| `get_message` | `mail:read` | Reads a whole email, with its thread, its attachments and, in a shared inbox, who has it. |
| `get_folder_counts` | `mail:read` | How many emails are in Inbox, Drafts and Spam, and how many of the agent's proposals wait for your OK. |
| `download_attachment` | `mail:read` | Downloads an attachment of up to 25 MB, with a link that expires in 10 minutes. |

**Writing and sending**

| Tool | Scope | What it does |
| --- | --- | --- |
| `save_draft` | `mail:draft` | Saves a draft marked as the agent's, without sending it. |
| `send_message` | `mail:send` | Writes an email or a reply. Unless told otherwise, it returns a preview and doesn't send. |
| `confirm_message` | `mail:send` | Sends the preview exactly as it was shown, after you approved it. |
| `mark_message` | `mail:draft` | Marks it read or unread, for everyone who shares the inbox. |

**The shared inbox**

| Tool | Scope | What it does |
| --- | --- | --- |
| `assign_thread` | `mail:draft` | Gives a thread an owner: the agent, a person in the inbox, or nobody. |
| `add_note` | `mail:draft` | Leaves a note for the team on the thread. The customer doesn't see it. |

**Skills**

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_skills` | `mail:read` | Your skills: instructions that rewrite an email's body before it goes out. |
| `create_skill` | `mail:draft` | Creates a skill: the tone, the language, the length, what always has to be said. |
| `update_skill` | `mail:draft` | Edits a skill's instructions, its description, or whether it always applies. |
| `delete_skill` | `mail:draft` | Deletes a skill. Mail already sent with it doesn't change. |

**Aliases and forwarding**

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_aliases` | `mail:read` | The aliases that deliver into an inbox. |
| `create_alias` | `mail:draft` | Adds an alias to the inbox, on its same domain. Up to five. |
| `delete_alias` | `mail:draft` | Removes an alias. The inbox stays. |
| `get_forward` | `mail:read` | An inbox's forwarding rules, and whether they are active yet. |
| `set_forward` | `mail:send` | Forwards what arrives: everything, mail from one sender, or mail sent to one address. An outside address stays pending until it confirms. |
| `delete_forward` | `mail:send` | Removes a forwarding rule. |

**The company, its people and its inboxes**

| Tool | Scope | What it does |
| --- | --- | --- |
| `list_organizations` | `mail:read` | The companies on your account. |
| `list_organization_mailboxes` | `company` | Every inbox in the company and who opens it. |
| `create_mailbox` | `company` | Creates an inbox, personal or shared. |
| `retry_mailbox` | `company` | Tries again an inbox that couldn't be created. |
| `report_mailbox` | `company` | Tells support about an inbox that couldn't be created. |
| `delete_mailbox` | `company` | Deletes an inbox and its aliases. |
| `list_members` | `company` | The people in the company and their role. |
| `list_invitations` | `company` | The invitations not accepted yet. |
| `create_invitation` | `company` | Invites someone: they get an email with a link to pick their password. |
| `resend_invitation` | `company` | Sends an invitation again. The previous link stops working. |
| `delete_invitation` | `company` | Cancels an invitation not accepted yet. |

**Your account and the plan**

| Tool | Scope | What it does |
| --- | --- | --- |
| `get_me` | `mail:read` | Your account: name, email and language. |
| `set_locale` | `mail:read` | The language of the emails we send you. |
| `get_entitlements` | `company` | How many people, domains and inboxes the plan allows, and how many are used. |

The server also publishes four prompts with the step by step of each task (`mail`, `addresses`, `team` and `account`) and ready examples, like summarizing what's unread or replying to an email.

## For developers

| | |
| --- | --- |
| Transport | Streamable HTTP, `POST /v1/mcp` |
| Authentication | OAuth with PKCE. Dynamic client registration or Client ID Metadata Documents. |
| Metadata | `https://mcp.duobox.io/.well-known/oauth-protected-resource` |
| Protocol | The 2026 version, and `2025-11-25` too. |
| Registry | [`io.duobox/mail`](https://registry.modelcontextprotocol.io/v0/servers?search=io.duobox/mail) in the official MCP registry |

This repository holds the server's public listing (`server.json`, `.mcp.json`), the ChatGPT plugin package (`chatgpt/`: `plugin.json`, `mcp.json` and the icons, zipped as-is for the OpenAI plugin dashboard) and its docs. The server itself runs on Duobox's infrastructure.

## You need a Duobox account

The agent signs in with a mail account. The account is born at [duobox.io](https://duobox.io): you create the company, turn on mail for your domain and create your address.

## Help

Something not working? Write to [hola@duobox.io](mailto:hola@duobox.io). Security reports: see [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
