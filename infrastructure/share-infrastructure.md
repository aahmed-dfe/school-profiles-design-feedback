# Share Infrastructure

## Point

We should be designing to contracts (e.g. `interfaces`) promoting loose coupling so that we can change implementations without impacting consumers.

We should share these contracts and implementations so that we do not duplicate effort.

The goal is not to share repositories or generic data access abstractions. It is to share infrastructure execution

## Problems

- Duplication (Unit testing, Integration testing, Bugs in differing implementations, effort in maintenance)

- Infrastructure cannot be changed simply as coupling to data-access in application model (e.g. changing SearchIndex / Database table)

- CrossCutting behaviour is harder to add in a single place because interfaces cannot be `decorated` or changed e.g. performance logging or caching

## Examples

### Search

Both applications duplicate Searching - via a `Lucene` implementation instead of creating a contract and backing the implementation

- [GSII](https://github.com/DFE-Digital/sap-sector/tree/main/SAPSec.Infrastructure/LuceneSearch)
- [Public](https://github.com/DFE-Digital/sap-public/tree/main/SAPPub.Infrastructure/LuceneSearch)

### Search - suggestion

- Create a Search abstraction for keyword search e.g. `ISearchServiceAdaptor`. Create a `Search.Contracts` semantically versioned NuGet package
- Create implementation for `LuceneSearchServiceAdaptor` -> `Search.Abstractions`
- Create a composition root that allows registration of e.g. `AddLuceneSearch(this IServiceCollection services)`
- Consume `ISearchServiceAdaptor` in `Sector.Infrastructure` and `Public.Infrastructure`
- Creating another implementation for PostGres text search can be worked on, by backing onto an abstraction - swapping in the future is easier.

See library built for [Azure AISearch example](https://github.com/DFE-Digital/infrastructure-cognitive-search).

### DataAccess

Both applications define their own data access abstractions.

`Public` uses a [IGenericRepository](https://github.com/DFE-Digital/sap-public/blob/main/SAPPub.Core/Interfaces/Repositories/Generic/IGenericRepository.cs) with `Dapper` and [`NpgqlDataSource`](https://github.com/DFE-Digital/sap-public/blob/2ce6a3451dea68163ff6dda492d975228f38de2f/SAPPub.Infrastructure/Repositories/Generic/DapperRepository.cs#L13) reflecting the requested type via a [Helper to database fields](https://github.com/DFE-Digital/sap-public/blob/2ce6a3451dea68163ff6dda492d975228f38de2f/SAPPub.Infrastructure/Repositories/Helpers/DapperHelpers.cs#L360) - this is wrapped across many other `Repositories` and only used in `Core` [here](https://github.com/DFE-Digital/sap-public/blob/2ce6a3451dea68163ff6dda492d975228f38de2f/SAPPub.Core/Services/Gateway/GatewaySettingsService.cs#L14-L15).

`GIIS` uses `INpgqlDataSource` and Dapper, creates inline SQL queries to retrieve data see [`IAttendanceRepository`](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Infrastructure/Postgres/PostgresAttendanceRepository.cs#L10-L11) and [`IEstablishmentRepository`](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Infrastructure/Postgres/PostgresEstablishmentRepository.cs#L8)

### Data access suggestions

`DapperRepository<T>` is a good example of having multiple reponsibilities and every repository consuming is locked into 1 SQL style, 1 error-handling strategy, one mapping strategy.

1) Generic lookups on `<T>` on Type hides coupling
2) SQL selection logic see `GetReadSingle(typeof(T))` or `GetReadMultiple(typeof(T)`
3) Domain rules `_codedValueMapper.Apply(items);`
4) Generic error handling policies (catch / throw)

We should share infrastructure mechanics and cross‑cutting behaviour, but not data‑shaping or domain‑aware abstractions. Based on both repositories this could include

- Connection creation & pooling (`NpgsqlDataSource`)
- Cancellation token propagation
- Logging conventions
- Retry / transient error policies
- Consistent error handling strategies

This is duplicated today. This should not aim to hide SQL or replace repositories.

Abstracting data-access `ISqlQueryHandler`

Follow the same principles as above create abstraction
Create `Dapper` implementation over abstraction.

``` cs

// This abstraction does not infer schema, generate SQL, or apply domain rules.

public interface ISqlQueryHandler
{
    Task<IReadOnlyList<T>> QueryAsync<T>(
        string sql,
        object? parameters,
        CancellationToken cancellationToken);

    Task<int> ExecuteAsync(
        string sql,
        object? parameters,
        CancellationToken cancellationToken);
}

public sealed class PostgresEstablishmentRepository : IEstablishmentRepository
{
    private readonly ISqlQueryHandler _handler;

    public PostgresEstablishmentRepository(ISqlQueryHandler handler)
    {
        _handler = handler;
    }

    public Task<IReadOnlyList<Establishment>> GetAllAsync(CancellationToken ct)
    {
        const string sql = """
            SELECT "URN", "EstablishmentName", "Street", "Postcode"
            FROM public.v_establishment
        """;

        return _handler.QueryAsync<Establishment>(sql, null, ct);
    }
}

```

See library built for [CosmosDB](https://github.com/DFE-Digital/infrastructure-persistence-cosmosdb) example
