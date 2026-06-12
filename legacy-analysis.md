# Analiza Legacy Codebase — .NET Framework 4.8

## Checklista wstępnej analizy

### Krok 1: Struktura projektu

```
[ ] Czy solution zawiera wiele projektów? (*.sln)
[ ] Jakie TFM są targetowane? (TargetFrameworkVersion w .csproj)
[ ] Stary format .csproj czy SDK-style?
[ ] Czy jest packages.config czy <PackageReference>?
[ ] Czy jest AssemblyInfo.cs? (stary format)
[ ] Jaka wersja C#? (LangVersion w .csproj — domyślnie brak = C# 5)
```

### Krok 2: Zależności zewnętrzne

```bash
# packages.config — wypisz wszystkie paczki
cat packages.config

# lub .csproj z PackageReference
grep -n "PackageReference" *.csproj
```

**Czerwone flagi** (stare, niezalecane pakiety):
- `Microsoft.AspNet.WebPages` < 3.x
- `EntityFramework` < 6.x
- `Newtonsoft.Json` < 12.x
- `log4net` (rozważ migrację do Serilog/NLog)
- `Unity` container (porzucony przez Microsoft)
- Paczki z ostatnim update przed 2018 r.

### Krok 3: Wzorce architektoniczne

**Pytania do ustalenia:**
1. Czy jest separacja warstw? (UI / BLL / DAL)
2. Czy są interfejsy dla serwisów? (mockability)
3. Czy logika biznesowa jest w kontrolerach? (antypattern)
4. Czy używany jest Repository Pattern?
5. Czy są Stored Procedures w SQL? Ile?
6. Czy jest kod z `HttpContext.Current` rozsiany po serwisach?

### Krok 4: Pokrycie testami

```
[ ] Czy istnieją projekty testowe?
[ ] Jaki framework? (MSTest / NUnit / xUnit)
[ ] Czy są testy integracyjne vs jednostkowe?
[ ] Czy mocki są używane? (Moq / NSubstitute / RhinoMocks)
[ ] Code coverage — czy jest mierzony?
```

---

## Typowe problemy do wykrycia

### Problemy z wątkami i stanem

```csharp
// PROBLEM: static state — nie-thread-safe
public static class CurrentUser
{
    public static string Name { get; set; } // ← NIEBEZPIECZNE w IIS!
}

// PROBLEM: HttpContext.Current w warstwie serwisowej
public class OrderService
{
    public void Process()
    {
        var userId = HttpContext.Current.User.Identity.Name; // ← coupling do warstwy web
    }
}
```

### Problemy z disposable i connection leaks

```csharp
// PROBLEM: brak using
SqlConnection conn = new SqlConnection(connStr);
conn.Open();
// ... brak conn.Dispose() jeśli rzuci wyjątek

// DOBRZE
using (var conn = new SqlConnection(connStr))
{
    conn.Open();
    // ...
}
```

### Problemy z konfiguracją

```csharp
// Stary styl — hardkodowane wartości lub magic strings
string conn = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
string apiKey = ConfigurationManager.AppSettings["ApiKey"];

// Sprawdź czy są transformacje web.config dla różnych środowisk:
// web.Debug.config, web.Release.config, web.Staging.config
```

---

## Narzędzia do analizy statycznej

| Narzędzie        | Zastosowanie                                   |
|------------------|------------------------------------------------|
| **NDepend**      | Metryki kodu, coupling, zależności cykliczne   |
| **ReSharper**    | Code smells, refaktoryzacja, dead code         |
| **Roslyn Analyzers** | Wbudowane w VS 2019+, reguły Roslyn        |
| **dotnet-upgrade-assistant** | Analiza gotowości do migracji      |
| **SonarQube**    | Security, bugs, code smells (CI/CD)            |

```bash
# dotnet upgrade assistant — analiza projektu
dotnet tool install -g upgrade-assistant
upgrade-assistant analyze MySolution.sln
```

---

## Wzorzec raportu z analizy

Gdy piszesz raport z analizy legacy codebase, użyj tej struktury:

```markdown
## Raport analizy — [Nazwa projektu]

### 1. Podsumowanie
- Wersja .NET: 4.8 / 4.72 / …
- Wersja C#: …
- Liczba projektów w solution: …
- Liczba zewnętrznych zależności: …

### 2. Architektura
[opis wzorców, warstw, komponentów]

### 3. Ryzyka techniczne
[lista problemów z priorytetem: Wysoki / Średni / Niski]

### 4. Pokrycie testami
[ocena jakości testów]

### 5. Rekomendacje
[konkretne kroki z szacowanym nakładem]
```
