# Match IQ — integration contract

Three things in this product cannot live in a browser: the odds feed, notifications to a
closed app, and an account that survives a cleared browser. This document specifies them
precisely enough that a backend team can build them without a single change to the front
end, and it is written against the prototype as deployed, not against an intention.

Everything else — the game, scoring, weekly reset, quest, badges, duels, prize logic,
sharing, offline — already runs client-side and needs no service.

---

## 1. Odds feed

### The seam that already exists

Every price read in the prototype goes through one object:

```js
oddsFeed.price(oddsArray, partnerIndex)  // what this selection pays at that partner
oddsFeed.isSuspended(partnerIndex)       // is that partner's market open
oddsFeed.updatedAt()                     // when the last update arrived, ms epoch
```

A real provider replaces the mock implementation. Nothing else in the UI changes: the
price-movement acceptance, the suspended row, the best-price highlight and the bet builder
all read through those three calls.

### What the service must deliver

`GET /match-iq/odds?match={matchId}` and a push channel (SSE or WebSocket) on the same
resource. One payload shape for both:

```json
{
  "match": "ars-liv",
  "updatedAt": 1789000000000,
  "partners": [
    {
      "id": "pm",
      "status": "open",
      "markets": {
        "winner": { "home": 2.10, "draw": 3.50, "away": 3.40 },
        "goals":  { "over": 1.50, "under": 2.60 },
        "scorer": { "saka": 3.20, "havertz": 4.10, "salah": 3.60 }
      }
    }
  ]
}
```

Rules the front end already assumes:

| Rule | Behaviour required |
|---|---|
| `status` | `open`, `suspended` or `closed`. `suspended` is temporary and returns; `closed` does not come back before kick-off. |
| Missing market | A partner may omit a market it does not price. Treat as suspended for that market only, never for the whole partner. |
| Refresh | Push on change; a poll fallback no slower than 30s. The screen shows the age of the data, so a stale timestamp is worse than a missing update. |
| Kick-off | All markets for a match close at kick-off. The client already refuses to lock after kick-off; the feed must not keep serving open prices past it. |
| Partner outage | A partner that stops responding is `suspended`, not absent. A row that vanishes moves the layout under the user's finger. |
| Price movement | Any change to a selection the user is looking at must arrive as a new payload; the client compares it against the price it showed and asks the user to accept. |

### Failure modes

- **No feed at all:** serve the last known payload with its real `updatedAt`. The client
  keeps working and shows the age.
- **Partial feed:** partners present in the payload are live, the rest are suspended.
- **Never** substitute a stale price silently. The acceptance step exists precisely because
  the price the user agreed to is a commitment.

---

## 2. Notifications

The prototype asks for browser permission itself and schedules two local notices while the
tab is open: kick-off and full-time. A service owns the same two events for a closed app,
plus one it cannot know about locally.

| Event | Fired when | Body |
|---|---|---|
| `kickoff` | Kick-off of a match the user locked | "Bayern v Dortmund is under way. Your prediction is live." |
| `result_ready` | Settlement of a match the user locked | Points and the score against CaptainAI |
| `streak_at_risk` | 3h before the last open match of the week, streak > 0, nothing called | One per week, never more |

Rules:

- **Opt-in is the browser's.** The service stores the subscription only after
  `Notification.requestPermission()` has returned `granted`; `notify_opt_out` revokes it.
- **One notification per event per user per match.** Deduplicate server-side: a retry must
  not produce a second buzz.
- **No promotional pushes.** Offers, prizes and partner messages never go through this
  channel. That is a responsible-gambling line, not a preference.
- **Quiet hours** 23:00–08:00 local: hold `streak_at_risk`, deliver the other two, since
  they are outcomes the user asked for.
- Payload carries `matchId` so the click opens `#/match/{id}/result`.

---

## 3. Account and sync

### What a profile is today

The transfer link already carries the full object, so the sync payload is the same shape:

```js
['name', 'iq', 'streak', 'correct', 'total', 'aiCorrect', 'marketCorrect',
 'humans', 'ai', 'club', 'badges', 'rivals', 'week', 'questDone', 'freezes',
 'accountSaved', 'matches']
```

`matches` maps `matchId → { status, answers, powerPick, points, correct, lockedAt, settledAt, duel }`.

### The contract

- `GET /match-iq/me` returns the object above, or 404 for a new account.
- `PUT /match-iq/me` accepts it whole. The client writes on every state change; the service
  may debounce.
- **Conflicts:** last write wins per match entry, not per profile. Two devices playing
  different matches in the same week must both survive; the same match locked twice keeps
  the earlier `lockedAt`, because a lock is a commitment and the first one is the real one.
- **Anonymous merge:** the client holds `uid` from first run. On sign-in, the service merges
  the anonymous profile into the account: sum the counters, union the badges and rivals,
  keep the higher streak, and keep every match entry. Merge once, then drop the anon id.
- **Server owns settlement.** The prototype's "Simulate full-time" is a demo control. In
  production the service writes `status: settled`, `points` and `correct` from official
  results, and the client renders what it is told. Nothing about scoring is trusted from
  the browser.
- **Deletion:** deleting an account deletes the profile and the click attribution rows tied
  to it, and leaves the aggregate leaderboards intact.

---

## 4. Analytics

Every event below is already emitted by the prototype through `dataLayer.push` with the
same envelope: `{ event, props, ts, sessionId, userId, screen, week }`.

**Funnel (the Metrics panel counts these):** `challenge_start` → `challenge_complete` →
`prediction_locked` → `odds_viewed` → `handoff_opened` → `bookmaker_click` →
`result_viewed`.

**The rest:** `account_prompt_dismissed`, `account_saved`, `age_confirmed`,
`answer_selected`, `challenge_resume`, `club_selected`, `content_click`, `duel_received`,
`duel_sent`, `duel_settled`, `handoff_dismissed`, `leaderboard_expand`, `leaderboard_view`,
`match_page_view`, `nav_click`, `notify_opt_in`, `notify_opt_out`, `power_pick_set`,
`price_moved`, `profile_restore_declined`, `profile_restored`, `profile_transfer_link`,
`quest_complete`, `reveal_viewed`, `share_click`, `share_dismissed`,
`welcome_offer_claimed`, `welcome_offer_dismissed`, `welcome_offer_selected`,
`welcome_offer_shown`, `welcome_offer_to_table`.

Definitions must be frozen before launch. A metric whose definition drifts is worse than a
metric nobody collects.

---

## 5. Partner attribution

- Every outbound link carries `click_id = hash(user_id, match_id, partner_id, ts)`.
- The partner returns it on registration and first deposit postbacks.
- Attribution window 30 days, last click.
- Duplicate `bookmaker_click` to the same partner within 5 minutes counts once for handoff
  conversion, and always once for the North Star metric.
- The North Star (Weekly Betting-Intent Predictors) counts logged-in users only: without a
  stable identity, "unique this week" is a guess.
