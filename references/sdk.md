# SDKs

Both clients wrap the same REST API: one method per endpoint, cursor pagination
handled for you, errors raised with the API's `code`, `message` and request id.

## Python

```bash
pip install ufcalendar
```

```python
from ufcalendar import FightAPI

api = FightAPI("ufcalendar_...")          # or export UFCAL_API_KEY=...

for ev in api.events(org="ufc", limit=5):        # upcoming, soonest first
    print(ev["starts_at"], ev["title"])

card = api.event("ufc-320")                       # full card + venue + broadcasts
rounds = api.fight_rounds(card["card"][0]["id"])  # round-by-round stats
cards = api.fight_scorecards(card["card"][0]["id"])  # the judges' cards
odds = api.fight_odds(card["card"][0]["id"])     # consensus line: current, opening, closing
board = api.rankings("ufc", date="2016-11-14")    # rank 0 = champion
history = api.fighter_history("islam-makhachev")  # multi-promotion career

plans = api.plans()                               # works without a key
```

Errors raise `FightAPIError` (`.status`, `.code`, `.message`, `.request_id`).
After every call `api.last_rate_limit` holds the `X-RateLimit-*` headers.

<https://pypi.org/project/ufcalendar/> · <https://github.com/UFCalendar/ufcalendar-python>

## TypeScript

```bash
npm install @ufcalendar/sdk
```

```ts
import { FightAPI } from '@ufcalendar/sdk';

const api = new FightAPI(process.env.UFCALENDAR_API_KEY);

for await (const ev of api.events({ org: 'ufc', limit: 5 })) {
  console.log(ev.starts_at, ev.title);
}

const card = await api.event('ufc-320');
const rounds = await api.fightRounds(card.card[0]!.id);
const cards = await api.fightScorecards(card.card[0]!.id);
const odds = await api.fightOdds(card.card[0]!.id);  // consensus line: current, opening, closing
const board = await api.rankings('ufc', { date: '2016-11-14' });
const history = await api.fighterHistory('islam-makhachev');

const plans = await api.plans();                  // works without a key
```

List methods are async generators; everything else returns the envelope's
`data`. Errors throw `FightAPIError` (`.status`, `.code`, `.message`,
`.requestId`); `api.lastMeta` and `api.lastRateLimit` hold the last response's
metadata. ESM + CJS, Node 18+, no runtime dependencies.

<https://www.npmjs.com/package/@ufcalendar/sdk> · <https://github.com/UFCalendar/ufcalendar-typescript>

## Neither language?

Generate a client from the OpenAPI 3.1 document at
<https://api.ufcalendar.com/openapi.json>, or call the REST endpoints directly —
see `endpoints.md` and `conventions.md`.
