# Model the problem

## Point

We should be modeling our problem using OOP and related paradigms to design software that fulfils application needs. Our implementations should adhere to the SOLID principles, and, we should try to build for reuse where applicable.

As per designs `GSII` and `public` have a need to retrieve many measures across academic years, characteristics - and to search.

While the exact measures presented is different the presentation **context** appears identical. Both systems treat measures similarly in intent and presentation context, with GSII layering additional behaviours such as ordering and aggregation.

`GSII` roadmap suggests the same pages for `Primary` and `Middle`. This will result in the same pages with different relevant measure-groupings for those Establishment types.

We can design/model this in an S**O**LID way - so that the software can be **extended** with extra measures, without duplicating services, repositories, application-models per-scope.

We should explicitly model a `Measure` and its possible value forms (scalar, estimated, redacted) as first‑class domain concept(s) within `Core`.

## Problems

- If we don't abstract `a measure` the different values a measure can have and the invariants for measure construction will bleed out into all consumers.
  - The concrete value of a measure can be different
    - In the simplest form it's a decimal; `32.592`
    - a more advanced form; Progress8 - an estimate value with a bounded confidence interval range e.g. `(-1.27 to 0.88) 0.21`
    - a redacted value e.g. `NA` / `SUPP` / `LOWCOV`

- Measures need to have comparison semantics
  - Whether a measure can be compared to another
    - should we be comparing `National:Attendance:Girls` with `National:Progress8:Girls`? Likely no.
    - should we be comparing a redacted value in the data like `NA` to a scalar value like `1.73`? Likely no.
  - Whether a measure value is greater, equivalent, or less than another

``` cs
interface IMeasure
{
    MeasureIdentifier Id { get; }
    MeasureValue Value { get; }
    bool CanCompareTo(IMeasure other);
}
```

These are representative examples of the domain rules that must be centralised when modelling measures.

## Examples

## Suggestions

See `GetMeasuresUseCase` which models application need of retrieving measures for most pages.

This could be centralised in some form to work for both `GSII` and `public`.

This avoids both teams needing to write UseCases and custom data-access abstractions.
