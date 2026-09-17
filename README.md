# My Privacy Agent plugin for Grok

Get your home address, phone number, and email off people-search and data-broker sites, with Grok doing the research and [My Privacy Agent](https://myprivacyagent.com) doing the follow-through.

This plugin connects Grok Build (and Grok Bot) to two hosted MCP servers from My Privacy Agent:

- **Public server** (`https://myprivacyagent.com/api/mcp`) — read-only, no account, no key. Verified opt-out steps for 44 sites, the documented removal route for 950+ sites, and search across the privacy library. It exposes only what is already published on myprivacyagent.com, and it never looks a person up.
- **Account server** (`https://myprivacyagent.com/api/mcp/me`) — sign-in required. After you sign in to My Privacy Agent and grant permissions one at a time, Grok can read *your own* report and case status, queue a Patrol re-check, and file the removal requests you already approved on the website. Nothing on your account is readable before you sign in.

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
3. Find **my-privacy-agent** and press `i` to install. Until the marketplace listing is approved, add the servers directly instead:
   ```bash
   grok mcp add --transport http my-privacy-agent https://myprivacyagent.com/api/mcp
   grok mcp add --transport http my-privacy-agent-account https://myprivacyagent.com/api/mcp/me
   ```
4. Ask Grok something like *"How do I get my address off Spokeo?"* The `privacy-removal` skill guides the answer and the public tools supply the facts. No sign-in is needed for that.
5. When you want Grok to see your own report: open `/mcps`, select **my-privacy-agent-account** and press `i`. A browser window shows My Privacy Agent's sign-in and consent screen. The first answer that needs a permission comes back with a link to `https://myprivacyagent.com/connect`, where you tick only what you want.

### Grok Bot (app)

1. Open **Settings → Plugins**, search for **My Privacy Agent**, and click **Add**. Until the listing is approved, tell the bot in chat: *"Add a custom MCP server called My Privacy Agent at https://myprivacyagent.com/api/mcp"* and confirm the card it posts.
2. Mention `@My Privacy Agent` in a task, or just ask how to remove yourself from a site.
3. The account server needs a sign-in step; Grok Bot's custom-server flow has not been verified with sign-in yet, so use Grok Build for account tools until it is.

## What it does

### Public tools (no sign-in)

| Tool | What it returns |
|---|---|
| `get_opt_out_instructions` | Step-by-step opt-out guide for one site: steps, requirements (CAPTCHA, email confirmation, ID), turnaround, gotchas, what the site exposes, and what My Privacy Agent does for that site after authorization. Accepts a slug, domain, alias, or URL. |
| `get_removal_route` | Documented removal method for any of 950+ sites, including the parent company that processes it, sibling domains, and whether the site is defunct. |
| `list_opt_out_guides` | Every published guide with method, difficulty, and turnaround; filter by category. |
| `search_privacy_library` | Ranked hits across articles, guides, help answers, and plan facts. |
| `get_service_overview` | Scope, handoff URLs, plan facts, and the rules an assistant should follow. |

Every response carries `schema_version`, `as_of`, `last_verified_at`, `source_urls`, and a `status` of `complete`, `partial`, or `not_found`.

### Account tools (sign in, then grant each permission separately)

| Tool | Needs | What it does |
|---|---|---|
| `list_my_subjects` | sign-in only | The subjects on your account and which permissions Grok holds for each. |
| `get_my_report` | Share results with this assistant | Your findings summary for one subject (site, status, counts, dates — never listing URLs, evidence, email, or phone). |
| `list_my_cases` | Share results with this assistant | Every removal case for a subject with its current status. |
| `get_case_status` | Share results with this assistant | One case in detail, including what you still have to do. |
| `request_recheck` | Run my check | Queues the same re-check the monthly Patrol schedule runs (Patrol plan, verified subject, once per 24 hours). |
| `submit_approved_plan` | Submit these requests | Previews, then with `confirm: true` files removal requests for listings you already approved on the website. A request is not a removal. |

Permissions are granted at `https://myprivacyagent.com/connect` and revoked at `https://myprivacyagent.com/connect/manage`. There is no spending permission: plans, upgrades, and payments happen on the website, by you.

| Skill | What it does |
|---|---|
| `privacy-removal` | Turns "my address is on X" into the official do-it-yourself route for X, with requirements, turnaround, and gotchas; states exactly what My Privacy Agent does for that site; hands the person to the private check when they want their own exposure reviewed; and, once signed in, reads their report and case status within the permissions they granted. |

## What it does not do

- It cannot search for a person. There is no lookup tool, and the skill tells Grok not to use web search to find anyone's listing.
- It cannot start your first check. The private check runs on the website at https://myprivacyagent.com/check (free, no card); only the person who authorizes a check can see it. The account tools re-check and file only for subjects that already exist on your account.
- It does not buy anything. There is no spending permission; the assistant cannot start checkout or change your plan.
- It does not file requests you have not approved on the website, and it never describes a request as a removal; nothing is marked removed until a recheck shows it.

## Network and data

- The plugin declares two network endpoints: `https://myprivacyagent.com/api/mcp` (public, MCP over Streamable HTTP; the same data is also available as HTTP + OpenAPI at `https://myprivacyagent.com/api/public/v1`) and `https://myprivacyagent.com/api/mcp/me` (account, MCP over Streamable HTTP, OAuth 2.1).
- Account sign-in uses My Privacy Agent's authorization server at `https://clerk.myprivacyagent.com` (`/oauth/authorize`, `/oauth/token`, `/oauth/register`; PKCE S256; dynamic client registration; a consent screen every time). Grok holds an access token for the signed-in person, never a password. Resource metadata: `https://myprivacyagent.com/.well-known/oauth-protected-resource/api/mcp/me`.
- No credentials, environment variables, or local files are read. No hooks, no commands, no shell.
- Requests carry a static `x-mpa-client: grok-plugin` header so usage of the plugin can be counted. Public tool arguments are site names, domains, or search terms that you provide; account tools take a `subject_id` from `list_my_subjects`, never a name, address, phone, or email.
- The public server holds no personal data. The account server returns your own data only after sign-in and only within the permissions you granted; each call is recorded on your account and can be reviewed at `https://myprivacyagent.com/connect/manage`.

The public server is listed in the official MCP registry as `com.myprivacyagent/public`.

## Rules the skill follows

Published for every assistant at https://myprivacyagent.com/agents.md: cite the guide URL when using its steps; never put a person's name, address, phone, or email into a URL; never say a listing is removed until a recheck shows it; do not create accounts, pay, or file requests autonomously; with the account tools, ask before `submit_approved_plan` runs with `confirm: true`.

## License

MIT. See [LICENSE](LICENSE).

Grok is a product of xAI. My Privacy Agent is not affiliated with or endorsed by xAI.
