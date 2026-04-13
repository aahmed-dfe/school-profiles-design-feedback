# Share behaviour

## Point

There is shared behaviour in the SchoolProfiles programme (public and sector) in what the applications do. This should be centralised where overlap and consumed in both.

## Problems

Duplication of behaviour and modelling decisions increases maintenance cost, inconsistency, and divergence between public and sector implementations.

## Examples

`GSII`

- Search
  - For a school by name or UniqueReferenceNumber(URN)
  - For similar schools to a school. filterable and sortable
- Measure
  - Get a measure(s) for schools for recent academic year(s) - filter the requested measure for given cohort (Girls, Boys) and a given scope(s) (National, LA). These measures should be comparable. Sometimes they are `DerivedMeasures` which are aggregated or calculations applied from across academic years. There is sometimes a need e.g. `NumberOfPupils` to retrieve `RelatedMeasures` to a `Measure`

`sip-public`

- Search
  - For a school by name. filterable
  - For a school by postcode
  - For a school by council
- Measure
  - Get a measure(s) for a school for a given academic year

## Suggestions

Expose capability that can be shared to avoid duplication. There are many ways to achieve this, depending on the desired physical/logical boundaries.

1) An SDK‑style, semantically versioned NuGet package exposing UseCase contracts and shared domain abstractions. This allows behaviour reuse without forcing a physical service boundary.

For example, both applications could depend on a shared `GetMeasuresUseCase` contract, while each application provides its own infrastructure implementations and presentation‑specific composition.
