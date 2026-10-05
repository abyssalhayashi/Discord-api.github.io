# [discord-api-reference.md](https://github.com/user-attachments/files/33072339/discord-api-reference.md)
Discord-api.github.io# Discord API v10 Quick Reference

Base URL: `https://discord.com/api/v10`

## Essentials

- **Auth:** Bot apps send `Authorization: Bot <token>`. User OAuth2 apps send `Bearer <access_token>`.

- **Format:** JSON bodies with `Content-Type: application/json`. IDs are snowflake strings. Send a `User-Agent` header.

- **Rate limits:** Global limit is 50 requests/second per app, plus per-route buckets. On `429`, wait `retry_after` seconds. Read `X-RateLimit-*` headers.

- **Errors:** Responses carry an HTTP status and JSON `{code, message, errors}`. `401` bad token, `403` missing permission, `404` unknown resource.

## Getting started checklist

1. Create an application in the [Discord Developer Portal](https://discord.com/developers/applications)
2. Open the Bot tab, copy the token (keep it secret, never commit it) and enable any privileged intents you need
3. Pick the permissions and scopes your bot needs
4. Invite the bot to a test server with an OAuth2 URL
5. Make your first call: GET /users/@me
6. Handle 429 responses using retry_after and the X-RateLimit headers

## Endpoints

### Users

#### `GET /users/@me`

Get the current user

```bash
curl -X GET "https://discord.com/api/v10/users/@me" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "id": "80351110224678912",
  "username": "nelly",
  "global_name": "Nelly",
  "bot": true
}
```

#### `GET /users/{user.id}`

Get a user by ID

| Name | Type | In | Notes |
|---|---|---|---|
| `user.id` | snowflake | path | User ID |

```bash
curl -X GET "https://discord.com/api/v10/users/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "id": "80351110224678912",
  "username": "nelly",
  "avatar": "8342729096ea3675442027381ff50dfe"
}
```

#### `POST /users/@me/channels`

Create a DM channel

| Name | Type | In | Notes |
|---|---|---|---|
| `recipient_id` | snowflake | body | User to DM |

```bash
curl -X POST "https://discord.com/api/v10/users/@me/channels" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"recipient_id":"80351110224678912"}'
```

Example response:

```json
{
  "id": "319674150115610528",
  "type": 1
}
```

#### `GET /users/@me/guilds`

List guilds the current user is in

| Name | Type | In | Notes |
|---|---|---|---|
| `limit` | int | query | 1-200 |
| `after` | snowflake | query | Pagination cursor |

```bash
curl -X GET "https://discord.com/api/v10/users/@me/guilds" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "80351110224678912", "name": "1337 Krew", "owner": true } ]
```

#### `DELETE /users/@me/guilds/{guild.id}`

Leave a guild

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |

```bash
curl -X DELETE "https://discord.com/api/v10/users/@me/guilds/GUILD_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Channels

#### `GET /channels/{channel.id}`

Get a channel

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |

```bash
curl -X GET "https://discord.com/api/v10/channels/CHANNEL_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "id": "41771983423143937",
  "type": 0,
  "name": "general",
  "guild_id": "41771983423143937"
}
```

#### `PATCH /channels/{channel.id}`

Modify a channel

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |
| `name` | string | body | 2-100 characters |
| `topic` | string | body | Up to 1024 characters |

```bash
curl -X PATCH "https://discord.com/api/v10/channels/CHANNEL_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"announcements","topic":"News only"}'
```

Example response:

```json
{ "id": "41771983423143937", "name": "announcements" }
```

#### `DELETE /channels/{channel.id}`

Delete or close a channel

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |

```bash
curl -X DELETE "https://discord.com/api/v10/channels/CHANNEL_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// Returns the deleted channel object
```

#### `POST /channels/{channel.id}/messages/bulk-delete`

Bulk delete 2-100 messages younger than 14 days

| Name | Type | In | Notes |
|---|---|---|---|
| `messages` | snowflake[] | body | Message IDs |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/messages/bulk-delete" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"messages":["334385199974967042","334385199974967043"]}'
```

Example response:

```json
// 204 No Content (needs MANAGE_MESSAGES)
```

#### `POST /channels/{channel.id}/invites`

Create an invite

| Name | Type | In | Notes |
|---|---|---|---|
| `max_age` | int | body | Seconds, 0 = never (default 86400) |
| `max_uses` | int | body | 0-100, 0 = unlimited |
| `unique` | bool | body | Always make a new code |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/invites" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"max_age":3600,"max_uses":5}'
```

Example response:

```json
{ "code": "0vCdhLbwjZZTWZLD", "channel": { "id": "41771983423143937" } }
```

#### `POST /channels/{channel.id}/messages/{message.id}/threads`

Start a thread from a message

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-100 characters |
| `auto_archive_duration` | int | body | 60, 1440, 4320 or 10080 minutes |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/messages/MESSAGE_ID/threads" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Discussion","auto_archive_duration":1440}'
```

Example response:

```json
{ "id": "6120934", "type": 11, "name": "Discussion" }
```

#### `POST /channels/{channel.id}/webhooks`

Create a webhook

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-80 characters |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/webhooks" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"CI Notifier"}'
```

Example response:

```json
{ "id": "223704706495545344", "token": "3d89bb7572e0fb30d8128367b3b1b44fecd1726de135cbe28a41f8b2f777c372ba2939e72279b94526ff5d1bd4358d65cf11" }
```

### Messages

#### `GET /channels/{channel.id}/messages`

List messages (newest first)

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |
| `limit` | int | query | 1-100, default 50 |
| `before` | snowflake | query | Messages before this ID |

```bash
curl -X GET "https://discord.com/api/v10/channels/CHANNEL_ID/messages" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[
  { "id": "334385199974967042", "content": "Supa Hot", "author": { "id": "53908232506183680" } }
]
```

#### `POST /channels/{channel.id}/messages`

Send a message

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |
| `content` | string | body | Up to 2000 characters |
| `embeds` | array | body | Up to 10 embed objects |
| `tts` | bool | body | Text-to-speech |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/messages" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Hello, world!"}'
```

Example response:

```json
{
  "id": "334385199974967042",
  "channel_id": "41771983423143937",
  "content": "Hello, world!"
}
```

#### `PATCH /channels/{channel.id}/messages/{message.id}`

Edit a message you sent

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |
| `message.id` | snowflake | path | Message ID |
| `content` | string | body | New content |

```bash
curl -X PATCH "https://discord.com/api/v10/channels/CHANNEL_ID/messages/MESSAGE_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Edited!"}'
```

Example response:

```json
{ "id": "334385199974967042", "content": "Edited!" }
```

#### `DELETE /channels/{channel.id}/messages/{message.id}`

Delete a message

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Channel ID |
| `message.id` | snowflake | path | Message ID |

```bash
curl -X DELETE "https://discord.com/api/v10/channels/CHANNEL_ID/messages/MESSAGE_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

#### `PUT /channels/{channel.id}/messages/{message.id}/reactions/{emoji}/@me`

Add a reaction

| Name | Type | In | Notes |
|---|---|---|---|
| `emoji` | string | path | URL-encoded emoji or name:id |

```bash
curl -X PUT "https://discord.com/api/v10/channels/CHANNEL_ID/messages/MESSAGE_ID/reactions/EMOJI/@me" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Guilds

#### `GET /guilds/{guild.id}`

Get a guild (server)

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |
| `with_counts` | bool | query | Include member counts |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "id": "197038439483310086",
  "name": "Discord Testers",
  "owner_id": "73193882359173120"
}
```

#### `GET /guilds/{guild.id}/channels`

List guild channels

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/channels" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "41771983423143937", "name": "general", "type": 0 } ]
```

#### `GET /guilds/{guild.id}/members/{user.id}`

Get a guild member

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |
| `user.id` | snowflake | path | User ID |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/members/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "user": { "id": "80351110224678912" },
  "nick": "NOT API SUPPORT",
  "roles": []
}
```

#### `PUT /guilds/{guild.id}/bans/{user.id}`

Ban a user

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |
| `user.id` | snowflake | path | User ID |
| `delete_message_seconds` | int | body | 0-604800 |

```bash
curl -X PUT "https://discord.com/api/v10/guilds/GUILD_ID/bans/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"delete_message_seconds":3600}'
```

Example response:

```json
// 204 No Content (requires BAN_MEMBERS)
```

#### `POST /guilds/{guild.id}/channels`

Create a channel

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-100 characters |
| `type` | int | body | 0 text, 2 voice, 4 category, 5 announcement, 13 stage, 15 forum |
| `parent_id` | snowflake | body | Category ID |

```bash
curl -X POST "https://discord.com/api/v10/guilds/GUILD_ID/channels" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"dev-chat","type":0}'
```

Example response:

```json
{ "id": "41771983423143999", "name": "dev-chat", "type": 0 }
```

#### `GET /guilds/{guild.id}/members`

List members (needs GUILD_MEMBERS intent)

| Name | Type | In | Notes |
|---|---|---|---|
| `limit` | int | query | 1-1000, default 1 |
| `after` | snowflake | query | Highest user ID of previous page |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/members" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "user": { "id": "80351110224678912" }, "roles": [], "joined_at": "2024-01-01T00:00:00Z" } ]
```

#### `PATCH /guilds/{guild.id}/members/{user.id}`

Modify a member (nick, roles, timeout)

| Name | Type | In | Notes |
|---|---|---|---|
| `nick` | string | body | Nickname |
| `roles` | snowflake[] | body | Replace roles |
| `communication_disabled_until` | ISO8601 | body | Timeout end (max 28 days); null clears |

```bash
curl -X PATCH "https://discord.com/api/v10/guilds/GUILD_ID/members/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"communication_disabled_until":"2026-10-06T12:00:00Z"}'
```

Example response:

```json
{ "user": { "id": "80351110224678912" }, "communication_disabled_until": "2026-10-06T12:00:00+00:00" }
```

#### `DELETE /guilds/{guild.id}/members/{user.id}`

Kick a member

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |
| `user.id` | snowflake | path | User ID |

```bash
curl -X DELETE "https://discord.com/api/v10/guilds/GUILD_ID/members/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content (needs KICK_MEMBERS)
```

#### `GET /guilds/{guild.id}/roles`

List roles

| Name | Type | In | Notes |
|---|---|---|---|
| `guild.id` | snowflake | path | Guild ID |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/roles" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "41771983423143936", "name": "@everyone", "permissions": "6546771529" } ]
```

#### `POST /guilds/{guild.id}/roles`

Create a role

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | Role name |
| `permissions` | string | body | Bitfield as string |
| `color` | int | body | RGB integer |
| `hoist` | bool | body | Show separately |

```bash
curl -X POST "https://discord.com/api/v10/guilds/GUILD_ID/roles" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Moderator","permissions":"1099511627776","color":5793266}'
```

Example response:

```json
{ "id": "41771983423143940", "name": "Moderator" }
```

#### `PUT /guilds/{guild.id}/members/{user.id}/roles/{role.id}`

Add a role to a member

| Name | Type | In | Notes |
|---|---|---|---|
| `role.id` | snowflake | path | Role ID |

```bash
curl -X PUT "https://discord.com/api/v10/guilds/GUILD_ID/members/USER_ID/roles/ROLE_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content (needs MANAGE_ROLES)
```

#### `DELETE /guilds/{guild.id}/bans/{user.id}`

Unban a user

| Name | Type | In | Notes |
|---|---|---|---|
| `user.id` | snowflake | path | User ID |

```bash
curl -X DELETE "https://discord.com/api/v10/guilds/GUILD_ID/bans/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Commands

#### `PUT /applications/{application.id}/commands`

Bulk overwrite global slash commands

| Name | Type | In | Notes |
|---|---|---|---|
| `application.id` | snowflake | path | Application ID |
| `[]` | array | body | Array of command objects |

```bash
curl -X PUT "https://discord.com/api/v10/applications/APPLICATION_ID/commands" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '[{"name":"ping","description":"Replies with pong","type":1}]'
```

Example response:

```json
[ { "id": "771091", "name": "ping", "type": 1 } ]
```

#### `POST /interactions/{interaction.id}/{interaction.token}/callback`

Respond to an interaction

| Name | Type | In | Notes |
|---|---|---|---|
| `type` | int | body | 4 = message, 5 = deferred |
| `data` | object | body | Message data |

```bash
curl -X POST "https://discord.com/api/v10/interactions/INTERACTION_ID/INTERACTION_TOKEN/callback" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":4,"data":{"content":"pong"}}'
```

Example response:

```json
// 204 No Content (respond within 3 seconds)
```

#### `GET /applications/{application.id}/commands`

List global commands

| Name | Type | In | Notes |
|---|---|---|---|
| `with_localizations` | bool | query | Include translations |

```bash
curl -X GET "https://discord.com/api/v10/applications/APPLICATION_ID/commands" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "771091", "name": "ping", "type": 1, "options": [] } ]
```

#### `POST /applications/{application.id}/guilds/{guild.id}/commands`

Create a guild command (updates instantly)

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-32 chars, lowercase |
| `description` | string | body | 1-100 chars |
| `options` | array | body | Up to 25 options |

```bash
curl -X POST "https://discord.com/api/v10/applications/APPLICATION_ID/guilds/GUILD_ID/commands" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"echo","description":"Repeat text","options":[{"type":3,"name":"text","description":"What to say","required":true}]}'
```

Example response:

```json
{ "id": "771092", "name": "echo", "guild_id": "197038439483310086" }
```

#### `POST /webhooks/{application.id}/{interaction.token}`

Send a follow-up message (token valid 15 min) (authenticated by the token in the URL)

| Name | Type | In | Notes |
|---|---|---|---|
| `content` | string | body | Message text |
| `flags` | int | body | 64 = ephemeral |

```bash
curl -X POST "https://discord.com/api/v10/webhooks/APPLICATION_ID/INTERACTION_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Done!","flags":64}'
```

Example response:

```json
{ "id": "334385199974967050", "content": "Done!" }
```

#### `PATCH /webhooks/{application.id}/{interaction.token}/messages/@original`

Edit the original response (after deferring) (authenticated by the token in the URL)

| Name | Type | In | Notes |
|---|---|---|---|
| `content` | string | body | New content |

```bash
curl -X PATCH "https://discord.com/api/v10/webhooks/APPLICATION_ID/INTERACTION_TOKEN/messages/@original" \
  -H "Content-Type: application/json" \
  -d '{"content":"Finished processing."}'
```

Example response:

```json
{ "id": "334385199974967042", "content": "Finished processing." }
```

### Webhooks

#### `POST /webhooks/{webhook.id}/{webhook.token}`

Execute a webhook (no bot token needed) (authenticated by the token in the URL)

| Name | Type | In | Notes |
|---|---|---|---|
| `content` | string | body | Message text |
| `username` | string | body | Override display name |
| `wait` | bool | query | Return the created message |

```bash
curl -X POST "https://discord.com/api/v10/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"Deploy finished","username":"CI"}'
```

Example response:

```json
// 204 No Content (or message object if wait=true)
```

#### `PATCH /webhooks/{webhook.id}/{webhook.token}/messages/{message.id}`

Edit a webhook message (authenticated by the token in the URL)

| Name | Type | In | Notes |
|---|---|---|---|
| `content` | string | body | New content |

```bash
curl -X PATCH "https://discord.com/api/v10/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN/messages/MESSAGE_ID" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated"}'
```

Example response:

```json
{ "id": "334385199974967042", "content": "Updated" }
```

#### `DELETE /webhooks/{webhook.id}/{webhook.token}`

Delete a webhook (authenticated by the token in the URL)

```bash
curl -X DELETE "https://discord.com/api/v10/webhooks/WEBHOOK_ID/WEBHOOK_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Invites

#### `GET /invites/{code}`

Get an invite

| Name | Type | In | Notes |
|---|---|---|---|
| `code` | string | path | Invite code |
| `with_counts` | bool | query | Approximate member counts |

```bash
curl -X GET "https://discord.com/api/v10/invites/CODE" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{ "code": "0vCdhLbwjZZTWZLD", "guild": { "name": "Discord Testers" } }
```

### Gateway

#### `GET /gateway/bot`

Get the WebSocket URL, shard count and session limits

```bash
curl -X GET "https://discord.com/api/v10/gateway/bot" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "url": "wss://gateway.discord.gg",
  "shards": 1,
  "session_start_limit": { "total": 1000, "remaining": 999, "reset_after": 14400000, "max_concurrency": 1 }
}
```

### Forums & Threads

#### `POST /channels/{channel.id}/threads`

Start a thread; in a forum channel this creates a post

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-100 chars |
| `message` | object | body | Required in forums: first post (content, embeds...) |
| `applied_tags` | snowflake[] | body | Forum tag IDs |
| `type` | int | body | 11 public, 12 private (non-forum) |

```bash
curl -X POST "https://discord.com/api/v10/channels/CHANNEL_ID/threads" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Bug: login fails","message":{"content":"Steps to reproduce..."},"applied_tags":["1234567890"]}'
```

Example response:

```json
{ "id": "6120934", "type": 11, "parent_id": "41771983423143937", "applied_tags": ["1234567890"] }
```

#### `PUT /channels/{channel.id}/thread-members/@me`

Join a thread

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Thread ID |

```bash
curl -X PUT "https://discord.com/api/v10/channels/CHANNEL_ID/thread-members/@me" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Stage

#### `POST /stage-instances`

Start a Stage (needs MANAGE_CHANNELS, MUTE_MEMBERS, MOVE_MEMBERS)

| Name | Type | In | Notes |
|---|---|---|---|
| `channel_id` | snowflake | body | Stage channel ID |
| `topic` | string | body | 1-120 chars |
| `privacy_level` | int | body | 2 = guild only |

```bash
curl -X POST "https://discord.com/api/v10/stage-instances" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel_id":"41771983423143937","topic":"Community AMA","privacy_level":2}'
```

Example response:

```json
{ "id": "840647391636226060", "channel_id": "41771983423143937", "topic": "Community AMA" }
```

#### `DELETE /stage-instances/{channel.id}`

End a Stage

| Name | Type | In | Notes |
|---|---|---|---|
| `channel.id` | snowflake | path | Stage channel ID |

```bash
curl -X DELETE "https://discord.com/api/v10/stage-instances/CHANNEL_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Voice

#### `GET /voice/regions`

List voice regions

```bash
curl -X GET "https://discord.com/api/v10/voice/regions" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "europe", "name": "Europe", "optimal": true, "deprecated": false } ]
```

#### `PATCH /guilds/{guild.id}/members/{user.id}`

Move, mute or deafen a member in voice

| Name | Type | In | Notes |
|---|---|---|---|
| `channel_id` | snowflake | body | Target voice channel; null disconnects |
| `mute` | bool | body | Server mute |
| `deaf` | bool | body | Server deafen |

```bash
curl -X PATCH "https://discord.com/api/v10/guilds/GUILD_ID/members/USER_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel_id":"41771983423143999"}'
```

Example response:

```json
{ "user": { "id": "80351110224678912" }, "mute": false }
```

### Scheduled Events

#### `GET /guilds/{guild.id}/scheduled-events`

List scheduled events

| Name | Type | In | Notes |
|---|---|---|---|
| `with_user_count` | bool | query | Include interested counts |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/scheduled-events" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "9001", "name": "Game night", "status": 1, "entity_type": 2 } ]
```

#### `POST /guilds/{guild.id}/scheduled-events`

Create an event

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 1-100 chars |
| `scheduled_start_time` | ISO8601 | body | Start time |
| `scheduled_end_time` | ISO8601 | body | Required for external events |
| `entity_type` | int | body | 1 stage, 2 voice, 3 external |
| `channel_id` | snowflake | body | For stage/voice events |
| `entity_metadata` | object | body | {location} for external events |
| `privacy_level` | int | body | 2 = guild only |

```bash
curl -X POST "https://discord.com/api/v10/guilds/GUILD_ID/scheduled-events" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Game night","scheduled_start_time":"2026-10-10T19:00:00Z","entity_type":2,"channel_id":"41771983423143999","privacy_level":2}'
```

Example response:

```json
{ "id": "9001", "name": "Game night", "status": 1 }
```

#### `PATCH /guilds/{guild.id}/scheduled-events/{event.id}`

Edit an event or change its status

| Name | Type | In | Notes |
|---|---|---|---|
| `status` | int | body | 2 active, 3 completed, 4 canceled (scheduled -> active -> completed) |

```bash
curl -X PATCH "https://discord.com/api/v10/guilds/GUILD_ID/scheduled-events/EVENT_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"status":2}'
```

Example response:

```json
{ "id": "9001", "status": 2 }
```

#### `GET /guilds/{guild.id}/scheduled-events/{event.id}/users`

List users interested in an event

| Name | Type | In | Notes |
|---|---|---|---|
| `limit` | int | query | Up to 100 |
| `with_member` | bool | query | Include member objects |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/scheduled-events/EVENT_ID/users" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "guild_scheduled_event_id": "9001", "user": { "id": "80351110224678912" } } ]
```

#### `DELETE /guilds/{guild.id}/scheduled-events/{event.id}`

Delete an event

```bash
curl -X DELETE "https://discord.com/api/v10/guilds/GUILD_ID/scheduled-events/EVENT_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### AutoMod

#### `GET /guilds/{guild.id}/auto-moderation/rules`

List AutoMod rules (needs MANAGE_GUILD)

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/auto-moderation/rules" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "969707018069872670", "name": "Block spam links", "trigger_type": 1, "enabled": true } ]
```

#### `POST /guilds/{guild.id}/auto-moderation/rules`

Create an AutoMod rule

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | Rule name |
| `event_type` | int | body | 1 = message send |
| `trigger_type` | int | body | 1 keyword, 3 spam, 4 preset, 5 mention spam, 6 member profile |
| `trigger_metadata` | object | body | keyword_filter, regex_patterns, allow_list, mention_total_limit... |
| `actions` | array | body | Block, alert or timeout |
| `exempt_roles` | snowflake[] | body | Up to 20 |

```bash
curl -X POST "https://discord.com/api/v10/guilds/GUILD_ID/auto-moderation/rules" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"No bad words","event_type":1,"trigger_type":1,"trigger_metadata":{"keyword_filter":["*badword*"]},"actions":[{"type":1},{"type":2,"metadata":{"channel_id":"41771983423143937"}}],"enabled":true}'
```

Example response:

```json
{ "id": "969707018069872670", "name": "No bad words", "enabled": true }
```

#### `PATCH /guilds/{guild.id}/auto-moderation/rules/{rule.id}`

Edit a rule

| Name | Type | In | Notes |
|---|---|---|---|
| `enabled` | bool | body | Toggle the rule |

```bash
curl -X PATCH "https://discord.com/api/v10/guilds/GUILD_ID/auto-moderation/rules/RULE_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"enabled":false}'
```

Example response:

```json
{ "id": "969707018069872670", "enabled": false }
```

#### `DELETE /guilds/{guild.id}/auto-moderation/rules/{rule.id}`

Delete a rule

```bash
curl -X DELETE "https://discord.com/api/v10/guilds/GUILD_ID/auto-moderation/rules/RULE_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

### Audit Log

#### `GET /guilds/{guild.id}/audit-logs`

Fetch audit log entries (needs VIEW_AUDIT_LOG)

| Name | Type | In | Notes |
|---|---|---|---|
| `user_id` | snowflake | query | Who did it |
| `action_type` | int | query | Filter by action |
| `before` | snowflake | query | Entries before this ID |
| `limit` | int | query | 1-100, default 50 |

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/audit-logs" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
{
  "audit_log_entries": [ { "id": "1", "action_type": 22, "user_id": "73193882359173120", "target_id": "80351110224678912", "reason": "spam" } ],
  "users": []
}
```

### Emoji & Stickers

#### `GET /guilds/{guild.id}/emojis`

List custom emoji

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/emojis" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "41771983429993937", "name": "blob", "animated": false } ]
```

#### `POST /guilds/{guild.id}/emojis`

Create an emoji (image max 256 KiB)

| Name | Type | In | Notes |
|---|---|---|---|
| `name` | string | body | 2-32 chars |
| `image` | data URI | body | data:image/png;base64,... |
| `roles` | snowflake[] | body | Roles allowed to use it |

```bash
curl -X POST "https://discord.com/api/v10/guilds/GUILD_ID/emojis" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"blob","image":"data:image/png;base64,iVBORw0KGgo..."}'
```

Example response:

```json
{ "id": "41771983429993937", "name": "blob" }
```

#### `DELETE /guilds/{guild.id}/emojis/{emoji.id}`

Delete an emoji

```bash
curl -X DELETE "https://discord.com/api/v10/guilds/GUILD_ID/emojis/EMOJI_ID" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
// 204 No Content
```

#### `GET /guilds/{guild.id}/stickers`

List guild stickers (creating one uses multipart/form-data)

```bash
curl -X GET "https://discord.com/api/v10/guilds/GUILD_ID/stickers" \
  -H "Authorization: Bot $BOT_TOKEN" \
  -H "Accept: application/json"
```

Example response:

```json
[ { "id": "749054660769218631", "name": "wave", "format_type": 1, "tags": "hello" } ]
```

## Building a bot

[**Open the Developer Portal -&gt;**](https://discord.com/developers/applications)  \|  [Full docs](https://discord.com/developers/docs)

**1. Create the app.** In the portal choose *New Application*. Note the **Application ID** (General Information) and, for HTTP interactions, the **Public Key**.

**2. Get the token.** *Bot* tab -\> *Reset Token* -\> copy it once. Store it in an environment variable. If it leaks, reset it immediately; old tokens stop working at once.

**3. Choose intents.** On the same tab, toggle *Presence*, *Server Members* and *Message Content* only if you need them. Slash-command-only bots need none of them.

**4. Install the bot.** *Installation* or *OAuth2 -\> URL Generator*: pick scopes `bot` and `applications.commands` and the permissions you need. The URL looks like `https://discord.com/oauth2/authorize?client_id=APP_ID&scope=bot%20applications.commands&permissions=2048`. Request the minimum permissions.

**5. Run it.** Either keep a Gateway connection open (a library handles this) or use an **Interactions Endpoint URL** (HTTP webhooks, no persistent connection, slash commands and components only).

**6. Grow.** Bots in roughly 75-100 servers need Discord verification, and privileged intents require approval at that point. Shard when the gateway tells you to (`/gateway/bot`).

**Minimal bot with a slash command**

```js
// npm install discord.js
const { Client, Events, GatewayIntentBits } = require("discord.js");
const client = new Client({ intents: [GatewayIntentBits.Guilds] });

client.once(Events.ClientReady, c => console.log("Logged in as " + c.user.tag));

client.on(Events.InteractionCreate, async i => {
  if (!i.isChatInputCommand() || i.commandName !== "ping") return;
  await i.reply({ content: "pong", flags: 64 }); // 64 = ephemeral
});

client.login(process.env.BOT_TOKEN);
// Register /ping first: see PUT .../commands in Endpoints
```

```python
# pip install discord.py
import os, discord
from discord import app_commands

client = discord.Client(intents=discord.Intents.default())
tree = app_commands.CommandTree(client)

@tree.command(name="ping", description="Replies with pong")
async def ping(interaction: discord.Interaction):
    await interaction.response.send_message("pong", ephemeral=True)

@client.event
async def on_ready():
    await tree.sync()  # registers commands
    print("Logged in as", client.user)

client.run(os.environ["BOT_TOKEN"])
```

### Slash commands and interactions

| Option type         | Value | Option type                                         | Value |
|---------------------|-------|-----------------------------------------------------|-------|
| SUB_COMMAND / GROUP | 1 / 2 | CHANNEL                                             | 7     |
| STRING              | 3     | ROLE / MENTIONABLE                                  | 8 / 9 |
| INTEGER             | 4     | NUMBER (float)                                      | 10    |
| BOOLEAN             | 5     | ATTACHMENT                                          | 11    |
| USER                | 6     | Command types: 1 slash, 2 user menu, 3 message menu |       |

An incoming interaction has `type`: 1 PING, 2 command, 3 component, 4 autocomplete, 5 modal submit. You must answer within **3 seconds**, then you have **15 minutes** of follow-ups using the interaction token.

| Callback type | Use                                                |
|---------------|----------------------------------------------------|
| 4             | Reply with a message                               |
| 5             | Defer ("thinking..."), then edit `@original` later |
| 6 / 7         | Defer / update the message a component was on      |
| 8             | Autocomplete choices (max 25)                      |
| 9             | Open a modal                                       |

Set `flags: 64` for an ephemeral reply only the invoker sees. Guild commands update instantly; global commands can take time to propagate, so use a test guild while developing.

### Embeds and components

```
{ "content": "Pick one",
  "embeds": [{ "title": "Status", "description": "All systems normal", "color": 5793266,
    "fields": [{ "name": "Uptime", "value": "99.9%", "inline": true }],
    "footer": { "text": "Updated" }, "timestamp": "2026-10-05T12:00:00Z" }],
  "components": [{ "type": 1, "components": [
    { "type": 2, "style": 1, "label": "Confirm", "custom_id": "confirm" },
    { "type": 2, "style": 5, "label": "Docs", "url": "https://discord.com/developers/docs" } ] }] }
```

**Limits:** content 2000 chars; up to 10 embeds per message and 6000 combined embed characters; title 256, description 4096, 25 fields, field name 256, field value 1024, footer 2048. An action row (type 1) holds up to 5 buttons or one select menu, and a message holds up to 5 rows.

**Component types:** 2 button, 3 string select, 4 text input (modals), 5 user select, 6 role select, 7 mentionable select, 8 channel select. **Button styles:** 1 primary, 2 secondary, 3 success, 4 danger, 5 link (needs `url`, sends no interaction). `custom_id` can be up to 100 characters; use it to route the interaction back to your code.

### HTTP interactions (no gateway)

Set the Interactions Endpoint URL in the portal. Discord sends a signed POST for every interaction. You **must** verify the Ed25519 signature using the app's public key over `timestamp + rawBody` (headers `X-Signature-Ed25519` and `X-Signature-Timestamp`) and return `401` if it fails; Discord tests this. Reply to `{"type":1}` pings with `{"type":1}`, and to commands with a callback such as `{"type":4,"data":{"content":"pong"}}`.

### Best practices and gotchas

- Never put the token in client-side code, logs, screenshots or git. Use a secrets manager.
- Prefer slash commands over reading message content; they need no privileged intent.
- Cache with the gateway instead of polling REST; respect rate limit headers and queue requests per bucket.
- Defer long tasks (type 5) so you don't miss the 3-second window.
- Check `permissions` before acting, and handle `50013` gracefully. A bot's role must sit above any role it manages.
- Mentions: set `allowed_mentions` to stop user input from pinging `@everyone`.
- Snowflakes are 64-bit: always treat IDs as strings in JSON and JavaScript. Created time = `(id >> 22) + 1420070400000` ms.
- Back off and retry on `5xx`; don't retry `4xx` except `429`.

## Advanced features

Endpoints for these live in the list above (filter by Forums, Stage, Voice, Scheduled Events, AutoMod, Audit Log, Emoji).

### Channel types

| Type | Name         | Type    | Name                    |
|------|--------------|---------|-------------------------|
| 0    | Text         | 11 / 12 | Public / private thread |
| 1    | DM           | 10      | Announcement thread     |
| 2    | Voice        | 13      | Stage                   |
| 4    | Category     | 15      | Forum                   |
| 5    | Announcement | 16      | Media                   |

### Forum channels

A forum channel (type 15) holds posts, and each post is a thread. Create a post with `POST /channels/{forum.id}/threads` and include a `message`. Configure the forum with `available_tags` (up to 20, each with `name` and optional emoji), `default_sort_order` (0 latest activity, 1 creation date), `default_forum_layout` (1 list, 2 gallery) and `default_reaction_emoji`. Flag `1<<4` (REQUIRE_TAG) forces posts to carry a tag. Threads auto-archive after 60, 1440, 4320 or 10080 minutes of inactivity; archived threads reopen when someone posts. Bots need `CREATE_PUBLIC_THREADS` and `SEND_MESSAGES_IN_THREADS`. Listen for `THREAD_CREATE` and `THREAD_UPDATE`.

### Stage channels

A Stage (type 13) has speakers and an audience. Start one with `POST /stage-instances` (the bot needs `MANAGE_CHANNELS`, `MUTE_MEMBERS` and `MOVE_MEMBERS`) and end it with `DELETE /stage-instances/{channel.id}`. Audience members are "suppressed" until promoted: the bot edits a member with `PATCH /guilds/{guild.id}/voice-states/{user.id}` and `{"channel_id": "...", "suppress": false}`. A user raises a hand via `request_to_speak_timestamp`. Events: `STAGE_INSTANCE_CREATE/UPDATE/DELETE`.

### Voice

Voice runs on a **separate WebSocket and UDP** connection, so most developers should use a library that supports voice (discord.js with `@discordjs/voice`, discord.py with voice extras, or Lavalink). Requires the `GUILD_VOICE_STATES` intent and `CONNECT` / `SPEAK` permissions. The flow:

1.  Send gateway **op 4** (Voice State Update) with `guild_id` and `channel_id` (null leaves).
2.  Receive `VOICE_STATE_UPDATE` (gives `session_id`) and `VOICE_SERVER_UPDATE` (gives `token` and `endpoint`).
3.  Open `wss://ENDPOINT?v=8`, send **Identify** (`server_id`, `user_id`, `session_id`, `token`), receive **Ready** (`ssrc`, `ip`, `port`, supported `modes`) and keep heartbeating.
4.  Do UDP IP discovery, send **Select Protocol**, and receive **Session Description** with the `secret_key`.
5.  Send Opus audio in RTP packets over UDP, encrypted with an AEAD mode such as `aead_aes256_gcm_rtpsize` or `aead_xchacha20_poly1305_rtpsize`, and set Speaking (op 5) when transmitting.

Discord has been rolling out end-to-end encryption (DAVE) for voice, so use an up-to-date library rather than implementing the protocol yourself, and check the voice docs for current requirements. Audio is 48 kHz stereo Opus in 20 ms frames.

### Modals

Open a modal by responding to a command or component interaction with callback type **9**. You cannot open one from a modal submit or an autocomplete.

```
{ "type": 9, "data": { "custom_id": "feedback", "title": "Send feedback",
  "components": [ { "type": 1, "components": [
    { "type": 4, "custom_id": "msg", "style": 2, "label": "Your message",
      "min_length": 10, "max_length": 1000, "required": true, "placeholder": "Tell us more" } ] } ] } }
```

Limits: title up to 45 chars, up to 5 rows, one text input per row, label up to 45, `max_length` up to 4000, placeholder up to 100. Style 1 is a short single line, style 2 a paragraph. The submit arrives as interaction type **5**; read values from `data.components[].components[]` as `{custom_id, value}` and reply within 3 seconds like any interaction. Discord has been adding newer modal layouts (a Label wrapper that can hold select menus, plus file upload, radio and checkbox fields), so check the components reference for the current shapes.

### Scheduled events

Entity types: **1** stage, **2** voice, **3** external. Stage and voice events need `channel_id`; external events need `entity_metadata.location` and a `scheduled_end_time`. Status moves one way: 1 scheduled -\> 2 active -\> 3 completed, or 4 canceled from scheduled. Needs `MANAGE_EVENTS` (and channel permissions for stage and voice). Gateway: `GUILD_SCHEDULED_EVENT_CREATE/UPDATE/DELETE` and `_USER_ADD/REMOVE` under the `GUILD_SCHEDULED_EVENTS` intent.

### AutoMod

| Trigger        | Value | Metadata                                                              |
|----------------|-------|-----------------------------------------------------------------------|
| KEYWORD        | 1     | `keyword_filter` (wildcards `*word*`), `regex_patterns`, `allow_list` |
| SPAM           | 3     | None                                                                  |
| KEYWORD_PRESET | 4     | `presets`: 1 profanity, 2 sexual content, 3 slurs                     |
| MENTION_SPAM   | 5     | `mention_total_limit` (max 50)                                        |
| MEMBER_PROFILE | 6     | Keyword filters applied to profiles                                   |

Actions: **1** block message (optional `custom_message`), **2** send alert to `channel_id`, **3** timeout for `duration_seconds` (max 4 weeks; not for non-message triggers). Managing rules needs `MANAGE_GUILD`. Each guild has limits per trigger type, such as a handful of keyword rules. Listen for `AUTO_MODERATION_ACTION_EXECUTION` via the `AUTO_MODERATION_EXECUTION` intent; reading `content` in it needs the Message Content intent.

### Audit log

Records admin actions for 45 days. Each entry has `user_id` (who), `target_id`, `action_type`, `changes` (old and new values), and an optional `reason`. Common types: 1 guild update, 10 channel create, 12 channel delete, 20 member kick, 22 ban add, 23 ban remove, 24 member update, 25 member role update, 72 message delete, 73 bulk delete. You can attach your own reason to most moderation calls with the `X-Audit-Log-Reason` header (URL-encoded). Realtime: `GUILD_AUDIT_LOG_ENTRY_CREATE` under the `GUILD_MODERATION` intent.

### Emoji and stickers

Send a custom emoji in message text as `<:name:id>` (or `<a:name:id>` if animated); use `name:id` (URL-encoded) in reaction routes. Emoji images are PNG, JPEG or GIF up to 256 KiB and sent as a base64 data URI; names are 2-32 characters. Stickers are PNG, APNG, GIF or Lottie JSON up to 512 KB (320x320 recommended), created with `multipart/form-data` fields `name`, `description`, `tags` and `file`. Send stickers by passing up to 3 IDs in `sticker_ids` on a message. Both need `MANAGE_GUILD_EXPRESSIONS`; the number you can add depends on the server's boost level. Events: `GUILD_EMOJIS_UPDATE`, `GUILD_STICKERS_UPDATE` (`GUILD_EXPRESSIONS` intent).

## Rate limits in detail

| Header                    | Meaning                                 |
|---------------------------|-----------------------------------------|
| `X-RateLimit-Limit`       | Requests allowed in the bucket          |
| `X-RateLimit-Remaining`   | Requests left before the reset          |
| `X-RateLimit-Reset-After` | Seconds (float) until the bucket resets |
| `X-RateLimit-Bucket`      | Bucket ID; routes can share one         |
| `X-RateLimit-Scope`       | `user`, `global` or `shared`            |

A `429` body is `{"message","retry_after","global"}`. Buckets are keyed by route plus the top-level resource (channel, guild or webhook ID). Also avoid sending lots of invalid requests (401, 403, 429): about 10,000 in 10 minutes can get your IP temporarily blocked.

## Common errors

| Code                          | Meaning                                             |
|-------------------------------|-----------------------------------------------------|
| 10003 / 10004 / 10008 / 10013 | Unknown channel / guild / message / user            |
| 30001                         | Maximum guilds reached                              |
| 40060                         | Interaction already acknowledged                    |
| 50001                         | Missing access (bot can't see the resource)         |
| 50013                         | Missing permissions (role or channel overwrites)    |
| 50035                         | Invalid form body; `errors` pinpoints the bad field |

HTTP statuses: `200/201/204` success, `400` bad payload, `401` bad token, `403` forbidden, `404` not found, `429` rate limited, `5xx` retry with backoff.

## Permissions

Permissions are a bitfield sent as a string. Combine with bitwise OR, and check channel overwrites after role permissions. `ADMINISTRATOR` bypasses everything.

| Permission      | Bit     | Permission                 | Bit               |
|-----------------|---------|----------------------------|-------------------|
| KICK_MEMBERS    | 1\<\<1  | SEND_MESSAGES              | 1\<\<11           |
| BAN_MEMBERS     | 1\<\<2  | MANAGE_MESSAGES            | 1\<\<13           |
| ADMINISTRATOR   | 1\<\<3  | EMBED_LINKS / ATTACH_FILES | 1\<\<14 / 1\<\<15 |
| MANAGE_CHANNELS | 1\<\<4  | READ_MESSAGE_HISTORY       | 1\<\<16           |
| MANAGE_GUILD    | 1\<\<5  | MANAGE_ROLES               | 1\<\<28           |
| ADD_REACTIONS   | 1\<\<6  | MANAGE_WEBHOOKS            | 1\<\<29           |
| VIEW_CHANNEL    | 1\<\<10 | MODERATE_MEMBERS (timeout) | 1\<\<40           |

## OAuth2

**Authorization code flow.** 1) Send the user to `https://discord.com/oauth2/authorize?client_id=ID&response_type=code&redirect_uri=URI&scope=identify%20guilds&state=RANDOM`. 2) Discord redirects back with `code` and `state`; verify `state`. 3) Exchange the code (form-encoded, not JSON):

```
curl -X POST https://discord.com/api/oauth2/token \
  -u "CLIENT_ID:CLIENT_SECRET" \
  -d grant_type=authorization_code \
  -d code=CODE \
  -d redirect_uri=URI
# -> { "access_token", "token_type": "Bearer", "expires_in": 604800, "refresh_token", "scope" }
```

Refresh with `grant_type=refresh_token&refresh_token=...`. Revoke at `POST /oauth2/token/revoke`. Call `GET /oauth2/@me` to see the token's scopes. To add a bot, use scopes `bot applications.commands` plus `&permissions=BITFIELD`.

| Scope                        | Grants                                           |
|------------------------------|--------------------------------------------------|
| identify / email             | User profile / email address                     |
| guilds / guilds.members.read | Guild list / the user's member object in a guild |
| guilds.join                  | Add the user to a guild (bot token required)     |
| bot / applications.commands  | Install the bot / register slash commands        |
| webhook.incoming             | Create a webhook in a channel the user picks     |

## Gateway (WebSocket)

Connect to the URL from `GET /gateway/bot`, adding `?v=10&encoding=json`. Flow: receive **Hello** (op 10) with `heartbeat_interval`, send a heartbeat after interval x random jitter, then send **Identify**. Keep heartbeating; if an ACK is missed, reconnect and Resume using `session_id` and the last sequence `s`.

```
{ "op": 2, "d": {
  "token": "BOT_TOKEN",
  "intents": 33281,
  "properties": { "os": "linux", "browser": "mylib", "device": "mylib" }
} }
```

| Op        | Name                                           | Direction |
|-----------|------------------------------------------------|-----------|
| 0         | Dispatch (events)                              | Receive   |
| 1         | Heartbeat                                      | Both      |
| 2 / 6     | Identify / Resume                              | Send      |
| 3 / 4 / 8 | Presence / Voice state / Request guild members | Send      |
| 7         | Reconnect                                      | Receive   |
| 9         | Invalid session (`d` says if resumable)        | Receive   |
| 10 / 11   | Hello / Heartbeat ACK                          | Receive   |

**Intents** choose which events you get. Privileged ones must be enabled in the Developer Portal.

| Intent                       | Bit    | Intent                       | Bit     |
|------------------------------|--------|------------------------------|---------|
| GUILDS                       | 1\<\<0 | GUILD_MESSAGES               | 1\<\<9  |
| GUILD_MEMBERS (privileged)   | 1\<\<1 | GUILD_MESSAGE_REACTIONS      | 1\<\<10 |
| GUILD_MODERATION             | 1\<\<2 | DIRECT_MESSAGES              | 1\<\<12 |
| GUILD_WEBHOOKS               | 1\<\<5 | MESSAGE_CONTENT (privileged) | 1\<\<15 |
| GUILD_VOICE_STATES           | 1\<\<7 | GUILD_SCHEDULED_EVENTS       | 1\<\<16 |
| GUILD_PRESENCES (privileged) | 1\<\<8 | AUTO_MODERATION_EXECUTION    | 1\<\<21 |

Key events: `READY`, `GUILD_CREATE`, `MESSAGE_CREATE`, `MESSAGE_UPDATE`, `MESSAGE_DELETE`, `GUILD_MEMBER_ADD`, `INTERACTION_CREATE`. Without `MESSAGE_CONTENT`, bots only see message text in DMs, mentions, and their own messages. Large bots must shard: guild ID \>\> 22 mod shard count.

## Resources

- [Developer Portal](https://discord.com/developers/applications): create apps, get tokens, set intents and the interactions URL
- [Official API documentation](https://discord.com/developers/docs)
- [OpenAPI spec](https://github.com/discord/discord-api-spec): generate a typed client in your language
- [discord.js](https://discord.js.org) and [discord.py](https://discordpy.readthedocs.io): popular bot libraries

