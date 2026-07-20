# Habito: Private Habit Tracker — Bot specification

**Archetype:** custom

**Voice:** encouraging and uplifting — write every user-facing message, button label, error, and empty state in this voice.

Habito is a private Telegram bot that helps users build habits through gentle reminders, one-tap check-ins, and flexible scheduling (daily/weekdays/N× per week). It tracks streaks, completion rates, and milestones with an encouraging tone, ensuring all data remains private per user. Missed days receive supportive nudges, and weekly summaries highlight progress without punitive language.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual Telegram users seeking private habit tracking
- People who prefer simple, non-intrusive productivity tools

## Success criteria

- Users can create and manage habits with flexible scheduling
- Daily reminders with one-tap check-in buttons function reliably
- Streak tracking and weekly summaries update correctly
- Missed/failed habit records trigger supportive messages without duplicates

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Begin onboarding or return to main menu
- **Create New Habit** (button, actor: user, callback: habit:new) — Initiates habit creation flow
  - inputs: habit name, schedule type, reminder time, optional note
  - outputs: habit summary confirmation
- **/habits** (command, actor: user, command: /habits) — View compact habit list with status indicators
- **Pause/Resume** (button, actor: user, callback: habit:toggle_pause) — Toggle habit active/paused status
- **Edit Schedule** (button, actor: user, callback: habit:edit_schedule) — Modify habit's schedule type and parameters

## Flows

### onboarding
_Trigger:_ /start

1. Welcome message
2. Timezone selection (auto-detect or manual)
3. First habit creation prompt

_Data touched:_ User

### habit_creation
_Trigger:_ habit:new

1. Request habit name
2. Select schedule type
3. If flexible N×/week: request N value (1-7)
4. Choose reminder time (default suggested)
5. Add optional note
6. Confirm habit creation

_Data touched:_ Habit, User

### weekly_summary
_Trigger:_ weekly_boundary

1. Generate summary of completions per habit
2. Compare with previous week's progress
3. Include milestone celebrations if applicable

_Data touched:_ DayRecord, Streaks & stats

### check_in
_Trigger:_ scheduled_reminder

1. Send reminder message with check-in buttons
2. Record check-in status if button clicked
3. Update streaks and display confirmation

_Data touched:_ DayRecord, Streaks & stats

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user profile with preferences
  - fields: telegram_id, timezone, language, week_start_day
- **Habit** _(retention: persistent)_ — User-defined habit with scheduling rules
  - fields: id, title, description, reminder_time, schedule_type, n_per_week, is_active, created_at, streak_start_override
- **DayRecord** _(retention: persistent)_ — Daily habit completion status
  - fields: date, habit_id, status, check_in_time
- **Streaks & stats** _(retention: persistent)_ — Derived analytics for habits
  - fields: current_streak, best_streak, total_completions, completion_percentage

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Create/delete habits
- Edit habit schedules and notes
- Pause/resume habits
- Manually mark day records as done/missed/failed
- Configure week start boundary

## Notifications

- Scheduled reminders with check-in buttons
- Milestone celebrations (streaks, completions)
- Weekly summary reports
- Gentle encouragement after missed days

## Permissions & privacy

- All data private to user's bot chat
- No sharing between users
- Local timezone-based data processing

## Edge cases

- Timezone changes mid-week
- Multiple check-in attempts for same day
- Flexible N×/week habits reaching quota early
- Week boundary changes after habit creation

## Required tests

- End-to-end habit creation with all schedule types
- Check-in button idempotency across multiple taps
- Weekly summary generation with milestone detection
- Pause/resume behavior for reminder suppression

## Assumptions

- Telegram can provide initial timezone guess
- Flexible schedule algorithm will evenly distribute reminders
- Monday is default week start for most users
