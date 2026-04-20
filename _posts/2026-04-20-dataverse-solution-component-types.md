---
title: "Dataverse solution component types"
date: 2026-04-20
permalink: /dataverse-solution-component-types/
author: Tomas Prokop
categories:
  - Microsoft
  - Microsoft Power Platform
tags:
  - Power Platform
  - ALM
  - Solution Framework
  - Dataverse
excerpt: "Two architectures for solution components live inside Dataverse. They import differently, diff differently, and show up differently in source control. This is Part 1 of a 3-part series."
---

> This is Part 1 of a 3-part series on Dataverse solution deployments:
> 1. **Dataverse solution component types** (you are here)
> 2. [Making Dataverse solution imports fast](/making-solution-imports-fast/)
> 3. [Package Deployer for solutions, data, and migrations](/package-deployer-solutions-data-migrations/)

The docs explain solutions as "containers for customizations." That is correct but not useful when you need to understand why some imports are fast and others are painfully slow. Or why some components show clean diffs in source control and others are opaque blobs.

This post covers the two distinct component architectures inside Dataverse. They import differently, diff differently, and have different tradeoffs. Understanding the difference helps you make better decisions about ALM tooling and troubleshooting.

## What is in a solution ZIP

A managed solution is a ZIP file. The most important files inside:

- **`solution.xml`** - the manifest. Lists `RootComponents` with their type codes and schema names. Contains solution metadata like version, publisher, and dependencies.
- **`customizations.xml`** - the definitions. This is where platform component definitions live. Entities, attributes, relationships, forms, views, sitemap fragments, ribbons, all packed into one XML file.
- **Individual payload files** - some components (plugins, web resources, SCF components) have separate files referenced from the manifest.

When you unpack a solution with [Solution Packager](https://learn.microsoft.com/en-us/power-platform/alm/solution-packager-tool), it splits `customizations.xml` into individual files organized by component type. Each entity gets a folder. Each form gets a file. This is what ends up in source control.

## Managed vs unmanaged

> **Important:** Unmanaged solutions are only supposed to be used for export to source control and hydrating developer environments. This post doesn't consider deploying unmanaged solutions beyond developer environments as a valid option.

Dataverse uses a **layering system** for solution components. Every component exists in one or more layers.

- The **System layer** is at the bottom. It contains out-of-box definitions shipped by Microsoft.
- **Managed layers** stack above the system layer in installation order. Each managed solution gets its own layer.
- The **Active (unmanaged) layer** sits on top. This is where maker customizations go when someone edits a component directly in the environment.

All unmanaged solutions in the same environment share that one Active layer. Working in Solution A versus Solution B does not isolate makers from each other. Preferred solution changes where new components are added, not which unmanaged layer they land in.

When Dataverse needs to compute the current state of a component, it resolves layers top-down. For most component types, the top layer simply wins. **Forms, site maps, and model-driven apps** are the notable merge cases. If a managed layer removes a property, the value from the layer below can surface again.

This is why `OverwriteUnmanagedCustomizations` matters for import performance. When set to `true`, the import overwrites the active layer, which forces Dataverse to reprocess every component. When `false`, SmartDiff can skip unchanged components entirely. More on that in [Part 2](/making-solution-imports-fast/).

### Layers versus built-in solutions

One thing that confuses almost everyone at first is that Dataverse mixes **layer concepts** and **solution containers**.

- **System** is the base platform layer.
- **Active** is the top unmanaged layer.
- **Default Solution** is the special built-in solution that exposes all components in the environment.
- **Common Data Service Default Solution** is the built-in maker default where new components land unless you choose another unmanaged solution.

Those are not interchangeable ideas. **System** and **Active** describe how Dataverse computes runtime behavior. **Default Solution** and **Common Data Service Default Solution** describe where components are surfaced or created.

This is also where **Preferred solution** fits in. Preferred solution is a per-maker routing setting. If you set it, newly created solution-aware components are added to that unmanaged solution instead of the Common Data Service Default Solution. If you do not set it, Dataverse uses the Common Data Service Default Solution by default.

There is also a hidden **Basic** solution. It is not part of the public maker model and it is not a place to author customizations. One of the reasons for it is **Block unmanaged customizations**. That feature is supposed to stop ordinary unmanaged maker edits in production environments. But the platform and some features still need ways to create certain components in runtime without treating them like ad hoc customizations in the Active layer.

### Built-in Dataverse solutions and GUIDs

| Solution | GUID | Notes |
|---|---|---|
| **System** | `FD140AAD-4DF4-11DD-BD17-0019B9312238` | Base platform layer |
| **Active** | `FD140AAE-4DF4-11DD-BD17-0019B9312238` | Active layer - top unmanaged layer |
| **Default Solution** | `FD140AAF-4DF4-11DD-BD17-0019B9312238` | View to all components |
| **Common Data Service Default Solution** | `00000001-0000-0000-0001-00000000009B` | Built-in maker default |
| **Basic** | `25A01723-9F63-4449-A3E0-046CC23A2902` | Hidden internal solution |

If you want to inspect them directly, query the `solution` table and filter by `solutionid`, `uniquename`, `friendlyname`, `isvisible`, and `ismanaged`.

### What "managed" means for ownership

A managed component is owned by its solution publisher. Makers in the target environment can customize it (adding an active layer on top) but cannot delete it. Only uninstalling or upgrading the managed solution can remove managed components.

This is the mechanism that enables safe component removal during upgrades. The [single-step upgrade](/making-solution-imports-fast/#why-single-step-is-different) (`StageAndUpgradeRequest`) compares the incoming solution against the installed version and deletes components that are no longer present.

### Managed properties

Managed ownership is only half the story. **Managed properties** are the switches the original publisher sets to control how much downstream customization is allowed. They are configured while the component is still unmanaged in development, then enforced after the managed solution is installed elsewhere.

That matters because "managed" does not automatically mean "locked down." A table can be managed but still allow new forms, new views, or label changes. Or it can be tightly restricted. If a downstream team says "we can't change this managed component," the next question is whether the publisher disabled that through managed properties.

### Publisher strategy: pick one early

Use **one publisher across related solutions** unless you have a strong reason to split ownership. The publisher owns managed components, and that ownership association is not something you change later.

The prefix choice also sticks. Schema names, logical names, and choice value prefixes are created in the context of that publisher. If you later decide the component should really belong to a different publisher, you are usually looking at delete-and-recreate, plus data migration if the component stores data. Pick the publisher and prefix like you expect to live with them for years.

## Platform components

These are the original component types. They are implemented directly in the Dataverse codebase and identified by fixed, well-known type codes. The full list is published in the [SolutionComponent entity reference](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/reference/entities/solutioncomponent#BKMK_ComponentType).

Examples: entities (type code 1), attributes (2), relationships (10), forms (60), views (26), plugins (90), web resources (61).

Type codes are static. They don't change between environments.

Platform component definitions land in `customizations.xml` inside the solution ZIP. Solution Packager knows exactly how to decompose them into individual files because the schema is fixed and documented.

In source control, platform components are readable. You can diff a form XML and see which tabs or sections changed. You can review an entity definition and spot new attributes. Code review works.

### Table segmentation keeps layers cleaner

For existing tables, include **only the assets you actually changed**. If you changed one column and one form, ship that. Do not drag the whole table into every update unless the table is brand new in the target environment.

Smaller table payloads create fewer unnecessary layers, fewer accidental dependencies, and fewer chances to stomp on someone else's top-wins component or form merge. It also helps import performance, which is one of the reasons [Part 2](/making-solution-imports-fast/) keeps coming back to smaller, more focused updates.

### SmartDiff for platform components

SmartDiff coverage on the platform side expands over time as Microsoft enables it for additional component types. When your import processes fewer components than the total in the solution, SmartDiff is working. When it processes all of them, either everything legitimately changed, `OverwriteUnmanagedCustomizations` is `true`, or the component type isn't covered by SmartDiff yet.

## SCF components

Solution Component Framework (SCF) is the newer architecture. Product teams at Microsoft use it to add new component types to Dataverse without changing the core platform code.

SCF components are registered via `solutioncomponentdefinition` metadata, brought to environments dynamically by Microsoft first-party solutions. Each product team decides what their component looks like in a solution export.

### How SCF differs from platform components

**Runtime-assigned type codes** - SCF components use `objecttypecode` values typically above 1000. These codes are assigned at runtime and can differ between environments. Dataverse resolves SCF components by their unique component names on import, not by type code.

**No static schema** - Each component owner decides the export format. Some use JSON. Some use XML. There is no single schema reference for validation.

**Lax import validation** - SCF component types vary in how strictly they validate incoming payloads. Some accept values that only fail at runtime rather than at import time, so errors can surface later than you would expect.

**Worse readability in source control** - SCF component files are typically files with non-descriptive name and unnecessary folder nesting. JSON payloads with GUIDs and encoded properties. Not friendly for code review or conflict resolution.

### SmartDiff for SCF components

SCF was designed with diff support built into the framework itself. SCF components get SmartDiff coverage automatically as new types are introduced. This is different from platform components where SmartDiff has to be enabled per type.

### Discovering SCF components

To see which SCF component types exist in a given environment:

```http
GET ORG/api/data/v9.2/solutioncomponentdefinitions?$select=name,objecttypecode
```

The response shows you every registered component type with its runtime type code. Compare across environments to see why type codes might differ.

## Deployment methods

Today the platform supports multiple ways to deploy solutions:

- **Maker UI** - manual import through `make.powerapps.com`
- **[PAC CLI](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/solution#pac-solution-import)** - `pac solution import` with various flags
- **Azure DevOps tasks and GitHub Actions** - [wrappers over PAC CLI](https://github.com/microsoft/powerplatform-cli-wrapper)
- **[Power Platform Pipelines](https://learn.microsoft.com/en-us/power-platform/alm/pipelines)** - managed by the platform, abstracts import details server-side
- **[Package Deployer](https://learn.microsoft.com/en-us/power-platform/alm/package-deployer-tool?tabs=cli)** - multi-solution orchestration with custom code hooks. You run Package Deployer yourself (locally, in a pipeline, or via PowerShell).
- **[Catalog](https://learn.microsoft.com/en-us/power-platform/developer/catalog/overview)** - submit a Package Deployer package to the platform and let it handle execution. Installations go through the same internal service (TPS) that processes AppSource installs and first-party Microsoft updates, so your deployment is queued alongside platform servicing rather than competing with it. Useful when weekend servicing windows cause import conflicts. See [Part 3](/package-deployer-solutions-data-migrations/#catalog) for details.
- **Native Git integration** - component-level sync between a dev environment and an Azure DevOps repo. Bypasses the solution import pipeline entirely. See [Part 2](/making-solution-imports-fast/#how-does-native-git-integration-relate-to-this).

All of these end up calling Dataverse APIs underneath (except native Git, which uses its own code path). The differences are in what parameters they pass, what defaults they use, and how much control you get.

[Part 2](/making-solution-imports-fast/) covers which import types exist and how to make them fast. [Part 3](/package-deployer-solutions-data-migrations/) covers Package Deployer for teams that need orchestration, custom code, and data migrations.
