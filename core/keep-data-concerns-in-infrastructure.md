# Keep data concerns in Infrastructure

## Point

Data concerns should not bleed into the `Core/Application` `Core/Domain`. By bleeding data concerns into the application model - voids benefits of Clean Architecture and dependency inversion.

This is briefly highlighted in [`GSII coding standards`](https://github.com/DFE-Digital/sap-sector/blob/main/docs/developers/coding-standards.md#models)

## Problems

- Creates N‑tier‑like coupling where application and presentation layers depend on persistence‑shaped models, increasing sensitivity to database and query changes.

## Examples

These entities are shaped to match storage/query output (scope‑specific, measure‑specific fields), and changes to persistence require changes to Core types.

### GSII

- [EnglandPerformance](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Core/Model/Generated/EnglandPerformance.cs#L8-L9)
- [EstablishmentDestinations](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Core/Model/Generated/EstablishmentDestinations.cs#L9)
- [LAAbsence](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Core/Model/Generated/LAAbsence.cs#L9)
- [Organisation](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Core/Model/Organisation.cs#L5)

### Public

- [EnglandDestinations](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Destinations/EnglandDestinations.cs#L22)
- [EstablishmentPerformance](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Performance/EstablishmentPerformance.cs#L15-L16)
- [LAAbsence](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Absence/LAAbsence.cs#L18)

## Suggestions

See any of the [Features](/examples/src/DfE.SchoolProfiles.Sector.Core/Features/) where `Response DTOs` from Core are owned by the application layer and are not defined by database schemas or repository shape.

Infrastructure DTOs (mutable, persistence‑shaped) are embedded ONLY in Infrastructure and consumed in a Repository **implementation** to construct a `Core` model. `IMapper<TIn, TOut>` maps a `DataTransferObject` to a Core model.

This means `Core` is not aware of which repository hydrated the model, and the Repository is free to change, provided the `Application` contract is met. It can be tested independently of a Repository, and, the repository implementation is free to change.
