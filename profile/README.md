# Zonit

Open-source C# libraries for building **.NET 10 / Blazor** applications — a strict value-object
foundation, a multi-site web kernel, in-process messaging, and a unified AI layer.

Everything targets `net10.0`, ships with XML docs and symbol packages, and is trim-annotated.

---

## 🔌 Extensions

The `Zonit.Extensions` repository is a monorepo: one value-object foundation plus the domain cores
and the web kernel layered on top. The five domain cores have **no ASP.NET Core dependency** — they
work from a console, worker or WASM host.

| Package | NuGet | Downloads | What it does |
|---|---|---|---|
| [Zonit.Extensions](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.svg)](https://www.nuget.org/packages/Zonit.Extensions/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.svg) | Value-object foundation: `Identity`, `Permission`, `Role`, `Title`, `Url`, `Asset`, `Price`/`Money`, `Schedule`, `Culture`, `Zone`. Registers nothing in DI. |
| [Zonit.Extensions.Auth](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Auth.svg)](https://www.nuget.org/packages/Zonit.Extensions.Auth/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Auth.svg) | Framework-agnostic auth core. You implement `IAuthSource`; the web side adds `[RequirePermission]`. |
| [Zonit.Extensions.Cultures](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Cultures.svg)](https://www.nuget.org/packages/Zonit.Extensions.Cultures/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Cultures.svg) | Translations, per-scope culture and time-zone state. |
| [Zonit.Extensions.Organizations](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Organizations.svg)](https://www.nuget.org/packages/Zonit.Extensions.Organizations/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Organizations.svg) | Workspace context — the active organization and switching between them. |
| [Zonit.Extensions.Projects](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Projects.svg)](https://www.nuget.org/packages/Zonit.Extensions.Projects/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Projects.svg) | Catalog / project context scoped to the active organization. |
| [Zonit.Extensions.Tenants](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Tenants.svg)](https://www.nuget.org/packages/Zonit.Extensions.Tenants/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Tenants.svg) | Per-domain tenant identity and settings, with strongly-typed plugin settings. |
| [Zonit.Extensions.Website](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Website.svg)](https://www.nuget.org/packages/Zonit.Extensions.Website/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Website.svg) | The web kernel: several Sites in one app, plug-in areas, layouts, page base classes, navigation, breadcrumbs, toasts, cookies. |
| [Zonit.Extensions.Website.MudBlazor](https://github.com/Zonit/Zonit.Extensions) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Website.MudBlazor.svg)](https://www.nuget.org/packages/Zonit.Extensions.Website.MudBlazor/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Website.MudBlazor.svg) | MudBlazor inputs bound to value objects, plus layout primitives. |
| [Zonit.Extensions.Ai](https://github.com/Zonit/Zonit.Extensions.Ai) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Ai.svg)](https://www.nuget.org/packages/Zonit.Extensions.Ai/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Ai.svg) | One `IAiProvider` over OpenAI, Anthropic, Google, xAI and ElevenLabs — text, images, embeddings, audio, agents, MCP, cost tracking. |
| [Zonit.Extensions.Databases](https://github.com/Zonit/Zonit.Extensions.Databases) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Extensions.Databases.svg)](https://www.nuget.org/packages/Zonit.Extensions.Databases/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Extensions.Databases.svg) | Repository abstractions over EF Core, with source-generated value-object mapping. |

> **Note on `*.Abstractions`.** Every `Zonit.Extensions.*.Abstractions` package is **retired**. The
> abstractions moved into the main package of each domain. Do not add them to new projects.

---

## ✉️ Messaging

In-process **Commands** (CQRS), **Events** (pub/sub), **Tasks** (background jobs) and **Schedules**
(recurring jobs). Handlers are discovered by a source generator at compile time — no assembly
scanning, no reflection.

| Package | NuGet | Downloads | What it does |
|---|---|---|---|
| [Zonit.Messaging.Commands](https://github.com/Zonit/Zonit.Services.EventMessage) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Messaging.Commands.svg)](https://www.nuget.org/packages/Zonit.Messaging.Commands/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Messaging.Commands.svg) | Request/response with `IRequestHandler<,>`. |
| [Zonit.Messaging.Events](https://github.com/Zonit/Zonit.Services.EventMessage) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Messaging.Events.svg)](https://www.nuget.org/packages/Zonit.Messaging.Events/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Messaging.Events.svg) | Pub/sub fan-out with `IEventHandler<T>`. |
| [Zonit.Messaging.Tasks](https://github.com/Zonit/Zonit.Services.EventMessage) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Messaging.Tasks.svg)](https://www.nuget.org/packages/Zonit.Messaging.Tasks/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Messaging.Tasks.svg) | Background jobs with progress, retries and monitoring. |
| [Zonit.Messaging.Schedules](https://github.com/Zonit/Zonit.Services.EventMessage) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Messaging.Schedules.svg)](https://www.nuget.org/packages/Zonit.Messaging.Schedules/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Messaging.Schedules.svg) | Recurring jobs driven by the `Schedule` value object. |

---

## 🛠️ Services

| Package | NuGet | Downloads | What it does |
|---|---|---|---|
| [Zonit.Services.Dashboard](https://github.com/Zonit/Zonit.Services.Dashboard) | [![NuGet](https://img.shields.io/nuget/v/Zonit.Services.Dashboard.svg)](https://www.nuget.org/packages/Zonit.Services.Dashboard/) | ![Downloads](https://img.shields.io/nuget/dt/Zonit.Services.Dashboard.svg) | Admin dashboard UI on top of the web kernel: MudBlazor chrome, drawer/toolbar extension slots, theming. |

---

## 🧩 Plugins

- [**Zonit.Plugins**](https://github.com/Zonit/Zonit.Plugins) — shared contracts and event types for
  plugin-based architectures.

---

## 🤖 Built for AI-assisted development

Installing a Zonit package also installs **instructions for your AI assistant**. At build time the
package unpacks guides into the surfaces your editor already reads:

```
.zonit/extensions/<area>/*.md    neutral guides, human- and agent-readable
.claude/skills/…/SKILL.md        Claude Code skill
.cursor/rules/*.mdc              Cursor rules, glob-scoped
.github/instructions/*.md        Copilot instructions
CLAUDE.md · .github/copilot-instructions.md
```

Only editors actually present in your repo get files, nothing is written on CI, and each package
ships only its own guides — installing `Zonit.Extensions.Cultures` does not dump the web-kernel
documentation on you. Turn it off with `<ZonitExtInstructions>false</ZonitExtInstructions>`.

---

## Getting started

```bash
dotnet add package Zonit.Extensions.Website
```

```csharp
builder.Services.AddWebsite(o => o.AddArea<MyArea>());   // registers Auth, Cultures,
                                                         // Organizations, Projects, Tenants too
var app = builder.Build();
app.UseWebsite<App>("/", o => o.AddArea<MyArea>());
app.Run();
```

Each repository's `Readme.md` documents its own packages; the per-package guides under
`.zonit/extensions/` go deeper.

---

**Contributions welcome** — open an issue or a pull request on any repository.
