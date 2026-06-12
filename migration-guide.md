# Migracja z .NET Framework 4.8 do .NET 8

## Strategia migracji — wybór podejścia

### Opcja A: Big Bang (przepisanie)
- Dla małych projektów (< 50k LOC)
- Gdy architektura jest mocno zepsuta
- Ryzyko: wysokie, długi czas dostawy

### Opcja B: Strangler Fig (stopniowa migracja)
- Dla dużych systemów produkcyjnych
- Nowe funkcje w .NET 8, stare zostają w 4.8
- Komunikacja przez HTTP / message bus
- Ryzyko: utrzymanie dwóch codebase przez pewien czas

### Opcja C: In-place upgrade z dotnet upgrade-assistant
- Najlepiej dla czystych bibliotek i prostych API
- Automatyzuje wiele kroków
- Nadal wymaga ręcznej pracy przy `System.Web`

---

## Mapa zmian API

### System.Web → ASP.NET Core

| .NET Framework 4.8                   | .NET 8 / ASP.NET Core               |
|--------------------------------------|--------------------------------------|
| `HttpContext.Current`                | `IHttpContextAccessor`               |
| `HttpRequest.QueryString["key"]`     | `HttpRequest.Query["key"]`           |
| `HttpResponse.Redirect(url)`         | `HttpContext.Response.Redirect(url)` |
| `Session["key"]`                     | `ISession` / distributed cache      |
| `FormsAuthentication`                | Cookie Authentication middleware     |
| `WebConfigurationManager`            | `IConfiguration`                     |
| `Global.asax` Application_Start      | `Program.cs` / `Startup.cs`         |
| `RouteConfig.RegisterRoutes`         | Attribute routing / Minimal API      |
| `FilterConfig.RegisterGlobalFilters` | Middleware / `AddControllers()`      |
| `BundleConfig`                       | Webpack / Vite / LibMan              |

### Konfiguracja

```csharp
// .NET Framework — web.config / ConfigurationManager
string val = ConfigurationManager.AppSettings["MyKey"];

// .NET 8 — appsettings.json / IConfiguration
// appsettings.json:
// { "MyKey": "value" }

// W kontrolerze / serwisie:
public class MyService
{
    private readonly IConfiguration _config;
    public MyService(IConfiguration config) => _config = config;

    public void DoWork()
    {
        string val = _config["MyKey"];
        // lub z sekcjami:
        var section = _config.GetSection("MySection").Get<MyOptions>();
    }
}
```

### Dependency Injection

```csharp
// .NET Framework — Autofac w Global.asax
builder.RegisterType<OrderService>().As<IOrderService>().InstancePerRequest();

// .NET 8 — wbudowany kontener w Program.cs
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddSingleton<ICacheService, RedisCacheService>();
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();
```

### Entity Framework 6 → EF Core 8

```csharp
// EF 6 — DbContext
public class AppDbContext : DbContext
{
    public AppDbContext() : base("name=DefaultConnection") { }
}

// EF Core 8 — DbContext
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Fluent API — podobna składnia, ale nowe możliwości
        modelBuilder.Entity<Product>()
            .HasKey(p => p.Id);
    }
}

// Rejestracja w Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

**Uwaga EF6 → EF Core**: 
- `LazyLoadingEnabled` domyślnie OFF w EF Core (trzeba włączyć proxy lub użyć `.Include()`)
- `ObjectContext` nieobsługiwany
- EDMX / Database-First wymaga regeneracji z EF Core Power Tools

---

## WCF → alternatywy

WCF **nie istnieje** w .NET 8 (poza CoreWCF — open-source port).

| Scenariusz                  | Rekomendacja                              |
|-----------------------------|-------------------------------------------|
| Internal service-to-service | **gRPC** (`Grpc.AspNetCore`)              |
| External API (HTTP)         | **ASP.NET Core Web API** (REST)           |
| BasicHttpBinding            | **CoreWCF** (drop-in replacement)         |
| NetTcpBinding               | **CoreWCF** lub gRPC                      |
| MSMQ / message queue        | **MassTransit** + RabbitMQ / Azure SB     |

```bash
# CoreWCF — minimalna migracja
dotnet add package CoreWCF.Http
dotnet add package CoreWCF.Primitives
```

---

## Kroki migracji krok po kroku

### Faza 1: Przygotowanie (bez zmiany TFM)
1. Zaktualizuj wszystkie NuGet do najnowszych wersji kompatybilnych z 4.8
2. Włącz `<Nullable>enable</Nullable>` i napraw ostrzeżenia
3. Zmień `LangVersion` na `latest` i użyj nowoczesnej składni C#
4. Dodaj testy tam gdzie ich nie ma (priorytety: krytyczne ścieżki)
5. Usuń martwy kod (`#if DEBUG` bloki, nieużywane serwisy)

### Faza 2: Multi-targeting (opcjonalna)
```xml
<TargetFrameworks>net48;net8.0</TargetFrameworks>
```
Pozwala wykryć problemy kompilacji bez porzucania 4.8.

### Faza 3: Migracja TFM
1. Zmień `<TargetFramework>net48</TargetFramework>` → `net8.0`
2. Usuń `System.Web.*` referencje
3. Zamień `packages.config` → `PackageReference` (jeśli jeszcze nie)
4. Napraw błędy kompilacji (zacznij od klas infrastruktury, nie UI)

### Faza 4: ASP.NET Core (jeśli projekt webowy)
1. Stwórz nowy projekt ASP.NET Core
2. Przenieś modele i serwisy (zazwyczaj bez zmian)
3. Przepisz kontrolery (dziedziczą z `ControllerBase` nie `Controller`)
4. Podmień `Global.asax` na `Program.cs`
5. Przenieś `web.config` → `appsettings.json`

---

## Przydatne narzędzia

```bash
# 1. Upgrade Assistant — analiza i częściowa automatyzacja
dotnet tool install -g upgrade-assistant
upgrade-assistant upgrade MySolution.sln --target-tfm-support LTS

# 2. Compatibility Analyzer (Roslyn)
dotnet add package Microsoft.DotNet.Analyzers.Compatibility

# 3. try-convert — konwersja starego .csproj na SDK-style
dotnet tool install -g try-convert
try-convert --workspace MySolution.sln
```

---

## Szacowanie nakładu migracji

| Rozmiar projektu | Typ              | Szacowany czas |
|------------------|------------------|----------------|
| < 10k LOC        | Biblioteka       | 1–3 dni        |
| < 10k LOC        | Web API          | 3–7 dni        |
| 10–50k LOC       | MVC App          | 2–6 tygodni    |
| 50–200k LOC      | Duża aplikacja   | 2–6 miesięcy   |
| > 200k LOC       | System enterprise| 6–18 miesięcy  |

Czynniki zwiększające nakład:
- Użycie `HttpContext.Current` w warstwie serwisowej (+30%)
- Wiele projektów WCF (+50% na projekt)
- Brak testów jednostkowych (+40%)
- Użycie COM Interop / P/Invoke (+20–100%)
- Zależność od specyficznych bibliotek Windows (+varies)
