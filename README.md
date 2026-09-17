# My Privacy Agent plugin for Grok

Get your home address, phone number, and email off people-search and data-broker sites, with Grok doing the research and [My Privacy Agent](https://myprivacyagent.com) doing the follow-through.

This plugin connects Grok Build (and Grok Bot) to My Privacy Agent's hosted, read-only [MCP server](https://myprivacyagent.com/api/mcp). The tools return verified opt-out steps for 44 sites, the documented removal route for 950+ sites, and search across the privacy library. There is nothing to sign up for and no key to paste: the tools expose only what is already published on myprivacyagent.com, and they never look a person up.

Page for this plugin: **https://myprivacyagent.com/for/grok**

## Install

### Grok Build (terminal)

1. Install Grok Build if you have not (see the [Grok Build docs](https://docs.x.ai/build/overview)):

   ```bash
   curl -fsSL https://x.ai/cli/install.sh | bash
   ```

2. Start it with `grok` in any directory, then open the marketplace:

   ```text
   /marketplace
   ```

3. Find **my-privacy-agent** and press `i` to install. Until the marketplace listing is approved, add the server directly instead:

   ```bash
   grok mcp add --transport http my-privacy-agent https://myprivacyagent.com/api/mcp
   ```

4. Ask Grok something like *"How do I get my address off Spokeo?"* The `privacy-removal` skill guides the answer and the tools supply the facts. No sign-in step: the server is public.

### Grok Bot (app)

1. Open **Settings → Plugins**, search for **My Privacy Agent**, and click **Add**. Until the listing is approved, tell the bot in chat: *"Add a custom MCP server called My Privacy Agent at https://myprivacyagent.com/api/mcp"* and confirm the card it posts.
2. Mention `@My Privacy Agent` in a task, or just ask how to remove yourself from a site.

## What it does

| Tool | What it returns |
|---|---|
| `get_opt_out_instructions` | Step-by-step opt-out guide for one site: steps, requirements (CAPTCHA, email confirmation, ID), turnaround, gotchas, what the site exposes, and what My Privacy Agent does for that site after authorization. Accepts a slug, domain, alias, or URL. |
| `get_removal_route` | Documented removal method for any of 950+ sites, including the parent company that processes it, sibling domains, and whether the site is defunct. |
| `list_opt_out_guides` | Every published guide with method, difficulty, and turnaround; filter by category. |
| `search_privacy_library` | Ranked hits across articles, guides, help answers, and plan facts. |
| `get_service_overview` | Scope, handoff URLs, plan facts, and the rules an assistant should follow. |

Every response carries `schema_version`, `as_of`, `last_verified_at`, `source_urls`, and a `status` of `complete`, `partial`, or `not_found`.

| Skill | What it does |
|---|---|
| `privacy-removal` | Turns "my address is on X" into the official do-it-yourself route for X, with requirements, turnaround, and gotchas; states exactly what My Privacy Agent does for that site; and hands the person to the private check when they want their own exposure reviewed. |

## What it does not do

- It cannot search for a person. There is no lookup tool, and the skill tells Grok not to use web search to find anyone's listing.
- It does not read your report or case status. The private check runs on the website at https://myprivacyagent.com/check (free, no card); only the person who authorizes a check can see it. Account-scoped tools behind OAuth are planned and will ask for each permission separately.
- It does not file requests, create accounts, or start checkout on your behalf. A request is not a removal; nothing is described as removed until a recheck shows it.

## Network and data

- The plugin declares one network endpoint: `https://myprivacyagent.com/api/mcp` (MCP over Streamable HTTP; the same data is also available as HTTP + OpenAPI at `https://myprivacyagent.com/api/public/v1`).
- No credentials, environment variables, or local files are read. No hooks, no commands, no shell.
- Requests carry a static `x-mpa-client: grok-plugin` header so usage of the plugin can be counted. No identifiers are sent; tool arguments are site names, domains, or search terms that you provide.
- Read-only. The server holds no personal data and the tools return only published guidance.

The same server is listed in the official MCP registry as `com.myprivacyagent/public`.

## Rules the skill follows

Published for every assistant at https://myprivacyagent.com/agents.md: cite the guide URL when using its steps; never put a person's name, address, phone, or email into a URL; never say a listing is removed until a recheck shows it; do not create accounts, pay, or file requests autonomously.

## License

MIT. See [LICENSE](LICENSE).

Grok is a product of xAI. My Privacy Agent is not affiliated with or endorsed by xAI.
