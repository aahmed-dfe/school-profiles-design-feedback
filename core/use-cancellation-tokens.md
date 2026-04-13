# Use cancellation tokens

## Point

C# provides `CancellationToken` for use in cancelling asynchronous operations. Libraries can be passed this for cancelling operations e.g. a query operation can be cancelled because a consumer cancels the HTTP request, enabling the resources consumed in the asynchronous scope to stop e.g. connections returned to the pool / disposed.

Cancellation is a cross‑cutting concern that must be propagated from presentation through application orchestration to infrastructure.

## Issues

Without providing a `CancellationToken` asynchronous operations cannot be cancelled once executed. These are provided by the `aspnetcore` request scope onto controller actions.

## Examples

### GSII

- [GetSchoolKs4HeadlineMeasures](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Ks4HeadlineMeasures/UseCases/GetSchoolKs4HeadlineMeasures.cs#L15-L16) no ctx on UseCase
- [IAttendanceRepository](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Attendance/IAttendanceRepository.cs#L3) no ctx on repository
- [IKs4PerformanceRepository](https://github.com/DFE-Digital/sap-sector/blob/c874df1b8010e9bd76d5590590871bbdd8ae455e/SAPSec.Core/Features/Ks4HeadlineMeasures/IKs4PerformanceRepository.cs#L5-L6) no ctx on repository

### Public

Public has cancellation tokens on most of it's `Services` and `Repositories` but there are some;

- [GatewayLocalAuthorityService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/Gateway/IGatewayLocalAuthorityService.cs#L10)
- [GatewayUserService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/Gateway/IGatewayUserService.cs#L10-L11)
- [IEmailRepository](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Repositories/IEmailRepository.cs#L11)
- [IPostCodeLookupService](https://github.com/DFE-Digital/sap-public/blob/dbc5bf806f612baa5dc5b797ac8c6be3a4afc291/SAPPub.Core/Interfaces/Services/IPostcodeLookupService.cs#L7-L8)

## Suggestions

- add a `CancellationToken` on `IUseCase` contracts and related Service contracts for cancelling async operations, and pass that down to repositories and other infrastructural concerns.

This forces implementations to implement.

```cs

interface IUseCase<TIn, TOut>
{
  Task<TOut> HandleAsync(TIn input, CancellationToken ctx);
}

internal sealed class MyUseCase : IUseCase<Input, Output>
{
    private readonly IReadOnlyRepository _repository;
    
    public async Task<Output> HandleAsync(Input input, CancellationToken ctx)
    {
        await _repository.Work(input, ctx);

    }
}
```
