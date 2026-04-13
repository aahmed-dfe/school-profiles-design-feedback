# Share Presentation Components

This refers to sharing presentation components and composition patterns, not entire pages or application layouts.

## Point

Both applications use MVC and Razor `(.cshtml)` to render views. Both applications use standardized GDS components, as well as similar charts and other components.

## Problems

- Large amounts of bespoke View/ViewModels being created when they could be created from a library of shared View/ViewComponents e.g. `GDSDetailsComponent`, `GDSPrimaryButtonComponent`.

## Examples

### GSII

- Duplication for each table, graph inside of the [School/Attendance.cshtml](https://github.com/DFE-Digital/sap-sector/blob/main/SAPSec.Web/Views/School/Attendance.cshtml) which is also rendered on [`Ks4HeadlineMeasures`](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Web/Views/School/Ks4HeadlineMeasures.cshtml#L97) as well as [here](https://github.com/DFE-Digital/sap-sector/blob/6eb40f234cdcb53b59079c01f9342e8740eb5aaa/SAPSec.Web/Views/School/Ks4HeadlineMeasures.cshtml#L158) than rendered via a reusable `TabbedGraphViewComponent`

## Suggestions

1) Create a shared ViewComponents versioned NuGet package e.g. `DfE.SchoolProfiles.Shared.Presentation`. Move shared ViewModels, ViewComponents, TagHelpers, and supporting partials into a shared package.

This could become a broader DfE package for other `dotnet` applications. It could consume the `GovUK.AspNetCore` library to generate these views, or use raw html, and construct composite Views used across DfE applications.

Partials within the applications can still exist - when specific to that applications context. e.g. `GSII` has a similar Primary/Middle View to Secondary schools, just with different data.
