# Tribuna Match IQ

Prototype for the Tribuna.com iGaming Product/Project Manager test task.

- Live prototype: https://youngerfornday.github.io/tribuna-match-iq/
- Local: open `index.html` in any browser. One file, no build step, works offline.
- Every screen has its own URL (`#/challenges`, `#/match/bay-bvb/play`, `#/board`, `#/me`), so deep links and the browser back button work. In production these map 1:1 to paths under `/match-iq/`.
- Navigation: a Match IQ sub-nav (Overview / Challenges / Table / You) sits under the site header on every browse screen; the play flow replaces it with one labelled back link. Browse screens use a 1120px two-column layout from 1024px up, the play flow stays a single 620px column.

Flow: match page → 3 blind calls (winner, over/under, scorer) → reveal CaptainAI / fans / market after each call → review the three calls and tap one as the Power Pick, then lock → odds at three partners (bet builder or single) → wait for full-time → result → Match IQ, streak, league, leaderboards → next match.

The "Simulate full-time" button on the waiting screen is a prototype control. In production the result arrives after the real match.

## 1. What is it and what user problem does it solve

Tribuna readers consume a match passively: preview, odds block, match centre, report. There is no personal stake in the outcome, nothing to compare their opinion against, and no reason to come back after the final whistle except the score.

Match IQ turns that into a three-tap game on the match page. The user makes three calls before seeing anyone else's view, then sees how CaptainAI, thousands of fans and the betting market rate the same call. One call becomes a Power Pick. The scenario is locked, scored after full-time, and feeds a persistent Match IQ rating, a match streak, a league tier and two weekly leaderboards: players against CaptainAI, and fanbase against fanbase.

The user problem it solves: "I have an opinion about this match and nowhere to put it." The product problem it solves: it gives every match page a pre-match action, a post-match return visit and a second content click in between.

## 2. iGaming conversion: how it motivates a bet natively

The bet is never the first ask. The order is opinion → comparison → price:

- The user has already formed and locked a prediction before any odds appear. The offer that follows is the price of a scenario they authored, not a banner.
- The locked scenario becomes a bet builder automatically. The Power Pick is also offered as a single, which is the low-friction option for users who would not stake on a three-leg builder.
- "Your call vs market" shows CaptainAI, fan and implied market probabilities for the Power Pick side by side. This is the native bettor motivation, the feeling of knowing something the price does not, presented as information with an explicit "not a guarantee" line.
- Three partners are compared with the best price highlighted, which is the honest version of "compare odds" and a stronger click driver than a single-partner button.
- Guardrails: 18+ everywhere, "odds are illustrative", partner labelling, no countdown pressure on the odds screen (the only countdown is the real kick-off), and a "continue without betting" path that is equally prominent. The existing 1X2 partner block on the match page stays; Match IQ adds a second, higher-intent route to the same partners rather than replacing it.

Lead quality matters more than raw clicks for an affiliate business, so the funnel is measured through to partner postbacks, not just outbound clicks.

## 3. Metrics

North Star Metric: **Weekly Betting-Intent Predictors**, the number of users who both locked at least one prediction and clicked out to a partner in the same week.

Why not pure engagement: "weekly active predictors" can grow while revenue does not, and it can be inflated by power users. Requiring both a lock and a click-out ties the game to the business without counting clicks that never had a prediction behind them.

Support metrics:

| Metric | Definition | Why it matters |
|---|---|---|
| Challenge completion rate | `prediction_locked / challenge_start` | Is the game itself working? Target 60%+ for a three-tap flow. |
| Return-to-result rate | Users with `result_viewed` within 48h of `prediction_locked` | The retention loop. This is what streaks, leaderboards and the "result is in" push are for. |
| Pages per predictor session | Content pages viewed in sessions that include a lock | View depth: the reveal links to the preview, the result links to the report. |

Guardrails: FTD and registration rate from partner postbacks (lead quality), CTR of the existing partner odds block (no cannibalisation), responsible-gambling flags and complaints.

Hypothesis: we believe that a blind-then-reveal prediction game on match pages will raise partner click-outs among logged-in readers by 20% and 7-day match-page return by 15%. We will know we are right when the NSM grows for four consecutive weeks without a drop in partner FTD rate or existing block CTR.

First experiments:

1. Blind reveal vs open reveal (probabilities shown before the user picks). Expect blind to win on completion and on click-out, because the comparison only means something after a commitment.
2. Odds screen before Lock vs after Lock. Expect after to win on lead quality, because the user has finished the game and is not being interrupted.

All events are visible in the prototype's Metrics panel at the bottom of every screen.

## Prototype notes

- Everything is mocked: fan percentages, CaptainAI probabilities and reasoning, partner odds, the leaderboards and the match results.
- Market % is computed in code from the average partner odds: implied probability with the bookmaker margin removed for mutually exclusive markets, raw implied probability for the scorer market.
- Partners are "Partner A/B/C" on purpose. Real brands with invented odds would be a compliance problem in a demo.
- Fans % has a cold-start problem: in the first days of a match the widget should show only CaptainAI and market until a vote threshold is reached.
- Profile state (Match IQ, streak, hit rate, club) persists in `localStorage`. Clearing site data resets it.

## Not built and why

Kept out of the prototype so the core flow stays readable. Each is a follow-up, not a missing piece:

- Badge shelf and a visual share card (badges are toasts, sharing copies text).
- "Save streak to your Tribuna account" prompt after the first result, the soft registration gate.
- Weekend Quest ("predict three matches this weekend"), streak freeze, friend duels.
- Real odds feed with geo-specific partner ordering, push notifications for "result is in" and "streak at risk", weekly reset of leaderboards.

Precedents: Sky Bet Super 6 is the reference for predictor-to-sportsbook; FotMob and OneFootball predictions are the content-side reference. Rollout: one top match per week, then top-5 leagues, then a weekly league with partner-funded prizes.

## Built with

Vibe-coded with Claude Code. Single HTML file, vanilla JavaScript, no dependencies.
