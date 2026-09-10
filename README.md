# Tribuna Match IQ

Prototype for the Tribuna.com iGaming Product/Project Manager test task.

- **Live prototype:** https://youngerfornday.github.io/tribuna-match-iq/
- **Answer deck (Ukrainian):** https://youngerfornday.github.io/tribuna-match-iq/deck.html — the three answers as a 14-slide deck, built from the prototype's own tokens. Arrow keys navigate, Cmd+P prints one slide per page.
- **Integration contract:** [`docs/integration.md`](docs/integration.md) — the three things a browser cannot host (odds feed, push, account) specified to the point where a backend team can build them without touching this front end: payload shapes, suspension and close semantics, notification rules, sync and merge conflicts, the event list and partner attribution.
- **Run it locally:** open `index.html` in any browser. No build step, no dependencies. Over HTTPS it installs as a PWA and works offline; the rest of the repo is the illustrations, icons and social card in `art/`, a manifest and a service worker.

Flow: match page → 3 blind calls (winner, over/under, scorer), each one selected and then committed, so a mis-tap on a phone is not a scored call → reveal CaptainAI / fans / market after each commit → review the three calls and tap one as the Power Pick, then lock → odds at three partners (bet builder or single) → wait for full-time → result → Match IQ, streak, league, leaderboards → next match.

The "Simulate full-time" button on the waiting screen is a prototype control. In production the result arrives after the real match.

## How the prototype is put together

- Every screen has its own URL (`#/challenges`, `#/match/bay-bvb/play`, `#/board`, `#/me`), so deep links and the browser back button work. In production these map 1:1 to paths under `/match-iq/`.
- A Match IQ sub-nav (Overview / Challenges / Table / You) sits under the site header on every browse screen; the play flow replaces it with one labelled back link, so every screen names exactly one way out.
- Width follows the task: browse screens open up to 1120px, the play flow stays a single 620px column where a wide measure would hurt.
- **Overview** reads top to bottom: hero, how it works, prize banner, weekly leaderboard, next open match, rules. The leaderboard shows the top 10 and expands to the top 100 of the field. The next-match card exists so the page does not dead-end on the terms block: it carries the game's own numbers - calls locked, time to kick-off, where the crowd stands - and one route back into a match, with no second betting surface under the prize banner.
- **Challenges** is one column: a progress strip for the week, then the fixtures. The next match to call is the only highlighted row and carries the only call to action on the screen; matches already played show what you called and what it scored.
- **Table** is the ranking: the prize, then where you stand, then the table itself.
- **You** is your own week — rating, league, streak, hit rate against CaptainAI and the market, every call you locked this week, and the next challenge. The leaderboard lives on Table alone, so the two screens do not repeat each other.
- The prize banner has two states. Before a first prediction it sells the entry ticket ("your first prediction unlocks the welcome offer") and names the weekly prize underneath; once a prediction is locked it switches to the CaptainAI target and the user's own distance from it ("CaptainAI is on 120. You are on 95 - 26 points to go"). The prize is also stated in the hero chips, so it is on screen before any scrolling.
- Partner bonuses are marked as bonuses: a badge on every price that carries one, and an accent card with the reward as its headline.

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
- The order is enforced, not just stated. CaptainAI's slip of the week only appears once the user has locked something, so a first-timer's challenges screen never ends in a bet. The waiting screen states where the prices are and does not re-ask after the user has already declined them on the odds screen. The result screen closes on the next match as its button, with the three prices below it as reference rather than as the last word.
- Guardrails: 18+ everywhere - including the welcome-offer sheet, which gates on the same confirmation as the handoff and reports the same click-out event - "odds are illustrative", partner labelling, one price per selection at handoff, no countdown pressure on the odds screen (the only countdown is the real kick-off), and a "continue without betting" path that is equally prominent. The existing 1X2 partner block on the match page stays; Match IQ adds a second, higher-intent route to the same partners rather than replacing it.

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
| Welcome offer claim rate | `welcome_offer_claimed / welcome_offer_shown` | The offer dialog fires once, after a user's first click-out. It is also where the weekly prize is introduced, so a low claim rate with a high `welcome_offer_to_table` rate still counts as working. |

Guardrails: FTD and registration rate from partner postbacks (lead quality), CTR of the existing partner odds block (no cannibalisation), responsible-gambling flags and complaints.

Hypothesis: we believe that a blind-then-reveal prediction game on match pages will raise partner click-outs among logged-in readers by 20% and 7-day match-page return by 15%. We will know we are right when the NSM grows for four consecutive weeks without a drop in partner FTD rate or existing block CTR.

First experiments:

1. Blind reveal vs open reveal (probabilities shown before the user picks). Expect blind to win on completion and on click-out, because the comparison only means something after a commitment.
2. Odds screen before Lock vs after Lock. Expect after to win on lead quality, because the user has finished the game and is not being interrupted.

After a user's first click-out the prototype shows a one-time welcome-offer dialog: pick one partner offer to claim, or go to the weekly table, where beating CaptainAI's weekly score earns another partner offer on Monday. It is dismissible, fires once per profile, and every offer in it is labelled illustrative.

All events are visible in the prototype's Metrics panel at the bottom of every screen.

## Prototype notes

- The Monday reset is real: the profile carries an ISO week key and rolls itself over. Entries, the quest, the humans-versus-model score and last week's rank clear; Match IQ, league, badges and the streak carry over, which is exactly the split the rules promise.
- One streak freeze per week, spent automatically when a kick-off passes without a call, and the profile says when it saved you.
- A profile can move without an account: "Move this profile to another device" packs progress, badges, rivals and the week into a link, and the receiving device asks before overwriting what it already has.
- Price reads go through one interface rather than being scattered: `oddsFeed.price()`, `oddsFeed.isSuspended()`, `oddsFeed.updatedAt()`. That is the seam a real feed plugs into, and it is why the drift and the suspension could be added without touching the screens.
- Rivals are the honest version of a friends list: the profile remembers the head-to-head record of everyone you actually duelled, and offers a rematch on the next open match. Nothing is invented about people you have not played.
- Duels travel in the link: the challenger's slip is encoded into the URL, the opponent plays the same match blind, and both slips are only compared at full-time. Showing the challenger's calls first would break the blind-then-reveal rule the whole game rests on.
- The weekly quest is computed from the fixtures, not stored: call every Sunday match and the week pays +25 Match IQ, awarded the moment the last one is locked.
- Installable and offline: a manifest, icons and a service worker that caches the shell. Navigations go network-first so a deploy lands immediately, and fall back to the cached page when there is no connection. A prototype gets opened on phones in bad reception, and a link that fails there is a link nobody looks at.
- One partner at a time can suspend its market. The row hides the price and disables its button, the best price moves to a live book, and a suspension while the handoff sheet is open blocks the exit instead of sending someone out on a price that no longer exists.
- Prices move while you are on the odds screen, and the handoff will not send you out on a price you did not accept: if it moved between opening the sheet and continuing, the sheet says so and asks again. The feed is simulated - the prices are fixed data - but the behaviour it forces on the UI is the real one.
- Partner order follows the market (`PARTNER_ORDER` by geo, UA leads with FAVBET), while the best price is highlighted wherever it sits. Ordering is a market decision; price is not.
- The first result asks whether to save the streak to a Tribuna account. It is the soft registration gate, shown once, and nothing is gated behind refusing it.
- Sharing produces a card, not a line of text: a 1080x1080 image drawn on a canvas with your three calls, the Power Pick and, after full-time, the points each call scored. It goes through the system share sheet where the browser accepts files, downloads otherwise, and falls back to copied text.
- Badges persist. Five of them, earned ids stored on the profile, shown on a shelf of medals where the locked ones stay visible with the condition that unlocks them. A badge earned at full-time lands in the same block that counts the points up, rather than in a toast that flies away.

- Everything is mocked: fan percentages, CaptainAI probabilities and reasoning, partner odds, the leaderboards and the match results. Amounts are in euro.
- The weekly table is a real field. One curve maps a score to a place among the 12,480 players the copy claims, and everything derives from it: the generated rows below the named players, your own rank, and CaptainAI's. So "#4,312 of 12,480" is a number the table can defend rather than a label over 131 rendered rows, and reaching the top of it takes a week of good calls rather than one lucky match.
- Each illustration is levelled for the block it sits in and inserted with the same radial mask, bleeding off the block's edge. Forcing one shared treatment on both — a pure black background for each — matched them to each other but made the banner read as a pasted rectangle, because its block is the lightest on the page.
- Two pieces of artwork, both generated for this prototype in one style: CaptainAI, the model you play against, on the overview hero, and the crowned winner holding the trophy and the free bet in the prize banner. Nothing is taken from an operator's site: lifting a casino's character art into a public demo is a copyright problem, and it would be the first thing an operator's own legal team objects to.
- The banner character is deliberately the winning player, not a gambling mascot aimed at a young audience: crowned, holding the trophy the weekly table is played for, with the free bet as the second prize.
- Market % is computed in code from the average partner odds: implied probability with the bookmaker margin removed for mutually exclusive markets, raw implied probability for the scorer market.
- Partners are Tribuna's own betting partners: Parimatch, GG.BET and FAVBET all have their own sections under tribuna.com/en/betting/sportsbook/, and Tribuna's international team has written about working with GG.BET Affiliates. Every price and welcome offer here is illustrative, and says so in the handoff sheet, the game rules and the site footer: live terms differ by market and change often, so a demo must not read as a live offer.
- Fans % has a cold-start problem: in the first days of a match the widget should show only CaptainAI and market until a vote threshold is reached.
- Profile state (Match IQ, streak, hit rate, club) persists in `localStorage`. Clearing site data resets it.

## Not built and why

Kept out of the prototype so the core flow stays readable. Each is a follow-up, not a missing piece:

- Push notifications for "result is in" and "streak at risk", and the weekly reset job behind them. The prototype says where they belong rather than faking a permission prompt.
Three of them are services rather than screens, and each is specified in [`docs/integration.md`](docs/integration.md) rather than left as a gesture:

- The odds provider. Every price read already goes through one interface (`oddsFeed`), and the document defines the payload, the suspended/closed semantics, the refresh floor and the failure modes a real feed must honour.
- Server-side notifications. The prototype uses the browser's own permission and schedules kick-off and full-time locally; the document defines the three server events, the deduplication rule, quiet hours and the ban on promotional pushes.
- An account. Progress already moves between devices through a transfer link; the document defines the sync payload, per-match conflict resolution, the anonymous merge and who owns settlement.

Precedents: Sky Bet Super 6 is the reference for predictor-to-sportsbook; FotMob and OneFootball predictions are the content-side reference. Rollout: one top match per week, then top-5 leagues, then a weekly league with partner-funded prizes.

Weekly prize: every player who finishes the week above CaptainAI's score receives the lead partner's welcome offer, credited on Monday when the table resets. The target is the model rather than a place in the table, because a place depends on twelve thousand strangers while the model is a visible, chaseable number - and beating it is what the first screen already promises. CaptainAI calls every match of the week, so the bar moves with its own results, and clearing it takes three or four well-called matches rather than one. It is partner-funded rather than a Tribuna cash prize, which is what makes it self-financing: the prize is the same offer the handoff already promotes, so the reward and the conversion event are one thing. Carried on a banner on the overview, table and profile screens, always with the partner terms, new-customers-only wording and 18+.

## Built with

Vibe-coded with Claude Code. One HTML file of vanilla JavaScript with no dependencies, plus the illustrations in `art/` and the answer deck in `deck.html`.
