# How to Post Product Updates to Discord and Telegram with OpenClaw

OpenClaw can turn a verified release record into separate Discord and Telegram product updates, then send the approved messages through Groniz. Groniz handles OAuth, platform formatting, scheduling, and delivery across 32+ networks, including Discord and Telegram.

Use one source packet, then write a separate draft for each community. Each draft needs human review and final publication approval. Before publishing or scheduling anything, inspect the live settings for both integrations because permissions, target identifiers, formatting, media support, and other capabilities may differ. The examples below have not been tested end to end against live Discord and Telegram connections. Treat each live schema as authoritative.

## The dual-channel workflow

```text
Release source
  -> extract user-visible change and evidence
  -> draft a Discord update and a Telegram update
  -> review community context and permissions
  -> discover each Groniz schema separately
  -> approve and deliver each message
  -> verify in both native destinations
```

This workflow is for community updates. It does not turn an OpenClaw conversation channel into a social publishing connection. Groniz remains the external delivery layer.

## Prerequisites

Before drafting, gather:

- An OpenClaw workspace with the Groniz Skill, MCP server, or CLI path.
- A Groniz account plus connected Discord and Telegram integrations.
- Permission to post in the intended Discord channel and Telegram chat or channel.
- A verified changelog, release note, or incident-resolution record.
- An approved link and any approved media.
- A reviewer who knows both communities.

Install the Skill with:

```bash
npx skills add groniz/groniz-cli
```

Or add the MCP server using the OpenClaw-specific key-in-URL setup:

```bash
openclaw mcp add groniz \
  --url https://mcp.groniz.com/mcp/YOUR_API_KEY \
  --transport streamable-http
```

OpenClaw's official documentation explains [workspace, project, and personal skills](https://docs.openclaw.ai/skills) and [managed outbound MCP definitions](https://docs.openclaw.ai/cli/mcp). The MCP URL contains the API key, so keep it secret and out of repository files.

The native CLI is a third path:

```bash
curl -fsSL https://groniz.com/install.sh | sh
groniz auth:login
```

The login command uses a browser device flow and does not require an API key. Whichever path you choose, confirm the connection with a read-only identity or integration-list command before preparing a write.

## Why Discord and Telegram need different drafts

Discord places messages inside channels, and permission to create a message depends on access to that channel. The [official Discord message resource](https://docs.discord.com/developers/resources/message) is the authoritative model. In practice, that means reviewing the exact destination channel, not merely the server name.

Telegram's Bot API sends a message to a target chat or channel and returns a Message on success. Its [official `sendMessage` documentation](https://core.telegram.org/bots/api#sendmessage) points to two checks: resolve the intended target and verify the native result.

The writing context is different too. A Discord release channel may sit beside support and discussion, so the update can direct readers to a thread or feedback channel when the source names one. A Telegram channel is often read as a linear broadcast. Its message should make sense on its own and lead with the user-visible change. Do not assume that Groniz exposes the same fields or formatting support for both destinations.

For the connection model, read [how to connect OpenClaw to social media](https://groniz.com/blog/connect-openclaw-to-social-media). [Introducing Groniz Connectors](https://groniz.com/blog/introducing-groniz-connectors) explains the publishing layer. The [Codex social publishing guide](https://groniz.com/blog/automate-social-media-posting-with-codex) applies the same review boundary to another agent, and [the GitHub release-to-X guide](https://groniz.com/blog/github-release-to-x-thread-with-codex) uses a similar repo-first evidence pattern.

## Prepare a bounded source packet

Give OpenClaw a compact, public-safe release file instead of asking it to infer an announcement from raw commit history:

```markdown
# Product update: Export progress events

Status: shipped
Available from: 2026-07-20 16:00 UTC
Audience: API customers using export jobs

What changed:
- Export jobs now emit progress events.

Why it matters:
- Clients can show progress without polling job state as often.

Evidence:
- docs/export-progress.md
- CHANGELOG.md#240

Link:
- https://example.com/changelog/export-progress

Exclude:
- internal queue names
- unmeasured performance claims
```

OpenClaw should verify every sentence against the evidence and flag any discrepancy. If availability is staged or conditional, say so in both public drafts.

Approve the packet for drafting only after checking the release state, availability, audience, link, and exclusions. This approval covers drafting, not publication. If the packet conflicts with the evidence, stop and correct the source. The agent should never choose whichever claim is easier to announce.

## Create channel-native drafts

Prompt OpenClaw with distinct requirements:

```text
Read updates/export-progress.md and its evidence. Produce two drafts.

Discord: write for an existing product-update channel. Lead with what shipped,
then who it helps, the one next action, and the approved link. Suggest a clear
place for follow-up only if the source names one.

Telegram: write a self-contained broadcast. Lead with the user-visible change,
keep the context understandable without earlier messages, and use the same
approved link.

Return a claim-to-source table and separate review checklists. Do not publish.
```

Cut generic excitement, invented outcomes, and piles of reactions or hashtags. Keep the facts aligned between drafts, but let the opening, length, call to action, mentions, and line breaks fit the destination.

## Reusable asset: dual-channel update card

Copy this for each release:

```markdown
# Dual-channel update card

Source release/version:
Availability and timezone:
Audience:
Verified user-visible change:
Approved benefit statement:
Known limitation:
Approved link:
Approved media:
Media rights and destination fit checked by:

## Discord
Server/community:
Target integration ID:
Intended channel: resolve from live settings/tools
Draft approved by:
Publication time:

## Telegram
Target integration ID:
Intended chat/channel: resolve from live settings/tools
Draft approved by:
Publication time:

Publication approval for Discord:
Publication approval for Telegram:
```

Use the card to catch a simple but consequential mistake: sending the approved copy to the wrong community target.

## Human review checkpoint

Review the source first. Check the release state, version, availability, link, limitation, and benefit. Remove internal identifiers, private rollout notes, and support instructions meant for staff. If the update has an image, confirm that it is approved for public use and fits the media treatment supported by the destination.

Next, review the communities separately. For Discord, confirm the connected server, intended channel, posting permissions, and expected use of mentions. Decide whether the message belongs in announcements, releases, or another named channel. For Telegram, confirm the exact chat or channel, make sure the bot can post there, and read the message without any surrounding conversation.

Be cautious with mentions. Broad notifications can disrupt a community, so include them only when someone has deliberately approved them and the live integration supports them. Approval for one destination does not carry over to the other.

## Discover live settings and deliver

Use OpenClaw through the Skill or CLI to run:

```bash
groniz whoami
groniz integrations:list
groniz integrations:settings <discord-integration-id>
groniz integrations:settings <telegram-integration-id>
```

Treat the settings responses as separate contracts. Supply every required value, follow the current limits, and call only the dynamic integration tools listed in that destination's response. If a tool can resolve a channel, chat, or other provider-owned identifier, use its live result instead of copying an ID from an earlier run.

If the update includes media, upload it first:

```bash
groniz upload ./assets/export-progress.png
```

Use the returned `.path` in the relevant request. Before either write, have OpenClaw render both complete requests, including the target and time. Content approval does not authorize delivery, so record publication approval for each destination.

Keep the two write commands separate:

```bash
groniz posts:create \
  -c "APPROVED_DISCORD_UPDATE" \
  -m "<returned-groniz-media-.path>" \
  -s "2026-08-21T15:00:00Z" \
  -t schedule \
  -i "DISCORD_INTEGRATION_ID" \
  --settings '<discord-required-settings-json>'

groniz posts:create \
  -c "APPROVED_TELEGRAM_UPDATE" \
  -m "<returned-groniz-media-.path>" \
  -s "2026-08-21T15:15:00Z" \
  -t schedule \
  -i "TELEGRAM_INTEGRATION_ID" \
  --settings '<telegram-required-settings-json>'
```

Both media arguments must use the `.path` returned by `groniz upload`. Omit `-m` if the approved post for that destination has no media. Build each settings JSON from its own live response, show both completed commands, and obtain separate publication approval before running either one. Approval of the source packet or message content does not authorize these commands.

## Verify delivery and handle partial failure

For each destination, record the Groniz post ID, status, requested time, and platform URL if Groniz returns one. A post ID proves that Groniz accepted the request. It does not prove that Discord or Telegram published the message. Open the native Discord channel and Telegram chat or channel, then confirm the account identity, target, text, link, media, formatting, and timestamp.

If one destination succeeds and the other fails, leave the successful post alone. Refresh the failed integration's settings and inspect its authorization, target permissions, required fields, current content limit, post type, and media path. Before retrying, check whether a timeout or delayed response created the message after all. Keep the approved copy unchanged when the fault is operational. If the correction changes the content, account, target, media, settings, or time, get fresh publication approval.

Use a failure record like this:

```markdown
Destination and integration ID:
Resolved native target:
Requested time/timezone:
Groniz post ID/status:
Native message found:
Error or rendering problem:
Correction:
Retry approved:
```

## Measure the next update natively

Use only the signals available inside each platform and account. On Discord, review meaningful replies, reactions, support questions, and whether readers found the right follow-up route. On Telegram, use the native metrics available to the account, plus replies or reactions when the channel configuration allows them. Exposure is only part of the picture. Repeated questions may mean the message left out a prerequisite or limitation.

Store these observations with the update card, then change one editorial choice for the next release. Discord and Telegram metrics are not equivalent, so do not fold them into one score. This workflow does not provide guaranteed community growth, automatic A/B testing, best-time optimization, or cross-platform audience-insight analytics.

When both destination accounts and permissions are ready, [connect them in Groniz Connectors](https://groniz.com/console/connectors).
