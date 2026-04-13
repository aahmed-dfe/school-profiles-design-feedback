# Use ValueObjects

## Point

Clean Architecture suggests;

- modelling the `Core` as objects, agnostic to `Infrastructure`.
- Repositories should return fully constructed models.

This ensures our `Core` models are agnostic to any infrastructure implementation.

`ValueObjects` should be used when modelling objects that do not have an identity (Entities) and should be used in composition of other Models.

`ValueObjects` enable us to validate the invariants as part of construction and centralise any parsing (e.g. null guarding, whitespace trimming, validating shape of inputs)

`ValueObjects` should be introduced where a value has meaning and invariants; primitives remain appropriate for purely technical or transient data.

## Problems

- Primitive obsession on `Core` models - leading to model property and method bloat with `string`, `int` and other primitives.

- Lack of type-safety around domain constructs

- This contributes to `Core` surfacing primitive, persistence‑shaped data on its DTOs, increasing the likelihood that presentation layers become coupled to storage concepts.

## Examples

These models expose large numbers of primitives and place parsing and validation responsibility on consumers rather than centralising it within `ValueObjects`.

### GSII

- [Establishment](https://github.com/DFE-Digital/sap-sector/blob/main/SAPSec.Core/Model/Generated/Establishment.cs#L9) used in [IEstablishmentRepository](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Interfaces/Repositories/IEstablishmentRepository.cs#L7-L8)
- [EstablishmentPerformance](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Model/Generated/EstablishmentPerformance.cs#L17)
- [IAttendanceRepository](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Attendance/IAttendanceRepository.cs#L3-L4) which surfaces [EstablishmentAttendance](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Attendance/IAttendanceRepository.cs#L13), [EnglandAttendance](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Attendance/IAttendanceRepository.cs#L25) and [LocalAuthorityAttendance](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Attendance/IAttendanceRepository.cs#L35-L36)

### Public

As public surfaces lots of data models with no `Core` models, this appears throughout.

- [Establishment](https://github.com/DFE-Digital/sap-public/blob/main/SAPPub.Core/Entities/Establishment.cs#L11) surfaced in [EstablishmentService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/IEstablishmentService.cs#L7)
- [EnglandPerformance](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Entities/KS4/Performance/EnglandPerformance.cs#L7-L8) surfaced in [IEnglandPerformanceService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/KS4/Performance/IEnglandPerformanceService.cs#L7)
- [AboutSchoolModel](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/ServiceModels/KS4/AboutSchool/AboutSchoolModel.cs#L3) surfaced in [IAboutSchoolsService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/KS4/AboutSchool/IAboutSchoolService.cs#L7-L8)
- [EstablishmentWorkforce](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Entities/KS4/Workforce/EstablishmentWorkforce.cs#L7-L8) [IEstablishmentWorkforceService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/KS4/Workforce/IEstablishmentWorkforceService.cs#L7)

## Suggestions

- Create `ValueObjects` and construct Application models with them
  - (using `ValueObject<T>`
  - CSharpFunctionalExtensions lib which has an optimised `ValueObject<T>`
  - Source generate them; VoGen)

```cs
// See https://www.get-information-schools.service.gov.uk/Guidance/LaNameCodes
public readonly struct LocalAuthorityCode
{
    public LocalAuthorityCode(int code)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(code);
        ArgumentOutOfRangeException.ThrowIfGreaterThan(code, 999);
        Code = code;
    }

    public LocalAuthorityCode(string code) : this(Parse(code))
    {
    }


    private static int Parse(string code)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(code);

        if (!int.TryParse(code.Trim(), NumberStyles.None, CultureInfo.InvariantCulture, out int parsed))
        {
            throw new ArgumentException("Local authority code must be numeric.", nameof(code));
        }

        return parsed;
    }

    public int Code { get; }
}


// This can then be passed into the construction of other ReadOnly contracts e.g. 

public Establishment : IReadOnlyEntity<UniqueReferenceNumber>
{
  public Establishment(UniqueReferenceNumber urn, LocalAuthorityCode laCode) : base(urn)
  {
    LocalAuthority = laCode;
  }

  public LocalAuthorityCode LocalAuthority { get; }
}
```