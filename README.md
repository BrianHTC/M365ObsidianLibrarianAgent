# Let Microsoft 365 Copilot Chat Manage an Obsidian Vault

> **Scope:** This guide connects a local Obsidian vault to a Microsoft 365 Copilot agent through Obsidian Local REST API, Omnisearch, the Obsidian Omnisearch LocalREST Integrator V1.1, and a Cloudflare Tunnel.
>
> **Security warning:** This design exposes a local API through a public hostname. Keep Local REST API authentication enabled, never publish the API key, restrict who can use the agent, and treat write/delete tools as privileged operations. Back up the vault before allowing an agent to modify files.

## Architecture

```text
Microsoft 365 Copilot Chat
        |
        v
Copilot Studio agent
        |
        | MCP over HTTPS + API-key authentication
        v
https://obsidian-mcp.example.me/mcp
        |
        v
Cloudflare Tunnel
        |
        v
Obsidian Local REST API MCP endpoint
        |
        +--> Vault file tools
        |
        +--> Omnisearch LocalREST Integrator V1.1
                  |
                  v
             Omnisearch index
```

## Important compatibility note

Microsoft's current documentation says Copilot Studio supports **Streamable HTTP** MCP transport and no longer supports legacy SSE transport after August 2025. Confirm that the installed V1.1 extension and Local REST API expose a Copilot Studio-compatible Streamable HTTP endpoint, normally ending in `/mcp`. If initialization fails despite correct authentication, transport compatibility is the first item to check.

---

## 1. Apply for the GitHub Student Developer Pack

1. Sign in to GitHub.
2. Open [GitHub Education Student Developer Pack](https://education.github.com/pack/).
3. Select **Sign up for Student Developer Pack**.
4. Complete the student-status application. Use your school-issued email where available and upload a current proof of enrollment if requested.
5. Wait until the Education benefit page shows that the application is approved.

The offers can change. Check the live Student Developer Pack page before relying on a specific benefit.

---

## 2. Claim the free Namecheap domain

At the time this guide was prepared, Namecheap advertised a free `.me` domain and one SSL certificate for one year to eligible GitHub students. Renewal after the free period is normally charged at the then-current rate, so review renewal and automatic-renewal settings.

1. While signed in to the approved GitHub account, open the [Namecheap GitHub Student offer](https://nc.me/landing/github).
2. Follow the unique redemption flow from GitHub Education.
3. Search for an available `.me` name.
4. Complete the Namecheap account and domain-registration process.
5. Record the domain, for example:

```text
example.me
```

6. In Namecheap, review **Auto-Renew** and the domain-expiration date.

> You do not need to manually install the Namecheap SSL certificate for the Cloudflare public hostname. Cloudflare handles HTTPS at its edge. The Local REST API can remain an HTTPS local origin behind the tunnel.

---

## 3. Move DNS management to Cloudflare

1. Create or sign in to a [Cloudflare](https://dash.cloudflare.com/) account.
2. Select **Add a domain/site** and enter the Namecheap domain.
3. Choose the Cloudflare plan appropriate for your use, including the free plan if sufficient.
4. Cloudflare displays two authoritative nameservers.
5. In Namecheap, open:

```text
Domain List > Manage > Nameservers > Custom DNS
```

6. Replace the Namecheap nameservers with the two Cloudflare nameservers.
7. Save the change.
8. Return to Cloudflare and wait until the zone status becomes **Active**.

Do not create a public `A` record pointing to your home IP. The Cloudflare Tunnel will create or manage the required hostname route without exposing the origin IP.

Suggested hostname:

```text
obsidian-mcp.example.me
```

---

## 4. Install the required Obsidian plugins

### 4.1 Install Local REST API

1. Open Obsidian.
2. Go to:

```text
Settings > Community plugins
```

3. Turn off Restricted Mode if necessary.
4. Select **Browse**, search for **Local REST API**, install it, and enable it.
5. Open its settings.
6. Keep authentication enabled.
7. Copy the API key into a password manager. Do not put it in the vault, GitHub repository, `prompt.txt`, screenshots, or this guide.
8. Note the HTTPS port shown by the plugin. A commonly used default is:

```text
27124
```

9. Confirm the MCP feature/endpoint is enabled in the Local REST API settings if the installed version provides a separate toggle.

### 4.1.1 Optional: use the same custom API key on multiple computers

This optional setup lets the existing Copilot Studio MCP connection continue using the same `Authorization` value when you move the same agent and public hostname between computers. It avoids replacing the API key in the Copilot Studio connection each time.

> **Important:** the API key alone does not move the Cloudflare route. Each computer must also run the same Cloudflare Tunnel configuration for the same public hostname. For a simple switching workflow, keep only one computer's Obsidian origin and tunnel connector active at a time. If multiple computers connect to the same tunnel simultaneously, requests can reach different origins. Only do that when every computer has an intentionally synchronized vault and identical plugin configuration.

1. Generate a strong random secret in a password manager. Use at least 32 random characters. For example:

```text
REPLACE-WITH-A-UNIQUE-RANDOM-SECRET-OF-32-OR-MORE-CHARACTERS
```

2. On the first computer, open:

```text
Obsidian > Settings > Local REST API
```

3. Locate the API-key field or the advanced setting that allows a custom API key. Replace the generated key with your chosen secret, then save or restart the plugin if requested. The exact label can vary by plugin version.
4. Repeat the same setting on every computer that will host the vault endpoint.
5. Install the same named Cloudflare Tunnel connector on each computer and use the same hostname and local service settings:

```text
Public MCP URL: https://obsidian-mcp.example.me/mcp
Local origin:    https://localhost:27124
API key:         the same custom secret on every computer
```

6. In Copilot Studio, configure the MCP connection once with:

```text
Header name:  Authorization
Header value: Bearer YOUR_SHARED_CUSTOM_API_KEY
```

7. When changing computers:

```text
1. Close Obsidian or stop cloudflared on the old computer.
2. Start Obsidian on the new computer.
3. Confirm Local REST API, Omnisearch, and the V1.1 integrator are enabled.
4. Start or verify cloudflared on the new computer.
5. Test the public /mcp endpoint.
6. Ask the existing agent to check the MCP connection.
```

Security requirements:

- Store the shared key in a password manager, not in the vault or a synced configuration note.
- Do not commit the key, tunnel token, plugin data file, or Copilot connection export to GitHub.
- Use a unique key that is not reused for any other service.
- Rotate the key on every computer and in Copilot Studio if any one computer is lost or compromised.
- A shared key increases the impact of one device compromise. Separate keys and connections are safer when seamless computer switching is not required.

The Local REST API MCP endpoint requires a bearer token in the `Authorization` header. See the project's current MCP client documentation for the authentication format.

### 4.2 Install Omnisearch

1. In **Community plugins**, search for **Omnisearch**.
2. Install and enable it.
3. Let the initial index finish.
4. Run a search inside Obsidian and confirm that expected notes appear.

If you need PDF or Office-document contents indexed, install and configure the text-extraction dependency recommended by Omnisearch, then rebuild or refresh the index.

### 4.3 Install Obsidian Omnisearch LocalREST Integrator V1.1

Download V1.1 from:

- [Obsidian Omnisearch LocalREST Integrator V1.1 release](https://github.com/BrianHTC/Obsidian-Omnisearch-Localrest-Integrator/releases/tag/V1.1)
- [Project repository](https://github.com/BrianHTC/Obsidian-Omnisearch-Localrest-Integrator)

Create this folder inside the target vault:

```text
YOUR-VAULT\.obsidian\plugins\omnisearch-mcp\
```

Place the V1.1 release files directly in that folder. The expected structure is:

```text
YOUR-VAULT/
└── .obsidian/
    └── plugins/
        └── omnisearch-mcp/
            ├── main.js
            ├── manifest.json
            └── styles.css
```

Avoid an extra nested directory such as:

```text
YOUR-VAULT\.obsidian\plugins\omnisearch-mcp\omnisearch-mcp\main.js
```

Reload Obsidian:

```text
Ctrl+R
```

Then enable the extension under:

```text
Settings > Community plugins > Omnisearch MCP
```

Confirm that these plugins are enabled:

```text
Local REST API
Omnisearch
Omnisearch MCP / Obsidian Omnisearch LocalREST Integrator V1.1
```

Keep Obsidian running during all connection tests. If Obsidian is closed, the local API origin is unavailable and Cloudflare commonly returns `502 Bad Gateway`.

---

## 5. Test the Local REST API MCP endpoint locally

Replace the placeholders before running this PowerShell test. Do not paste the real key into screenshots or shared logs.

```powershell
$ApiKey = Read-Host "Enter the Obsidian Local REST API key"
$Uri = "https://127.0.0.1:27124/mcp"

$Headers = @{
    Authorization = "Bearer $ApiKey"
    Accept        = "application/json, text/event-stream"
    "Content-Type" = "application/json"
}

$Body = @{
    jsonrpc = "2.0"
    id      = 1
    method  = "initialize"
    params  = @{
        protocolVersion = "2025-03-26"
        capabilities    = @{}
        clientInfo      = @{
            name    = "local-mcp-test"
            version = "1.0.0"
        }
    }
} | ConvertTo-Json -Depth 10

Invoke-WebRequest `
    -Uri $Uri `
    -Method Post `
    -Headers $Headers `
    -Body $Body `
    -SkipCertificateCheck
```

Expected result:

```text
HTTP 200
A JSON-RPC initialize response
An MCP protocolVersion and server capabilities
```

Interpret common results:

```text
200 = MCP endpoint responded
401 = API key or Authorization header is missing/incorrect
404 = wrong path or MCP endpoint is not enabled
502 through Cloudflare = Obsidian is closed, cloudflared is offline, or the local origin is wrong
```

> `-SkipCertificateCheck` is supported in PowerShell 7. If Windows PowerShell 5.1 is being used, test from a browser/curl environment that can explicitly trust or bypass the local self-signed certificate.

---

## 6. Install and configure Cloudflare Tunnel

Cloudflare's supported dashboard flow is **Networking > Tunnels > Create Tunnel**, followed by installing `cloudflared` and mapping a public hostname to a local service.

### 6.1 Create the tunnel

1. In Cloudflare, open:

```text
Networking > Tunnels
```

2. Select **Create Tunnel**.
3. Choose **Cloudflared**.
4. Name it, for example:

```text
obsidian-localrest
```

5. Select Windows and the correct CPU architecture.
6. Copy the installation command shown by Cloudflare.
7. Open **Command Prompt as Administrator** and run that command.

A typical command has this form. Use the token generated by your own Cloudflare dashboard:

```cmd
cloudflared.exe service install YOUR_ONE_TIME_TUNNEL_TOKEN
```

Never commit or share the tunnel token.

### 6.2 Add the public hostname route

In the tunnel configuration:

1. Select **Add route** or **Add a public hostname / Published application**.
2. Configure:

```text
Subdomain: obsidian-mcp
Domain:    example.me
Path:      leave blank
Type:      HTTPS
URL:       localhost:27124
```

Equivalent service URL:

```text
https://localhost:27124
```

3. Because Local REST API commonly uses a local self-signed certificate, open the route's origin/TLS settings and enable the setting equivalent to:

```text
No TLS Verify / Disable TLS certificate verification
```

Use this only for the tunnel-to-localhost hop. Public client traffic remains protected by Cloudflare HTTPS.

4. Save the route.
5. Confirm that the tunnel status is **Healthy**.

Your public MCP URL should now be:

```text
https://obsidian-mcp.example.me/mcp
```

### 6.3 Test the public endpoint

```powershell
$ApiKey = Read-Host "Enter the Obsidian Local REST API key"
$Uri = "https://obsidian-mcp.example.me/mcp"

$Headers = @{
    Authorization = "Bearer $ApiKey"
    Accept        = "application/json, text/event-stream"
    "Content-Type" = "application/json"
}

$Body = @{
    jsonrpc = "2.0"
    id      = 1
    method  = "initialize"
    params  = @{
        protocolVersion = "2025-03-26"
        capabilities    = @{}
        clientInfo      = @{
            name    = "public-mcp-test"
            version = "1.0.0"
        }
    }
} | ConvertTo-Json -Depth 10

Invoke-WebRequest `
    -Uri $Uri `
    -Method Post `
    -Headers $Headers `
    -Body $Body
```

Do not proceed until the public test returns a valid MCP initialization response.

---

## 7. Create the agent from Microsoft 365 Copilot Chat and open it in Copilot Studio

The labels can vary slightly by tenant and rollout.

1. Open Microsoft 365 Copilot Chat in a desktop browser.
2. Open **Agents**.
3. Select **Create agent**.
4. Select **New agent**.
5. If an initial guided setup appears, select **Skip**.
6. Select the **...** menu in the upper-right corner.
7. Select **Copy to Copilot Studio**.
8. Wait for the copied agent to open in Copilot Studio.

### Recommended agent model selection

On the agent's **Overview** page, open the **Model** selector and choose the primary model before final testing.

#### Recommended model: GPT 6 Sol

For this Obsidian MCP agent, select **GPT 6 Sol** when it is available in your Copilot Studio environment. The agent's main workload combines fast vault retrieval with dependable append and update operations. Sol provides the best balance for that workload: fast search, excellent append reliability, and strong overall tool-use suitability.

```text
Primary recommendation:       GPT 6 Sol
Complex research and editing: GPT 6 Astra
Strong general alternative:   Claude Sonnet 5.5
Substantial synthesis:        Claude Opus 5.5
Search-first configuration:   GPT 6 Luna
Legacy/general fallback:      GPT 5.5 Chat
```

#### Model comparison for this Obsidian agent

| Model | Search speed | Append reliability | Overall suitability |
|---|---|---|---|
| **GPT 6 Sol** | Fast | **Excellent** | **Best overall** |
| **GPT 6 Astra** | Slower | Excellent | Best for complicated research and editing |
| **Claude Sonnet 5.5** | Fast | Very good | Strong alternative |
| **Claude Opus 5.5** | Slow | Excellent | Good for substantial synthesis before writing |
| **GPT 6 Luna** | **Fastest** | Good for tightly constrained writes | Better as search-only |
| **GPT 5.5 Chat** | Fast | Good | Usable, but less attractive than Sol |

#### Why GPT 6 Sol is the default recommendation

Choose **GPT 6 Sol** for the standard agent because the same agent must do more than retrieve results. It must reliably select the correct MCP tool, preserve the target file path, append content without overwriting unrelated material, and verify the result afterward. Sol's excellent append reliability makes it preferable to Luna even though Luna is faster for search-only requests.

Typical Sol workloads include:

```text
Search the vault and return ranked results
Open the selected note
Append content under a specified heading
Create a new note at an exact vault-relative path
Update frontmatter or a constrained section
Read the changed note back for verification
```

#### When to select GPT 6 Astra

Use **GPT 6 Astra** when the agent must conduct complicated research before editing, such as synthesizing many notes, reconciling conflicting sources, planning a substantial restructuring, or carrying out a long sequence of dependent tool calls. Astra has excellent append reliability, but its slower search speed makes it less suitable as the everyday default.

#### When to select GPT 6 Luna

Use **GPT 6 Luna** for an agent dedicated primarily to search and retrieval. It is the fastest option in this comparison, but its writing behavior is better suited to tightly constrained changes. It should not be the default when users regularly ask the agent to append, revise, or reorganize notes.

For a Luna-based search agent, keep the instructions read-only where possible:

```text
Search, list, and read vault content. Do not create, modify, move, rename, or delete files unless the user explicitly switches to a write-enabled agent.
```

#### When to select a Claude model

Use **Claude Sonnet 5.5** as a strong alternative when you want fast search with very good append reliability. Use **Claude Opus 5.5** when substantial synthesis and careful planning matter more than response speed. Confirm that the tenant allows the selected model and that its data-processing requirements are acceptable before publication.

#### GPT 5.5 Chat as a fallback

**GPT 5.5 Chat** remains usable for fast, straightforward MCP operations, but it is less attractive than Sol for an agent that combines search with frequent file appends. Use it when the preferred models are not available in the target Copilot Studio environment.

#### Verify model availability in the target environment

Only select models that actually appear in the Copilot Studio agent's **Model** menu. Availability can differ by tenant, region, environment, licensing, rollout stage, and administrator policy. If **GPT 6 Sol** is unavailable, use the highest-ranked available production model from the comparison above.

#### Required model test

Run the same test at least five times with each available candidate. The test should include both retrieval and a controlled append because search-only testing does not measure the most important difference between Sol and Luna.

First run the read-only test:

```text
Check the MCP connection. Call the Obsidian MCP server and search the vault for "Directive". Return up to 10 results with vault-relative paths. Do not modify any file.
```

Then run a controlled append test against a disposable note:

```text
Open Copilot-Tests/Model-Append-Test.md. Append a new bullet under the heading "Test Results" containing the current date, model name, and the words "append test succeeded". Preserve all existing content and formatting. Read the changed section back to me after writing. Do not modify any other file.
```

Record these measurements:

```text
Time until the first MCP tool call
Successful MCP calls out of five attempts
Correct selection of the Omnisearch tool
Correct target path selection
Append success without overwriting existing content
Correct placement under the requested heading
Successful read-after-write verification
Number of unnecessary tool calls
Total response time
```

Keep **GPT 6 Sol** as the default unless testing in the actual tenant shows that another available model is both faster and at least as reliable for append operations. End-to-end testing is more relevant than a general model benchmark because performance also depends on Copilot Studio orchestration, the Cloudflare route, Local REST API, the V1.1 integration, vault size, and tool descriptions.

Prerequisites:

```text
A tenant that permits agent creation
Permission to access a Copilot Studio environment
Permission to create connections/connectors
Power Platform and Teams policies that allow the intended agent distribution
```

If **Copy to Copilot Studio** is missing, the tenant may not have the required environment, maker permission, license/capacity, or feature rollout. Contact the Microsoft 365/Power Platform administrator.

---

## 8. Add the MCP server and create its connection

Microsoft documents the current flow as:

```text
Agent > Tools > Add a tool > New tool > Model Context Protocol
```

Enter values similar to:

```text
Server name: Obsidian Vault MCP
Server description: Securely lists, reads, searches, creates, and updates files in the user's Obsidian vault. Use Omnisearch for ranked vault search and inspect results before opening or modifying files.
Server URL: https://obsidian-mcp.example.me/mcp
Authentication: API key
API key location/type: Header
Header name: Authorization
```

For the connection's API-key value, enter the complete header value expected by Local REST API:

```text
Bearer YOUR_OBSIDIAN_LOCAL_REST_API_KEY
```

Do not include that value in the agent instructions, description, conversation, exported documentation, or source control.

Then:

1. Select **Create**.
2. Select **Create a new connection**.
3. Supply the API-key value when prompted.
4. Select **Add to agent**.

### Use the V1.1 `prompt.txt` as the agent instructions

1. Download `prompt.txt` from the [V1.1 release](https://github.com/BrianHTC/Obsidian-Omnisearch-Localrest-Integrator/releases/tag/V1.1).
2. Open it locally.
3. Copy its complete contents.
4. In Copilot Studio, open the agent's **Instructions** section.
5. Paste the contents as the agent instructions.
6. Save the agent.

Do not add the API key to `prompt.txt`.

---

## 9. Inspect the Tools section

Open the agent's **Tools** page and select the Obsidian MCP server. Confirm that the expected tool list is visible and that tool names and descriptions are readable.

At minimum, verify both categories:

```text
Vault-management tools from Local REST API
Omnisearch tool supplied by the V1.1 integrator, such as omnisearch_search
```

The exact Local REST API tool names can vary by installed version. Confirm that the list includes the operations you intend to permit, for example:

```text
List vault files or folders
Read/open a file
Create or update a note
Search through Omnisearch
Open or navigate to a selected result, if provided by V1.1
```

If tools are missing:

1. Verify all three Obsidian plugins are enabled.
2. Restart or reload Obsidian.
3. Confirm the local `/mcp` initialize test succeeds.
4. Confirm the public `/mcp` initialize test succeeds.
5. Reopen or refresh the MCP connection in Copilot Studio.
6. If needed, remove and re-add the MCP tool so Copilot Studio retrieves the current tool manifest.

Do not publish until the tool list loads correctly.

---

## 10. Test the agent inside Copilot Studio

Start with a non-destructive request:

```text
Check the MCP connection. Report whether the Obsidian MCP server is available, list the tools you can access, and do not create, modify, move, or delete any file.
```

Then test search:

```text
Use the Obsidian search tool to search for "copyright". Show the first 10 individual occurrences and the first 10 matching files. Do not modify the vault.
```

Only after read/search tests pass should you test a write in a disposable folder:

```text
Create a test note at Copilot-Tests/MCP-Connection-Test.md containing only today's date and the sentence "MCP write test succeeded." Then read the note back to me. Do not modify any other file.
```

Delete the test note manually after verification if you do not want the agent to exercise delete operations.

---

## 11. Publish to Teams + Microsoft 365

Microsoft requires the agent to be published before users can interact with it through Teams and Microsoft 365.

1. Save all changes.
2. Select **Publish** and publish the latest content.
3. Open **Channels**.
4. Select **Teams + Microsoft 365** or the equivalently named **Teams and Microsoft Copilot** channel.
5. Ensure the option equivalent to **Make agent available in Microsoft Copilot** is enabled.
6. Add/configure the channel.
7. Publish again if Copilot Studio requests it.

If the agent is only for you or a small test group, use limited sharing. Organization-wide availability can require administrator approval.

---

## 12. Copy and open the share link first

1. In the Teams + Microsoft 365 channel or availability settings, select **Copy link**.
2. Open the copied link in a desktop browser while signed in to the correct Microsoft 365 tenant.
3. Add/install the agent if prompted.
4. Start a conversation and send:

```text
Check the MCP connection. Report whether the Obsidian MCP server is available, list the available tools, and do not modify the vault.
```

If a connection consent card appears, create/authorize the connection using the API key stored securely in your password manager.

A successful response should identify available vault and Omnisearch tools. A generic conversational answer without a real tool call is not sufficient evidence.

---

## 13. Open the agent in Microsoft 365 Copilot Chat

1. Return to Microsoft 365 Copilot Chat.
2. Refresh the page or sign out and back in if needed.
3. Open the **Agents** tab.
4. Select the newly installed/published agent.
5. Start a new conversation.
6. Send:

```text
Check the MCP connection. Call the MCP server, report whether it is connected, and list the available Obsidian tools. Do not create, modify, move, or delete any file.
```

Then run a read-only vault check:

```text
List the first 10 files in the root of my Obsidian vault. Do not modify anything.
```

Finally, test Omnisearch:

```text
Search my Obsidian vault for "Directive". Show up to the first 10 individual occurrences and the first 10 matching files. Include vault-relative paths and enough excerpt context to distinguish the results. Do not modify any file.
```

---

## Troubleshooting

### HTTP 401 Unauthorized

Likely causes:

```text
Missing Authorization header
Wrong API key
Header value lacks the "Bearer " prefix
A stale Copilot Studio connection still contains an old key
```

Expected configuration:

```text
Header name: Authorization
Header value: Bearer YOUR_API_KEY
```

Regenerate or rotate the key if it might have been exposed, then update the Copilot Studio connection.

### HTTP 502 Bad Gateway

Check in this order:

```text
1. Obsidian is running.
2. Local REST API is enabled.
3. https://127.0.0.1:27124/mcp responds locally.
4. The cloudflared Windows service is running.
5. The tunnel is Healthy in Cloudflare.
6. The route points to https://localhost:27124.
7. No TLS Verify is enabled for the self-signed local origin.
```

Windows service checks:

```powershell
Get-Service cloudflared
```

```powershell
Restart-Service cloudflared
```

If `Get-Service cloudflared` finds nothing, reinstall the service using the command generated by the Cloudflare dashboard.

### HTTP 404 Not Found

Confirm the URL includes the MCP path:

```text
https://obsidian-mcp.example.me/mcp
```

Also confirm MCP is enabled in Local REST API and that the installed version uses `/mcp`.

### Tool list is incomplete

```text
Confirm Local REST API is enabled
Confirm Omnisearch is enabled
Confirm Omnisearch MCP V1.1 is enabled
Reload Obsidian with Ctrl+R
Wait for Omnisearch indexing to finish
Retest the endpoint
Refresh or recreate the Copilot Studio MCP connection
```

### Agent does not appear in Microsoft 365 Copilot Chat

```text
Open the share/install link in a desktop browser first
Confirm the Teams + Microsoft 365 channel is enabled
Confirm "Make agent available in Microsoft Copilot" is enabled
Confirm the latest agent version is published
Refresh Copilot Chat or sign out and back in
Check tenant app policies and admin approval requirements
```

### Changes do not appear in an existing conversation

Start a new conversation or send:

```text
Start over
```

Published updates might not appear in a conversation that was already open.

---

## Security and operating checklist

- [ ] Back up the vault before enabling write operations.
- [ ] Keep the vault in version control or use another reliable snapshot system.
- [ ] Keep Local REST API authentication enabled.
- [ ] Store the API key only in a password manager and Copilot Studio connection secret storage.
- [ ] Never place the API key in `prompt.txt`, the vault, GitHub, screenshots, or chat messages.
- [ ] Limit the agent to your account or an explicit test group before broader publication.
- [ ] Review every write/delete tool before publishing.
- [ ] Prefer read-only prompts during connection testing.
- [ ] Keep Obsidian, Local REST API, Omnisearch, the V1.1 integrator, and cloudflared updated.
- [ ] Re-test tools after any plugin, tunnel, authentication, or agent update.
- [ ] Rotate the Local REST API key immediately if it is exposed.
- [ ] Stop or disable the tunnel when remote access is not needed.

---

## End-to-end completion checklist

- [ ] GitHub Student Developer Pack approved.
- [ ] Namecheap domain claimed and renewal settings reviewed.
- [ ] Domain nameservers changed to Cloudflare and zone is Active.
- [ ] Local REST API installed, enabled, and authenticated.
- [ ] Optional multi-computer setup uses the same custom API key and controlled tunnel handoff.
- [ ] Omnisearch installed, enabled, and indexed.
- [ ] Obsidian Omnisearch LocalREST Integrator V1.1 installed and enabled.
- [ ] Local MCP initialize test returns HTTP 200.
- [ ] Cloudflare Tunnel installed as a service and Healthy.
- [ ] Public hostname routes to the Local REST API HTTPS port.
- [ ] Public MCP initialize test returns HTTP 200.
- [ ] Agent created in Microsoft 365 Copilot Chat and copied to Copilot Studio.
- [ ] Primary agent model selected and tested for tool-call speed and reliability.
- [ ] MCP server added under Tools with an API-key connection.
- [ ] V1.1 `prompt.txt` copied into agent instructions.
- [ ] Expected vault and Omnisearch tools appear.
- [ ] Read-only Copilot Studio test succeeds.
- [ ] Teams + Microsoft 365 channel enabled.
- [ ] Agent published.
- [ ] Share link opened in a browser and MCP connection checked.
- [ ] Agent appears in Microsoft 365 Copilot Chat.
- [ ] MCP connection checked again from Microsoft 365 Copilot Chat.

---

## References

- [GitHub Student Developer Pack](https://education.github.com/pack/)
- [Namecheap for GitHub Students](https://nc.me/landing/github)
- [Cloudflare Tunnel: Get started](https://developers.cloudflare.com/tunnel/get-started/)
- [Obsidian Local REST API documentation](https://coddingtonbear.github.io/obsidian-local-rest-api/)
- [Omnisearch](https://github.com/scambier/obsidian-omnisearch)
- [Obsidian Omnisearch LocalREST Integrator](https://github.com/BrianHTC/Obsidian-Omnisearch-Localrest-Integrator)
- [V1.1 release and prompt.txt](https://github.com/BrianHTC/Obsidian-Omnisearch-Localrest-Integrator/releases/tag/V1.1)
- [Microsoft: Connect an agent to an existing MCP server](https://learn.microsoft.com/en-us/microsoft-copilot-studio/mcp-add-existing-server-to-agent)
- [Microsoft: Connect and configure an agent for Teams and Microsoft 365](https://learn.microsoft.com/en-us/microsoft-copilot-studio/publication-add-bot-to-microsoft-teams)

---

**Guide date:** 12 October 2026. Cloud services and UI labels change frequently. If a label differs, follow the equivalent current option and verify against the official references above.
