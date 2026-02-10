# @dgtools/disc-defender

Lightweight TypeScript client for the [Disc Defender API](https://discdefender.com/api). Fetches public lost-and-found account and disc data with in-memory filtering.

## Installation

```bash
npm install @dgtools/disc-defender
# or
yarn add @dgtools/disc-defender
```

Requires a runtime with `fetch` available (Node 18+, modern browsers).

## Quick start

```ts
import {
  findAccounts,
  findAccountDetails,
  getDiscs,
  filterDiscs,
} from '@dgtools/disc-defender'

// List all accounts
const accounts = await findAccounts()

// Find accounts by name
const matching = await findAccounts({ accountName: 'maple hill' })

// Get full account details (settings, stats, users, activity)
const [details] = await findAccountDetails('maple hill')
console.log(details.settings.businessName) // "Maple Hill Disc Golf"
console.log(details.stats.discsActive) // 26

// Fetch and filter discs for an account
const discs = await getDiscs({
  accountName: 'maple hill',
  brand: 'Innova',
  color: 'Pink',
})
```

## API

All functions are async and hit `https://discdefender.com/api`.

### `findAccounts(filters?)`

Fetches all accounts and filters them in-memory using exact (case-sensitive) string matching.

```ts
function findAccounts(filters?: AccountFilters): Promise<AccountResponse[]>
```

```ts
type AccountFilters = {
  accountName?: string | string[]
  id?: string | string[]
  createdAt?: string | string[]
}
```

```ts
// All accounts
const all = await findAccounts()

// Single filter
const accounts = await findAccounts({ accountName: 'maple hill' })

// Multiple values (OR — matches any)
const accounts = await findAccounts({
  accountName: ['maple hill', 'tri-fox disc golf'],
})
```

### `findAccountDetails(accountName)`

Looks up accounts by name and fetches full details for each match in parallel. Throws if no account is found.

```ts
function findAccountDetails(accountName: string): Promise<AccountDetailsResponse[]>
```

```ts
const [details] = await findAccountDetails('maple hill')

details.settings.businessName // "Maple Hill Disc Golf"
details.stats.discsActive     // 26
details.stats.discsReturned   // 100
details.isActive              // true
```

### `getDiscs(options)`

Fetches discs for the first matching account and applies **case-insensitive** filters in-memory.

```ts
function getDiscs(
  options: { accountName: string } & DiscFilters
): Promise<DiscResponse[]>
```

```ts
type DiscFilters = {
  brand?: string | string[]
  model?: string | string[]
  color?: string | string[]
  plastic?: string | string[]
  pdga?: string | string[]
  firstName?: string | string[]
  lastName?: string | string[]
}
```

```ts
// All discs for an account
const discs = await getDiscs({ accountName: 'maple hill' })

// Single filter
const innova = await getDiscs({ accountName: 'maple hill', brand: 'Innova' })

// Multiple filters (AND — all must match)
const pinkInnova = await getDiscs({
  accountName: 'maple hill',
  brand: 'Innova',
  color: 'Pink',
})

// Multiple values for one field (OR — any can match)
const multiColor = await getDiscs({
  accountName: 'maple hill',
  color: ['Pink', 'Blue', 'Red'],
})
```

### `filterDiscs(discs, filters?)`

Filters an existing array of discs without re-fetching. Uses the same case-insensitive matching as `getDiscs`.

```ts
function filterDiscs(discs: DiscResponse[], filters?: DiscFilters): DiscResponse[]
```

```ts
const allDiscs = await getDiscs({ accountName: 'maple hill' })

// Re-filter locally
const innova = filterDiscs(allDiscs, { brand: 'Innova' })

// Combine with your own filters
const activeInnova = filterDiscs(
  allDiscs.filter((d) => d.status === 'active'),
  { brand: 'Innova' },
)
```

## Filter behavior

| Aspect | Behavior |
| --- | --- |
| Single value | `brand: 'Innova'` — must match that value |
| Array of values | `color: ['Pink', 'Blue']` — matches if **any** value matches (OR) |
| Multiple fields | `{ brand: 'Innova', color: 'Pink' }` — **all** fields must match (AND) |
| Omitted filter | Field is not checked (matches everything) |
| `null` field value | Never matches a filter (disc filters only) |
| Case sensitivity | Disc filters are **case-insensitive**; account filters are **case-sensitive** |

## Types

Type definitions ship with the package.

### `AccountResponse`

```ts
type AccountResponse = {
  id: string
  accountName: string
  createdAt: string
  accountPath: `/${string}`
}
```

### `AccountDetailsResponse`

```ts
type AccountDetailsResponse = {
  primaryEmail: string
  id: number
  accountName: string
  createdAt: string
  lastActive: string
  since: { days: number; hours: number; minutes: number; seconds: number; milliseconds: number }
  accountPath: `/${string}`
  settings: {
    accountPath: `/${string}`
    timezone: string
    smsTemplate: string
    businessName: string
    phone: string
    state: string
    zipCode: string
    city: string
    address: string
    title: string
    accountName: string
    metaDescription: string
    gracePeriodInDays: string
    customInstructions: string
  }
  stats: {
    discsCount: number
    discsActive: number
    discsDeleted: number
    discsReturned: number
    discsSold: number
  }
  users: Array<{
    id: string
    email: string
    verified: boolean
    lastActive: string | null
    sessionStart: string | null
  }>
  accountActivity: {
    discsCreatedInLastMonth: number
    discsCreatedInLastTwoWeeks: number
  }
  isActive: boolean
}
```

### `DiscResponse`

```ts
type DiscResponse = {
  id: string
  firstName: string
  lastName: string
  phone: string
  pdga: string
  description: string | null
  createdAt: string
  accountId: string
  status: 'active' | 'returned' | 'sold' | 'deleted'
  location: string | null
  notes: string | null
  updatedAt: string
  brand: string | null
  model: string | null
  color: string | null
  plastic: string | null
  lastNotified: string | null
  binLocation: string | null
  smsStatus: 'pending' | 'delivered' | 'failed' | 'no-phone'
  contactStatus: string | null
  contactStatusNotes: string | null
  allowReminder: boolean
  isPastGracePeriod: boolean
  daysLeftToPickup: number
  days: number
  searchField: string
}
```

## Error handling

- Network failures or non-2xx responses throw an `Error`.
- `findAccountDetails` and `getDiscs` throw if no account matches the provided name:

  ```txt
  Error: No account found with name: <accountName>
  ```

## Development

- Build: `yarn workspace @dgtools/disc-defender build`
- Tests: `yarn workspace @dgtools/disc-defender test`
