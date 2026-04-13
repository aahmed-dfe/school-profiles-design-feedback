# Keep core DTOs immutable

## Point

UseCases often return `Data Transfer Objects` (DTOs) that consumers of Core depend on.

Immutable DTOs act as stable contracts between the application core and its consumers, preventing accidental behavioural coupling

If they are not immutable, consumers can introduce bugs (and embed domain behaviour) into their presentation-layer(e.g. UI) by editing properties on models.

## Examples

### GSII

- [Establishment](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Model/Generated/Establishment.cs#L9)
- [EstablishmentAbsence](https://github.com/DFE-Digital/sap-sector/blob/9bf6df56225c27cb9e35d4f14e52547fdba2dfae/SAPSec.Core/Model/Generated/EstablishmentAbsence.cs#L17-L18)
- [LaPerformance](https://github.com/DFE-Digital/sap-sector/blob/9bf6df56225c27cb9e35d4f14e52547fdba2dfae/SAPSec.Core/Model/Generated/LAPerformance.cs#L16-L17)

### Public

Public DTOs in `Core` are shaped primarily by persistence and query concerns rather than protected application contracts. The DTOs are still muteable with public `setters`

- [EnglandDestinations](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Destinations/EnglandDestinations.cs#L22)
- [EstablishmentPerformance](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Performance/EstablishmentPerformance.cs#L15-L16)
- [LAAbsence](https://github.com/DFE-Digital/sap-public/blob/0f23db332b135b4f5e51fbbae707453c322fcc04/SAPPub.Core/Entities/KS4/Absence/LAAbsence.cs#L18)

## Suggestions

- Remove public setters from DTOs and replace them with construction-time initialisation (init, constructors, or records).

- create projections from domain models into DTOs and return them from usecases.

    ```cs
    private readonly IEstablishmentRepository _repository;
        
    private readonly IMapper<Establishment, GetEstablishmentResponseDataTransferObject> _mapper;
    
    public async Task UseCase(string urn)
    {   
        Establishment establishment = await _repository.GetAsyncById(urn);

        GetEstablishmentResponseDataTransferObject response = _mapper.Map(establishment);

        return response;
    }
    ```

Then either

1) Use `records` with value semantics, or, readonly objects

    ```cs

    // records
    public record GetEstablishmentResponseDataTransferObject
    {
        public required EstablishmentDto Establishment { get; init; } 
        public required IDictionary<string, string[]> Facets { get; init; }
    }

    //  read-only objects.

    ```cs
    public sealed class GetEstablishmentResponseDataTransferObject
    {

    public GetEstablishmentResponseDataTransferObject(...args)
    {
        // Guard args if necessary - set props
    }
    public EstablishmentDto Establishment { get; } 
    public IDictionary<string, string[]> Facets { get; }
    }

    ```
