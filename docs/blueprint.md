# CryptoPing — Bot specification

**Archetype:** finance

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

CryptoPing is a Telegram bot that allows users to maintain private cryptocurrency watchlists and receive price-threshold and percent-change alerts. Users can add/remove coins via inline buttons, query current prices, and set optional daily digests. The bot respects quiet hours and rate-limits alerts to avoid spam. The bot owner receives analytics on active users and most frequent alerts.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual crypto traders and holders
- Non-technical users who prefer simple inline controls

## Success criteria

- Users can create and manage watchlists with price and percent-change alerts
- Users receive timely and non-spammy alerts respecting quiet hours
- Owner dashboard shows active user count and alert frequency analytics

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with quick buttons for managing watchlists, price checks, and settings
- **/price** (command, actor: user, command: /price) — Check current price of a specific coin or all watchlist coins
- **Manage Watchlist** (button, actor: user, callback: watchlist:manage) — Open the watchlist management menu with options to add, remove, and configure alerts for coins
- **Settings** (button, actor: user, callback: settings:open) — Configure quiet hours, digest time, and alert cooldown settings
- **Add Coin** (button, actor: user, callback: watchlist:add:predefined) — Quickly add predefined coins like Bitcoin, Ethereum, and Toncoin to the watchlist
- **Add Custom Ticker** (button, actor: user, callback: watchlist:add:custom) — Add a custom cryptocurrency ticker to the watchlist

## Flows

### Onboarding
_Trigger:_ /start

1. Show welcome message with quick buttons for Manage Watchlist, /price, and Settings
2. Ask for optional timezone if not inferable
3. Confirm default quiet hours (23:00-07:00) or allow customization

_Data touched:_ User profile

### Manage Watchlist
_Trigger:_ watchlist:manage

1. Show list of watched coins with inline buttons for each item
2. Allow adding predefined coins (Bitcoin, Ethereum, Toncoin)
3. Allow adding custom tickers via text input
4. Allow removing coins from the watchlist

_Data touched:_ Watchlist item

### Create Price Alert
_Trigger:_ alert:create:price

1. Ask user to select direction (above/below)
2. Ask for target price and currency
3. Confirm the alert rule with a summary message

_Data touched:_ Watchlist item

### Create Percent-Change Alert
_Trigger:_ alert:create:percent

1. Ask for percentage threshold
2. Ask for time window (15m, 1h, 4h, 24h)
3. Confirm the alert rule with a summary message

_Data touched:_ Watchlist item

### Price Check
_Trigger:_ /price

1. If ticker provided, show current price, 24h change, and timestamp
2. If no ticker or 'all', show prices for all watchlist coins and active thresholds

_Data touched:_ Watchlist item, Alert event record

### Morning Digest
_Trigger:_ digest:send

1. Send daily summary at user-selected time
2. Include notable price movements and current prices
3. Include any alerts that triggered since last digest

_Data touched:_ Alert event record, User profile

### Owner Dashboard
_Trigger:_ dashboard:show

1. Show active user count
2. Show alert-type frequency counters
3. Show top tickers by triggers

_Data touched:_ Owner analytics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Telegram user id, timezone, quiet hours, morning-digest time, alert cooldown settings, and watchlist
  - fields: Telegram user id, timezone, quiet hours start, quiet hours end, digest time, alert cooldown settings, watchlist
- **Watchlist item** _(retention: persistent)_ — Cryptocurrency ticker, display name, and alert rules
  - fields: ticker, display name, price thresholds, percent-change alerts, enabled flag
- **Alert event record** _(retention: persistent)_ — Record of triggered alerts with details
  - fields: ticker, timestamp, old price, new price, percentage change, alert type, delivered status
- **Owner analytics** _(retention: persistent)_ — Aggregated metrics for the bot owner
  - fields: active user count, alert-type frequency, top tickers by triggers

## Integrations

- **Telegram** (required) — Bot API messaging for user interface and notifications
- **Price data API** (required) — External market price data source
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View active user count
- View alert-type frequency counters
- View top tickers by triggers

## Notifications

- Price-threshold alerts
- Percent-change alerts
- Morning digest summaries
- Quiet hours alert consolidation

## Permissions & privacy

- User watchlists and settings are private and only accessible to the user and bot owner for aggregated analytics
- No user data is shared with third parties
- Price data is only used for alert triggering and display

## Edge cases

- Price data source failures with retry and fallback behavior
- Ambiguous or unknown tickers with helpful suggestions
- Quiet hours during which alerts are queued and delivered after hours end
- Rate-limiting to prevent alert spamming

## Required tests

- Verify alert triggering and suppression during quiet hours
- Test price-threshold and percent-change alert creation and confirmation
- Validate morning digest content and timing
- Ensure user data privacy and correct aggregation for owner dashboard

## Assumptions

- Timezone defaults to Telegram-provided or UTC
- Price data retries silently with fallback messages
- Predefined quick-add coins are Bitcoin, Ethereum, and Toncoin
- Default quiet hours are 23:00-07:00
- Default cooldowns are 4h for price-threshold and 1h for percent-change alerts
- Morning digest includes notable triggers and current prices
