---
title: "Package Deployer for solutions, data and migrations"
date: 2026-04-20
permalink: /package-deployer-solutions-data-migrations/
author: Tomas Prokop
categories:
  - Microsoft
  - Microsoft Power Platform
tags:
  - Power Platform
  - ALM
  - Package Deployer
  - DevOps
  - Dataverse
excerpt: "When pac solution import isn't enough. Package Deployer gives you multi-solution orchestration, custom code hooks at every stage, data migrations, and runtime parameters. Part 3 of a 3-part series."
---

> This is Part 3 of a 3-part series on Dataverse solution deployments:
> 1. [Dataverse solution component types](/dataverse-solution-component-types/)
> 2. [Making Dataverse solution imports fast](/making-solution-imports-fast/)
> 3. **Package Deployer for solutions, data, and migrations** (you are here)

Package Deployer is the most flexible deployment option for Dataverse. If you are deploying a single solution with no special requirements, PAC CLI is simpler and does the job. Package Deployer is for when you outgrow that.

## When and why Package Deployer

- **Configuration data between environments** - The most convenient way to move reference data, lookup values, and environment-specific settings alongside your solutions. CMT (Configuration Migration Tool) data packages are imported as part of the same deployment unit, so schema and data stay in sync.
- **Conditional automation via pipeline parameters** - Pass `RuntimeSettings` from the CLI without rebuilding the package. Your custom code can branch on these values: skip a data load, toggle a feature, set up connection, enable flows - whatever the target environment needs.
- **Data migrations with stage-for-upgrade** - When you need to move data from one column to another (e.g., different data type), combine Package Deployer with the [Stage for Upgrade + Apply Upgrade](/making-solution-imports-fast/#two-step-upgrades-and-when-they-are-still-useful) pattern from Part 2. Stage the new solution version so both old and new columns are present, run your migration code in a Package Deployer hook, then apply the upgrade to remove the old column. This gives you a safe window to transform data in-place.
- **Multi-solution deterministic ordering** - You control which solutions import first, second, third.
- **Custom code execution at every stage** - Seven hooks in the lifecycle, from initialization to post-deployment.
- **Explicit control over import type per solution** - Force a specific import action, skip solutions conditionally, override flags per solution.

That said, Package Deployer does not make solution architecture concerns disappear. Keep the number of solutions as low as practical and avoid overlapping unmanaged ownership and cross-solution dependencies where you can. PD is excellent at orchestration, not a substitute for clean layering.

## Default parameter values

Package Deployer's defaults differ from PAC CLI in one important way:

| Parameter | Package Deployer default | PAC CLI default |
|---|---|---|
| `OverwriteUnmanagedCustomizations` | **`false`** | `false` |
| `PublishWorkflows` (activate plugins/workflows) | **`true`** | **`false`** (requires `--activate-plugins`) |

The `PublishWorkflows` difference matters if your solution contains cloud flows or SDK message steps. Both tools default to `false` for `OverwriteUnmanagedCustomizations`, so [SmartDiff](/making-solution-imports-fast/#smartdiff) is enabled by default in both. The `PreSolutionImport` hook lets a package override both flags per-solution at runtime.

## Version-skip logic

Package Deployer applies this decision matrix before every import:

| Condition | Action | Import runs? |
|---|---|---|
| Incoming version == installed | `SkipSameVersion` | **No** |
| Incoming version < installed | `SkipLowerVersion` | **No** |
| Incoming version > installed | `Import` | Yes |
| Package min CRM version > org version | `SkipOrganizationVersionInCompatible` | **No** |

Both `SkipSameVersion` and `SkipLowerVersion` are silent. They produce an info log entry and the deployment continues as successful. An operator cannot distinguish "was imported" from "was skipped" without reading Package Deployer logs. The `OverrideSolutionImportDecision` hook lets package code force-reimport the same version or skip a version that would otherwise be imported.

## Automatic one-step vs two-step path selection

Package Deployer 4.0+ automatically decides between one-step and two-step upgrade at runtime. It inspects your package to see whether `RunSolutionUpgradeMigrationStep` contains real migration code. An empty override is treated as "no migration code" and the one-step path is chosen (provided the org supports single-step upgrades, version 9.2.21013.00131+). You can confirm the decision from PD's log line `User Provided Upgrade code is not detected in package.`. Under the hood, PD sets `USESTAGEANDUPGRADEMODE=true` on the import request so `ServiceClient` issues a `StageAndUpgradeRequest` instead of a classic `ImportSolutionRequest`.

The two-step holding path is chosen when:

- Custom upgrade code is detected in the package, or
- `forceupgradecodestep="true"` is set on a solution in the import config XML, or
- One-step upgrade is explicitly disabled in the app configuration

When the two-step path runs, `RunSolutionUpgradeMigrationStep` is called **after** the holding import completes but **before** `DeleteAndPromote` fires. This is the migration window.

Package Deployer also does not change the underlying Dataverse rules covered in [Part 1](/dataverse-solution-component-types/). Dependencies still have to resolve. Publisher ownership can still determine if you can perform operations. [Managed properties](/dataverse-solution-component-types/#managed-properties) decide how much a managed component can be customized downstream.

## Package lifecycle and custom code hooks

A PD package contains an [`ImportConfig.xml`](https://learn.microsoft.com/en-us/power-platform/alm/package-deployer-tool?tabs=cli) manifest (lists solutions to import, CMT data files, and import settings) and a .NET assembly with a C# class that provides overridable hooks. Running [`pac package init`](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/package#pac-package-init) scaffolds the project from a `dotnet new` template (shortName `pp-pdpackage`).

Package Deployer exposes seven overridable methods in a fixed execution order. Your package class inherits from the SDK's `ImportExtension` base class (which implements [`IImportExtensions2`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.tooling.packagedeployment.crmpackageextentionbase.iimportextensions2) through `IImportExtensions8`). The scaffolded class called `PackageImportExtension` extends `ImportExtension` and is discovered at runtime via MEF (`[Export(typeof(IImportExtensions))]`). You override the methods you need.

### 1. InitializeCustomExtension

```csharp
public override void InitializeCustomExtension()
```

Called first, before any solution import starts. Use it to read `RuntimeSettings`, configure flags, and set up your deployment context.

```csharp
public override void InitializeCustomExtension()
{
    if (RuntimeSettings != null && RuntimeSettings.ContainsKey("SkipData"))
    {
        DataImportBypass = Convert.ToBoolean(RuntimeSettings["SkipData"]);
    }
}
```

Pass runtime settings from the CLI: `--settings SkipData:true`

Key properties you can set here:
- `DataImportBypass` - skip all CMT data imports
- `OverrideDataImportSafetyChecks` - skip safety checks on CMT data import (only for clean environments, speeds up data import)

### 2. BeforeImportStage

```csharp
public override bool BeforeImportStage()
```

Called once before the solution import loop begins. Return `true` to continue, `false` to abort. Use it for environment validation, pre-deployment checks, or data preparation.

```csharp
public override bool BeforeImportStage()
{
    var orgVersion = CrmSvc.ConnectedOrgVersion;
    if (orgVersion < new Version("9.2.21013.00131"))
    {
        PackageLog.Log("Org version too old for single-step upgrade", TraceEventType.Error);
        return false;
    }
    return true;
}
```

### 3. PreSolutionImport

```csharp
public override void PreSolutionImport(
    string solutionName,
    bool solutionOverwriteUnmanagedCustomizations,
    bool solutionPublishWorkflowsAndActivatePlugins,
    out bool overwriteUnmanagedCustomizations,
    out bool publishWorkflowsAndActivatePlugins)
```

Called **for each solution** before its import starts. You can override `OverwriteUnmanagedCustomizations` and `PublishWorkflows` per solution.

```csharp
public override void PreSolutionImport(
    string solutionName,
    bool solutionOverwriteUnmanagedCustomizations,
    bool solutionPublishWorkflowsAndActivatePlugins,
    out bool overwriteUnmanagedCustomizations,
    out bool publishWorkflowsAndActivatePlugins)
{
    // Keep SmartDiff enabled for all solutions
    overwriteUnmanagedCustomizations = false;
    // Activate flows only for the main solution
    publishWorkflowsAndActivatePlugins = solutionName == "CoreSolution";
}
```

### 4. OverrideSolutionImportDecision

```csharp
public override UserRequestedImportAction OverrideSolutionImportDecision(
    string solutionUniqueName,
    Version organizationVersion,
    Version packageSolutionVersion,
    Version inboundSolutionVersion,
    Version deployedSolutionVersion,
    ImportAction systemSelectedImportAction)
```

From [`IImportExtensions3`](https://learn.microsoft.com/en-us/dotnet/api/microsoft.xrm.tooling.packagedeployment.crmpackageextentionbase.iimportextensions3). Called after the version-skip check, once per solution. `systemSelectedImportAction` is the `ImportAction` value PD already picked (for example `SkipSameVersion`, `SkipLowerVersion`, or `Import`). Return a `UserRequestedImportAction` to override it:

- `Default` - accept the system decision
- `Skip` - skip this solution
- `ForceUpdate` - import even if the system would have skipped (same or lower version)

```csharp
public override UserRequestedImportAction OverrideSolutionImportDecision(
    string solutionUniqueName,
    Version organizationVersion,
    Version packageSolutionVersion,
    Version inboundSolutionVersion,
    Version deployedSolutionVersion,
    ImportAction systemSelectedImportAction)
{
    // Skip satellite solutions in non-production environments
    if (solutionUniqueName.EndsWith("_Satellite")
        && RuntimeSettings != null
        && RuntimeSettings.ContainsKey("Environment")
        && RuntimeSettings["Environment"]?.ToString() != "Production")
    {
        return UserRequestedImportAction.Skip;
    }
    return UserRequestedImportAction.Default;
}
```

### 5. RunSolutionUpgradeMigrationStep

```csharp
public override void RunSolutionUpgradeMigrationStep(
    string solutionName,
    string oldVersion,
    string newVersion,
    Guid oldSolutionId,
    Guid newSolutionId)
```

The migration window. Called **after** the holding import completes but **before** `DeleteAndPromote` fires. This is the only time both old and new versions exist simultaneously in the environment.

Use it for data transformation, schema migration, or cleanup that depends on comparing old and new component states.

```csharp
public override void RunSolutionUpgradeMigrationStep(
    string solutionName,
    string oldVersion,
    string newVersion,
    Guid oldSolutionId,
    Guid newSolutionId)
{
    if (solutionName == "DataModel" && new Version(oldVersion) < new Version("2.0.0.0"))
    {
        // Migrate data from old schema to new schema
        // Both old and new solution layers are active right now
        MigrateAccountCategories(oldSolutionId, newSolutionId);
    }
}
```

> **Important:** If you implement this method with real code (not just an empty override), Package Deployer automatically switches to the two-step holding path for that solution. The one-step upgrade is not used. You can confirm this from PD's log: `User Provided Upgrade code is not detected in package. if allowed, Package Deployer will use one step upgrade pattern for this package.`

### 6. OverrideConfigurationDataFileLanguage

```csharp
public override int OverrideConfigurationDataFileLanguage(
    int selectedLanguage,
    List<int> availableLanguages)
```

Override which CMT data language file to import. Return the LCID of the language you want. Called when the package contains language-specific configuration data files.

### 7. AfterPrimaryImport

```csharp
public override bool AfterPrimaryImport()
```

Called after all solutions and data have been imported. Return `true` if successful, `false` to mark the deployment as failed. Use for post-deployment automation.

```csharp
public override bool AfterPrimaryImport()
{
    // Activate cloud flows that were imported as draft
    ActivateFlows(new[] { "ApprovalFlow", "NotificationFlow" });

    // Create application users
    EnsureApplicationUser("00000000-0000-0000-0000-000000000001", "IntegrationUser");

    // Update environment-specific connection references
    ConfigureConnectionReferences();

    return true;
}
```

## Data migrations during upgrades

The [two-step holding path](/making-solution-imports-fast/#two-step-upgrades-and-when-they-are-still-useful) exists for a reason. When you need to transform data between solution versions, the migration window provided by `RunSolutionUpgradeMigrationStep` is the supported approach.

If a managed upgrade removes a custom table or column, the platform can remove that schema and the data stored in it. That is one of the main reasons to do migrations before `DeleteAndPromote`.

### Cross-solution dependency cleanup

When removing a component referenced by higher layers, you need to control the upgrade order. Import affected solutions as holding and then apply upgrades top-down (reverse order) so the upper layer drops the dependency before the base layer is upgraded.

Example: a UI solution references a table you plan to drop from the Data Model solution. If you upgrade the Data Model first, the delete is blocked by dependencies from the UI solution.

## Configuration data and reference data

Package Deployer supports [CMT (Configuration Migration Tool)](https://learn.microsoft.com/en-us/power-platform/admin/manage-configuration-data) data packages defined in `ImportConfig.xml`. The `cmtdatafiles` element lists data files with LCID (Locale ID) based language variants.

Import ordering: solutions first, then data. This ensures schema is in place before records are created.

Key properties for data import:

- **`DataImportBypass = true`** - skip all CMT data imports. Useful when deploying to environments that already have reference data. Set this in `InitializeCustomExtension` based on `RuntimeSettings`.
- **`OverrideDataImportSafetyChecks = true`** - skip duplicate detection and other safety checks during data import. Only use on clean/empty environments. Significant performance gain for initial data loads.

## Observability

### Client-side logging

Package Deployer produces a structured `SOLUTION_STATS` log line for every solution after import completes:

```
SOLUTION_STATS=SOL:Bankaccountchangerequests|Result:Success|Operation:Import|IsUpgrade:True
  |OldVersion:1.0.0.7|NewVersion:1.0.0.8|ImportType:Async
  |RT:00:01:45.384   ← total round-trip (client wall time)
  |ST:00:01:39.865   ← server processing time
  |DPT:00:00:00.000  ← DeleteAndPromote time (0 = one-step, no separate promote)
  |WT:00:00:05.211   ← polling wait time
  |WTMB:00:00:00.000 ← metadata-blocked wait time (retries needed)
```

`DPT=0` confirms the one-step path. `WTMB` shows whether metadata-contention retries occurred.

### Dataverse logging entities

Package Deployer writes to two platform entities:

- **`package`** - a persistent "what's installed" registry. One row per package, upserted on every deployment.
- **`packagehistory`** - a per-run audit log. One row per deployment with `operationid`, status, and timing.

Both are first-party platform tables (no `msdyn_` prefix). **Neither has a foreign key back to `msdyn_solutionhistory`.** The `msdyn_correlationid` on solution history is all zeros for PD imports. To confirm a PD deployment, cross-reference `packagehistory` by time window.

## Other notable behaviors

- **Metadata-blocked retry:** If the import is rejected because a concurrent metadata operation is holding a lock, PD retries up to 10 times with 60-second waits. The retry count and wait can be tuned via the `SolutionImportBlockedRetryCountOverride` and `SolutionImportBlockedWaitOverride` settings.
- **Stale `_Upgrade` recovery:** Before starting an upgrade, PD queries for a pre-existing `{uniqueName}_Upgrade` holding solution. If one is found, PD deletes it before proceeding, then re-imports from scratch. This prevents interrupted upgrades from blocking the next deployment.
- **Post-loop `PublishAll`:** If the package contains unmanaged solutions, PD issues a `PublishAllXmlRequest` after all imports complete.
- **Orchestration queuing (PD 4.0+):** Packages can be submitted as a single async job via `QueuePackageDeployment`. Falls back to the per-solution loop if unsupported.
- **`AsyncRibbonProcessing`:** PD can pass `AsyncRibbonProcessing=true` per-solution when configured (org ≥ 9.1.0.15400). PAC CLI has no flag for this.

Package Deployer ships as .NET Framework (net462) assemblies, **Windows-only**. The modern .NET version of PAC CLI (net10.0) dropped Package Deployer and CMT entirely rather than porting them. If you need to run Package Deployer or CMT data imports on Linux or macOS (e.g., containerized CI agents), [TALXIS CLI](https://github.com/TALXIS/tools-cli) (`txc`) solves this by IL-patching the original net462 assemblies with Mono.Cecil at startup so they load on modern .NET cross-platform. Commands: `txc environment deploy <package>` for Package Deployer, `txc data import <path>` for CMT data packages.

## Catalog: platform-managed Package Deployer {#catalog}

Instead of running Package Deployer yourself, you can submit a package to the [Power Platform Catalog](https://learn.microsoft.com/en-us/power-platform/developer/catalog/overview) and let the platform execute it. The catalog uses the same backend service (Trusted Publisher Service / TPS) that processes AppSource installs and first-party platform updates.

If your CI/CD pipelines run during weekends or off-hours, they can collide with platform servicing windows. First-party updates are deployed on a schedule and your concurrent `ImportSolution` calls compete for the same metadata locks, leading to retries, failures, and much longer deployment times. Catalog deployments enter the same internal queue, so they're scheduled alongside platform updates rather than fighting them.

### Submitting a package

Every catalog item is a Package Deployer package. If you already have a `.pdpkg.zip`, you can submit it directly. If you only have a solution ZIP, the platform can convert it into a PD package for you (via the Maker UI or the `mspcat_PackageStore` entity with `operation: Create Package`).

Submit via CLI:

```bash
pac catalog submit --path ./submission.json
```

The [submission metadata JSON](https://learn.microsoft.com/en-us/power-platform/developer/catalog/submit-items) references your package file and includes publisher info, description, and deployment type (template or managed).

Or via SDK: use the `mspcat_SubmitCatalogApprovalRequest` message. Submissions go through an approval workflow (configurable, auto-approve is possible).

### Installing a catalog item

```bash
pac catalog install -cid MyCatalogItem -te https://target-org.crm.dynamics.com/
```

Or via SDK: `mspcat_InstallCatalogItemByCID` message. You can pass `RuntimeSettings` (same pipe-delimited format as PD's `RuntimeSettings`):

```csharp
var request = new mspcat_InstallCatalogItemByCIDRequest
{
    CID = "MyCatalogItem@2.0.0.0",
    DeployToOrganizationUrl = "https://target-org.crm.dynamics.com/",
    Settings = "SkipData:true|Environment:Production"
};
```

Version pinning is supported: `MyCatalogItem@2.0.0.0` installs that specific version.

### Tracking installation status

The `mspcat_InstallHistory` entity tracks each deployment: Requested → Pending → In Progress → Completed / Failed. Check via:

```bash
pac catalog status --tracking-id <guid> --type Install
```

### When to use Catalog over running PD yourself

- You want the platform to handle execution, retries, and queuing - no pipeline agent needed.
- You want to avoid import conflicts with platform servicing windows.
- You're distributing packages to multiple environments or tenants within your organization.
- You want an approval workflow before deployments.

---

That wraps up the series. [Part 1](/dataverse-solution-component-types/) covers what's inside a solution ZIP and the two component architectures. [Part 2](/making-solution-imports-fast/) covers import types, SmartDiff, and how to diagnose slow imports. This post covered the orchestration layer on top of all that. If you're deploying anything beyond a single solution, Package Deployer is worth the setup cost.
