# TrendRadar Evening-Only Email Summary And Keyword Tuning Design

## Goal

Change the current scheduling behavior so the project sends exactly one email summary each day in the evening, instead of allowing pushes throughout the day.

At the same time, tune the tracked keyword set so the daily summary better matches the user's long-term interests:

- AI / technology
- finance / investing
- new energy / automotive
- cost estimation / construction
- general important headlines

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
- Replace the current tracked keyword set with a curated mixed-interest set aligned to the user's stated themes.

## Recommended Approach

Update the `morning_evening` preset in `config/timeline.yaml`:

- Change the preset `default.push` from `true` to `false`.
- Keep `default.collect = true` so data is still accumulated all day.
- Keep the `evening_summary` period active with:
  - `push = true`
  - `report_mode = "daily"`
  - `once.push = true`

This preserves the existing workflow and only changes when push notifications are allowed.

Update `config/frequency_words.txt` with a layered keyword strategy:

- AI / technology
- finance / investing
- new energy / automotive
- cost estimation / construction
- general important headlines

The keyword set should stay broad enough to avoid empty summaries, but focused enough to reduce low-value noise.

## Why This Approach

- Lowest risk: it reuses the existing push and email pipeline.
- Clear behavior: no hidden logic in the notification layer.
- Good data quality: all-day collection still feeds a richer evening summary.
- Operationally simple: the likely send time remains around `20:33` Beijing time, with `21:33` acting as a natural retry opportunity if the earlier run fails.
- Better relevance: the keyword update reshapes the output into a domain-focused briefing rather than a generic trend feed.

## Keyword Strategy

The new keyword list will be organized around five practical buckets.

### 1. AI / Technology

Representative terms:

- AI
- OpenAI
- NVIDIA / 英伟达
- chips / 芯片 / 半导体
- large models / 大模型
- robotics / 机器人
- cloud / 云计算
- compute / 算力

### 2. Finance / Investing

Representative terms:

- A-shares / A股
- Hong Kong stocks / 港股
- U.S. stocks / 美股
- funds / 基金
- macro / 宏观
- central bank / 央行
- Bitcoin / 比特币
- crypto / 加密货币
- gold / 黄金
- quantitative trading / 量化

### 3. New Energy / Automotive

Representative terms:

- new energy / 新能源
- EV / 电动车
- battery / 电池 / 锂电
- photovoltaic / 光伏
- energy storage / 储能
- charging / 充电桩
- Tesla / 特斯拉
- BYD / 比亚迪
- intelligent driving / 智能驾驶

### 4. Cost Estimation / Construction

Representative terms:

- cost estimation / 造价
- construction / 建筑
- engineering / 工程
- infrastructure / 基建
- housing / 住建
- real estate / 地产
- bidding / 招投标
- BIM
- cement / 水泥
- steel / 钢筋

### 5. General Important Headlines

Representative terms:

- policy / 政策
- official release / 发布 / 官方
- State Council / 国务院
- MIIT / 工信部
- NDRC / 发改委
- CSRC / 证监会
- breaking / 突发

These general terms are intentionally limited and serve as a safety net for major developments that overlap with the user's core interests.

## Expected Outcome

After the change:

- The workflow still runs hourly.
- Reports and stored data continue updating during the day.
- Email is only sent once in the evening summary window.
- The user receives one daily summary email instead of daytime updates.
- The summary content becomes more aligned with the user's chosen sectors and is less dominated by unrelated broad-appeal topics.

## Risks and Mitigations

- If the `20:33` run fails, the `21:33` run may still send the summary because it is inside the same summary window.
- If AI analysis or translation remains enabled without API credentials, logs may still show AI-related warnings, but email delivery itself can still work.

## Testing Plan

1. Update the schedule config.
2. Update the keyword file with the curated mixed-interest list.
3. Trigger the workflow manually during a non-summary period and confirm no email is sent.
4. Trigger the workflow during the evening summary period and confirm one email is sent.
5. Verify the report mode is `daily` in the evening email behavior.
6. Inspect the resulting report and confirm that the major matched items cluster around the intended themes.
