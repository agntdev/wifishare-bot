# WiFiShare Bot — Bot specification

**Archetype:** community

**Voice:** friendly and helpful — write every user-facing message, button label, error, and empty state in this voice.

A public Telegram bot that collects and shares Wi-Fi credentials (SSID + password) with auto-sanitization. Submissions are published to a public channel and sent privately to submitters with QR codes for easy connection.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- public Telegram users
- venue operators
- travelers

## Success criteria

- Public Wi-Fi entry posted to owner's channel
- Private reply with QR code sent to submitter
- Auto-sanitized spam rejection

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with Add/Find options
- **/add** (command, actor: user, command: /add) — Start Wi-Fi submission form
- **/find** (command, actor: user, command: /find <term>) — Search existing entries by SSID/label
- **/remove** (command, actor: admin, command: /remove <id>) — Delete entry by ID
- **/edit** (command, actor: admin, command: /edit <id>) — Edit existing entry

## Flows

### Submission flow
_Trigger:_ /add or Add button

1. Request SSID
2. Request password
3. Optional label/router IP
4. Auto-sanitize
5. Post to public channel
6. Send private reply with QR

_Data touched:_ Wi-Fi entry, Submission record

### Search flow
_Trigger:_ /find <term>

1. Validate search term
2. Query matching entries
3. Format results with QR codes
4. Cap at 10 results

_Data touched:_ Wi-Fi entry

### Moderation flow
_Trigger:_ Admin command

1. Verify admin status
2. Modify entry fields
3. Update public/private records

_Data touched:_ Wi-Fi entry, Submission record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Wi-Fi entry** _(retention: persistent)_ — Stored credential with metadata
  - fields: ssid, password, label, router_ip, submitter_username, timestamp, sanitized_flag
- **Submission record** _(retention: persistent)_ — Tracking published message IDs and edits
  - fields: telegram_message_id, submitter_id, entry_id

## Integrations

- **Telegram** (required) — Bot API messaging and channel posting
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Set public channel via /setchannel
- Approve/reject entries via moderation commands
- View submission analytics

## Notifications

- Public channel post confirmation
- Private submission QR code delivery
- Moderation action alerts

## Permissions & privacy

- Store minimal user metadata (username only)
- Auto-sanitize inputs for spam
- Publicly display submissions with user consent

## Edge cases

- Spam submission bypasses
- QR code generation failure
- Duplicate entries from same submitter

## Required tests

- End-to-end submission flow with sanitization
- Search result accuracy with 10-result cap
- Moderation command permissions

## Assumptions

- Owner will pre-configure public channel
- QR code generation uses WPA2 standard
- Auto-sanitization handles common spam patterns
