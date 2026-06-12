---
name: dotnet-framework-48
description: >
  Use this skill for any task involving .NET Framework 4.8 — including legacy codebase analysis,
  migration planning, C# code generation, WCF/WPF/WinForms development, MSBuild configuration,
  COM interop, reflection, dependency injection in full-framework projects, and debugging production
  issues on IIS. Trigger whenever the user mentions .NET Framework (as opposed to .NET Core / .NET 5+),
  legacy ASP.NET (MVC 5, Web API 2, WebForms), Windows services, or any project targeting
  net48 / net472 / net462 or earlier. Also trigger for questions about interoperability between
  .NET Framework and modern .NET, testing private methods via reflection or InternalsVisibleTo,
  or diagnosing culture/locale issues (e.g. double.TryParse differences between Windows and Linux).
  Do NOT use for .NET 6/7/8/9 (those are .NET Core lineage).
---

# .NET Framework 4.8 Skill

## Szybka orientacja

.NET Framework 4.8 to **ostatnia wersja pełnego .NET Framework** (wydana maj 2019).  
Działa **wyłącznie na Windows**. Nie jest tym samym co .NET Core / .NET 5+.  
Projekty targetują `net48` (lub wcześniejsze TFM: `net472`, `net462`, …).

---

## 1. Kluczowe obszary techniczne

### 1.1 Typy projektów i szablony

| Typ projektu        | Przestrzeń nazw / technologia           |
|---------------------|-----------------------------------------|
| ASP.NET MVC 5       | `System.Web.Mvc`                        |
| ASP.NET Web API 2   | `System.Web.Http`                       |
| ASP.NET WebForms    | `System.Web.UI`                         |
| WPF                 | `System.Windows` (PresentationFramework)|
| WinForms            | `System.Windows.Forms`                  |
| WCF Service         | `System.ServiceModel`                   |
| Windows Service     | `System.ServiceProcess`                 |
| Class Library       | `net48` TFM w `.csproj`                 |

### 1.2 Konfiguracja projektu — stary vs nowy format .csproj

**Stary format (legacy, typowy dla VStudio pre-2017):**
```xml
<Project ToolsVersion="15.0" DefaultTargets="Build"
         xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
  </PropertyGroup>
</Project>
```

**SDK-style (obsługiwany od VS 2017+ dla bibliotek):**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
</Project>
```

> ⚠️ ASP.NET MVC 5 / WebForms / WCF **wymagają starego formatu** — nie można ich przepisać na SDK-style bez dużego nakładu.

---

## 2. Najczęstsze pułapki i gotowe rozwiązania

### 2.1 Kultura / locale a parsowanie liczb

**Problem**: `double.TryParse("3.14")` działa na Windows (Polish locale), ale **nie** na Linux w kontenerze Docker.

```csharp
// ŹLE — zależy od CurrentCulture
double.TryParse(value, out double result);

// DOBRZE — zawsze InvariantCulture
double.TryParse(value,
    NumberStyles.Any,
    CultureInfo.InvariantCulture,
    out double result);
```

Dotyczy też: `DateTime.Parse`, `decimal.Parse`, `float.TryParse`.

### 2.2 Testowanie prywatnych metod

Trzy podejścia — wybierz zależnie od kontekstu:

```csharp
// 1. Reflection (zawsze dostępne, wolniejsze)
var method = typeof(MyClass).GetMethod("PrivateMethod",
    BindingFlags.NonPublic | BindingFlags.Instance);
var result = method.Invoke(instance, new object[] { arg });

// 2. InternalsVisibleTo (wymaga atrybutu w AssemblyInfo.cs)
// W AssemblyInfo.cs projektu testowanego:
[assembly: InternalsVisibleTo("MyProject.Tests")]
// Zmień private -> internal

// 3. PrivateObject (MSTest v1 / v2 helper)
var po = new PrivateObject(instance);
var result = po.Invoke("PrivateMethod", arg);
```

### 2.3 async/await w .NET 4.8

.NET 4.8 **obsługuje** `async/await` (C# 5+), ale brakuje wielu API z `System.Threading.Tasks` dostępnych w .NET Core.

```csharp
// Brakuje: ValueTask, IAsyncEnumerable, Task.WhenEach
// Dostępne: Task, Task<T>, async/await, ConfigureAwait(false)

// W ASP.NET MVC 5 — action może być async:
public async Task<ActionResult> Index()
{
    var data = await _service.GetDataAsync();
    return View(data);
}
```

### 2.4 Dependency Injection (brak wbudowanego kontenera)

.NET Framework 4.8 nie ma `Microsoft.Extensions.DependencyInjection` out-of-the-box.  
Standardowe opcje:

- **Autofac** — najpopularniejszy, dobre wsparcie dla MVC 5 / Web API 2
- **Unity** — stary Microsoft container
- **SimpleInjector** — lekki, szybki
- **Castle Windsor** — enterprise, dojrzały

```csharp
// Autofac + MVC 5 (Global.asax.cs)
var builder = new ContainerBuilder();
builder.RegisterControllers(typeof(MvcApplication).Assembly);
builder.RegisterType<MyService>().As<IMyService>().InstancePerRequest();
var container = builder.Build();
DependencyResolver.SetResolver(new AutofacDependencyResolver(container));
```

### 2.5 Entity Framework (pełny EF, nie EF Core)

.NET 4.8 używa **Entity Framework 6.x** (nie EF Core).

```csharp
// DbContext — EF6
public class AppDbContext : DbContext
{
    public AppDbContext() : base("name=DefaultConnection") { }
    public DbSet<Product> Products { get; set; }
}

// Migracje Code First:
// Enable-Migrations, Add-Migration, Update-Database (Package Manager Console)
```

---

## 3. Analiza legacy codebase

Kiedy użytkownik prosi o analizę istniejącego projektu .NET 4.8, wykonaj kolejno:

1. **Zinwentaryzuj zależności** — przejrzyj `packages.config` lub `<PackageReference>` w `.csproj`
2. **Sprawdź wersję C#** — `.csproj` → `<LangVersion>` (domyślnie C# 5 dla starych projektów!)
3. **Zidentyfikuj wzorce architektoniczne** — foldery `Controllers/`, `Services/`, `Repositories/`, `Models/`
4. **Znajdź problemy migracyjne** — `System.Web.*`, `HttpContext.Current`, `Thread.CurrentPrincipal`
5. **Oceń pokrycie testami** — szukaj projektów `*.Tests`, `*.UnitTests`

Szczegółowe checklisty → `references/legacy-analysis.md`

---

## 4. Migracja do .NET 8+

Jeśli zadanie dotyczy migracji, przeczytaj: `references/migration-guide.md`

Kluczowe decyzje:
- `System.Web` → Microsoft.AspNetCore (brak 1:1 mappingu)
- `HttpContext.Current` → `IHttpContextAccessor`
- `ConfigurationManager` → `IConfiguration` (appsettings.json)
- WCF → gRPC / REST / CoreWCF (open-source port)
- WinForms/WPF → dostępne na .NET 8, ale wymagają Windows

---

## 5. IIS i deployment

```xml
<!-- web.config — kluczowe sekcje -->
<system.web>
  <compilation debug="false" targetFramework="4.8" />
  <httpRuntime targetFramework="4.8" maxRequestLength="10240" />
  <customErrors mode="Off" />
</system.web>

<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="10485760" />
    </requestFiltering>
  </security>
</system.webServer>
```

**Application Pool**: ustaw na `No Managed Code` gdy używasz OWIN self-host, albo `.NET CLR v4.0` dla klasycznego managed mode.

---

## 6. NuGet i packages.config vs PackageReference

Stare projekty używają `packages.config`. Migracja do `PackageReference`:

```bash
# Visual Studio: prawym na packages.config → "Migrate packages.config to PackageReference"
# Lub ręcznie w .csproj:
<ItemGroup>
  <PackageReference Include="Autofac" Version="6.5.0" />
  <PackageReference Include="EntityFramework" Version="6.4.4" />
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

---

## 7. Często używane NuGet (ecosystem .NET 4.8)

| Cel                     | Pakiet                                      |
|-------------------------|---------------------------------------------|
| JSON                    | `Newtonsoft.Json` (nie System.Text.Json!)   |
| ORM                     | `EntityFramework` 6.x                       |
| DI Container            | `Autofac`, `Autofac.Mvc5`, `Autofac.WebApi2`|
| HTTP Client             | `RestSharp`, `HttpClient` (wbudowany)       |
| Logowanie               | `NLog`, `log4net`, `Serilog`                |
| Testy jednostkowe       | `MSTest.TestFramework`, `NUnit`, `xUnit`    |
| Mocking                 | `Moq`, `NSubstitute`                        |
| Mapper                  | `AutoMapper` (≤ v10 dla net48)             |
| Walidacja               | `FluentValidation`                          |
| Task Queue / BG Jobs    | `Hangfire`                                  |

---

## 8. Ograniczenia vs .NET Core / .NET 5+

| Funkcja                          | .NET 4.8   | .NET 8     |
|----------------------------------|------------|------------|
| Cross-platform (Linux/Mac)       | ❌         | ✅         |
| `System.Text.Json` (wbudowany)   | ❌         | ✅         |
| `IAsyncEnumerable`               | ❌         | ✅         |
| Minimal API / Blazor             | ❌         | ✅         |
| `Span<T>` / `Memory<T>`          | ⚠️ częściowo | ✅       |
| Docker (oficjalne obrazy)        | ⚠️ Windows only | ✅   |
| Wbudowany DI                     | ❌         | ✅         |
| `Record` types (C# 9+)           | ✅ (z LangVersion=latest) | ✅ |
| `nullable` reference types       | ✅ (z LangVersion=8+) | ✅ |

---

## Zasoby pomocnicze

- `references/legacy-analysis.md` — checklista analizy legacy codebase
- `references/migration-guide.md` — ścieżki migracji do .NET 8
