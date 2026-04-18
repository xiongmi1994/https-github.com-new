# TrendRadar Evening-Only Email Summary Design

## Goal

Change the current scheduling behavior so the project sends exactly one email summary each day in the evening, instead of allowing pushes throughout the day.

## Current State

- GitHub Actions triggers the crawler every hour at `:33`.
- The active schedule preset is `morning_evening`.
- In the current preset, the default behavior still allows pushing outside the evening summary window.
- Email delivery itself is already implemented and configured through GitHub Actions Secrets.

## Desired Behavior

- Keep hourly scheduled runs so the system continues collecting data all day.
- Do not send email during the daytime or other non-summary periods.
- Send one evening summary email during the existing `20:00-22:00` summary window.
- Use `daily` report mode for that email so the message contains the day's full summary.
- Keep the current hourly workflow schedule unchanged.

## Recommended Approach

Update the `morning_evening` preset in `config/timeline.yaml`:

- Change the preset `default.push` from `true` to `false`.
- Keep `default.collect = true` so data is still accumulated all day.
- Keep the `evening_summary` period active with:
  - `push = true`
  - `report_mode = "daily"`
  - `once.push = true`

This preserves the existing workflow and only changes when push notifications are allowed.

## Why This Approach

- Lowest risk: it reuses the existing push and email pipeline.
- Clear behavior: no hidden logic in the notification layer.
- Good data quality: all-day collection still feeds a richer evening summary.
- Operationally simple: the likely send time remains around `20:33` Beijing time, with `21:33` acting as a natural retry opportunity if the earlier run fails.

## Expected Outcome

After the change:

- The workflow still runs hourly.
- Reports and stored data continue updating during the day.
- Email is only sent once in the evening summary window.
- The user receives one daily summary email instead of daytime updates.

## Risks and Mitigations

- If the `20:33` run fails, the `21:33` run may still send the summary because it is inside the same summary window.
- If AI analysis or translation remains enabled without API credentials, logs may still show AI-related warnings, but email delivery itself can still work.

## Testing Plan

1. Update the schedule config.
2. Trigger the workflow manually during a non-summary period and confirm no email is sent.
3. Trigger the workflow during the evening summary period and confirm one email is sent.
4. Verify the report mode is `daily` in the evening email behavior.
