# StreakBuddy — Bot specification

**Archetype:** custom

**Voice:** encouraging and supportive — write every user-facing message, button label, error, and empty state in this voice.

StreakBuddy is a private Telegram bot that helps users track multiple personal habits with time-zone-aware reminders, one-tap check-ins, and encouraging streak tracking. It supports daily, weekday, and weekly schedule types, and provides weekly summaries and milestone celebrations.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual Telegram users seeking a simple, private habit tracker with one-tap check-ins and streak motivation

## Success criteria

- User can create and manage multiple habits with custom schedules
- Users receive time-zone-aware reminders with one-tap actions
- Users can view weekly summaries and milestone celebrations
- All habit data remains private and accessible only to the user

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu and begin onboarding
- **/habits** (command, actor: user, command: /habits) — Show all habits at a glance with next reminder times and streaks
- **/week** (command, actor: user, command: /week) — Show a weekly summary of habit progress
- **Create New Habit** (button, actor: user, callback: habit:create) — Start the habit creation flow
  - inputs: Habit title, Schedule type, Preferred reminder time, Optional start date
  - outputs: New habit added to user's list
- **Mark Done** (button, actor: user, callback: habit:mark_done) — Mark a habit as done for the current day
  - inputs: Habit ID, Date
  - outputs: Day record updated as done
- **Not Today** (button, actor: user, callback: habit:mark_missed) — Mark a habit as missed for the current day
  - inputs: Habit ID, Date
  - outputs: Day record updated as missed
- **Fail** (button, actor: user, callback: habit:mark_failed) — Mark a habit as failed for a specific past day
  - inputs: Habit ID, Date
  - outputs: Day record updated as failed
- **Pause/Unpause** (button, actor: user, callback: habit:toggle_pause) — Pause or unpause a habit
  - inputs: Habit ID
  - outputs: Habit paused/unpaused status updated

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message with brief intro
2. Prompt for first habit creation
3. Auto-detect time zone with option to change
4. Create first habit with name, schedule type, preferred time

_Data touched:_ User, Habit

### Habit Creation
_Trigger:_ habit:create

1. Prompt for habit title
2. Select schedule type (daily, weekdays, N times/week)
3. Set preferred reminder time
4. Optional: set start date

_Data touched:_ Habit

### Reminder Handling
_Trigger:_ scheduled reminder

1. Send reminder message with 'Mark Done' and 'Not Today' buttons
2. Handle button presses to update day status
3. For N times/week schedules, calculate remaining checks and update habit detail

_Data touched:_ Day record, Habit

### Weekly Summary
_Trigger:_ /week

1. Display 7-day grid for each habit
2. Show completion percentage and current/best streaks
3. Include encouraging text for missed days

_Data touched:_ Day record, Metrics

### Milestone Celebration
_Trigger:_ streak milestone reached

1. Detect when user reaches 7/14/30/60/100 day milestones
2. Send celebratory message with encouraging note
3. Update best streak if applicable

_Data touched:_ Metrics

### Manual Edit
_Trigger:_ habit:edit

1. Show habit detail with current status
2. Allow editing of past day records
3. Update habit schedule or time if needed

_Data touched:_ Habit, Day record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user account with time zone preference
  - fields: Telegram ID, Time zone, Creation date
- **Habit** _(retention: persistent)_ — User-defined habit with schedule and status
  - fields: Title, Schedule type, Preferred reminder time, Paused flag, Creation date
- **Day record** _(retention: persistent)_ — Daily status for a habit
  - fields: Date (local time), Status (done/missed/failed/paused-ignored)
- **Metrics** _(retention: persistent)_ — Streak and completion statistics
  - fields: Current streak, Best streak, Completion percentage (windowed)
- **Milestone** _(retention: persistent)_ — Predefined celebration thresholds
  - fields: Threshold days, Celebration message

## Integrations

- **Telegram** (required) — Bot API messaging for all user interaction and reminders
- **Time Zone Handling** (required) — Ensure local time for scheduling and day boundaries
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create and manage habits
- Edit past day records
- Pause/unpause habits
- View weekly summaries
- Receive milestone celebrations

## Notifications

- Time-zone-aware reminders with one-tap actions
- Weekly summary notifications
- Milestone celebration notifications

## Permissions & privacy

- All habit data is private and accessible only to the user's Telegram account
- No analytics or sharing unless explicitly enabled later
- Last-write-wins for simultaneous edits from multiple devices

## Edge cases

- User changes time zone after habit creation
- User tries to mark a day as done twice
- User reaches multiple milestones simultaneously
- User has multiple habits with overlapping reminders
- User pauses a habit mid-week for N times/week schedule

## Required tests

- End-to-end onboarding flow from /start to first habit creation
- Reminder delivery at correct local time with one-tap actions
- Weekly summary display with 7-day grid and completion stats
- Milestone celebration detection and message delivery
- Idempotent 'Mark Done' button press handling
- Time zone change handling for existing habits

## Assumptions

- Telegram provides reliable time zone detection
- Users expect local time for all scheduling and day boundaries
- Idempotent 'Mark Done' is preferred to prevent accidental double-counting
- N times/week schedule will distribute reminders evenly by default
- Milestone thresholds are tasteful and motivating without being intrusive
