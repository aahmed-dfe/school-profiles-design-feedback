# Share behaviour

## Point

There is shared behaviour in the SchoolProfiles programme (public and sector) in what the applications do. This should be centralised where overlap and consumed in both.

## Problems

Duplication of behaviour and modelling decisions increases maintenance cost, inconsistency, and divergence between public and sector implementations.

## Examples

`GSII`

- Search
  - For a school by name or UniqueReferenceNumber
  - For similar schools to a school. Filterable and sortable
- Measure
  - Get a measure(s) for schools for recent academic year(s) - filter for given cohorts (Girls, Boys) and scopes (National, LA). These measures should be comparable and sometimes, aggregated from prior years. There is sometimes a need `NumberOfPupils` to retrieve `RelatedMeasures` to a measure.

`sip-public`

- Search
  - For a school by name. Filterable
  - For a school by postcode
  - For a school by council
- Measure
  - Get a measure(s) for a school for a given academic year

## Suggestions

Expose capability that can be shared to avoid duplication. There are many ways to achieve this, depending on the desired physical/logical boundaries.

1) An SDK‑style, semantically versioned NuGet package exposing UseCase contracts and shared domain abstractions. This allows behaviour reuse without forcing a physical service boundary.

For example, both applications could depend on a shared `GetMeasuresUseCase` contract, while each application provides its own infrastructure implementations and presentation‑specific composition.
