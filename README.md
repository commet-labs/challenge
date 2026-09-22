# Commet Challenge

Commet is the platform where a company configures how it charges its customers. This exercise is an interface inside the Commet dashboard.

Nimbus is a customer of Commet: it generates images, copy and video with AI, and charges for it in credits. `data/catalog.ts` holds their entire pricing — 12 features, 5 plans, 9 plan versions, 105 feature configurations, and the metrics for the current period.

The people using it are the Nimbus team configuring their own product. Not Nimbus end customers.

## Objective

Someone at Nimbus opens the interface, understands how they are charging their customers, and understands what happens if they change it.

Questions they need to answer:

- What separates one plan from the next
- Who is affected by changing the price of a feature
- What changed between one plan version and the previous one, and who stayed on which
- Where consumption is happening, and where extra billing is piling up

## Data model

Nimbus charges in credits: every plan includes an amount per period, and every action in the product consumes a different amount.

| Entity | Description |
| --- | --- |
| `organization` | The company doing the charging |
| `feature` | Everything the product offers. Defined once, in a catalog |
| `plan` | What a customer subscribes to. `isPublic` marks whether it is offered openly or sold privately |
| `PlanPrice` | A plan billed monthly or yearly, each with its own included credits |
| `PlanRelease` | A version of a plan. Changing a plan publishes a new version |
| `ReleaseFeature` | A catalog feature as configured inside one version of one plan |
| `creditPack` | Loose credits a customer buys when they run short, without changing plan |
| `metrics` | What happened during the period. Not configuration |

| Field | Values |
| --- | --- |
| `feature.type: "credit"` | Spends credits. `creditsPerUnit` sets how many per unit: an image costs 5, an API call 1, a video 30 |
| `feature.type: "capacity"` | Does not spend credits. Includes an amount and, past it, either bills per unit (`overage: billed`) or cuts off (`overage: blocked`). Seats, storage, workspaces |
| `feature.type: "boolean"` | On or off. SSO, audit log, priority support |
| `pricing.type` | `free`, or `standard` with monthly and yearly prices |
| `exhaustionPolicy` | What happens once a customer spends every credit: `block` cuts off the service, `bill_overage` keeps it running and bills the excess at `pricePer1000Credits` |
| `release.status` | `published` is what new customers get, `retired` is an old version that still has customers on it, `building` is a draft |

Amounts are in cents: `9900` is $99.00.

### How the entities relate

The credit chain: the `PlanPrice` sets a budget (`includedCredits`), consumption features spend it (`creditsPerUnit` × usage), the `exhaustionPolicy` decides what happens when it runs out, and a `creditPack` buys more without changing plan.

- A plan has several versions. `currentReleaseVersion` is the published one, but customers stay on the version they subscribed to. A plan can have customers spread across versions, and the published one is not always where most of them are.
- A `ReleaseFeature` configures, by `code`, a feature that already exists in the catalog. The same feature is configured differently in every version of every plan.
- Capacity features are the only ones that bill outside the plan price.
- `metrics` joins the configuration on `planCode` + `version`, and on `featureCode` where it applies. Plans describe how you charge; `metrics` describes what happened.

## Scope

The interface can show, create or edit. One page or several, laid out however presents the information best. Nothing has to persist — faked editing is fine. What matters is that the experience makes clear what changing a field means before it is changed, and how the information is ordered.

The data model is not fixed. Types are in `lib/catalog.ts` and the data is a `satisfies`, so the compiler reports any change that leaves it inconsistent.

You can use AI agents. The code you hand in is yours and we will read it.

## Delivering

Create a private repository from this template with "Use this template", work there, and send us the link.

## Running

```bash
pnpm install
pnpm dev
```

The starting page prints the raw data as stored.
