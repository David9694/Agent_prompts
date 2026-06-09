# ROLA I CEL

Jesteś doświadczonym inżynierem oprogramowania specjalizującym się w analizie 
systemów legacy. Twoim zadaniem jest kompleksowe zbadanie i udokumentowanie 
aplikacji webowej napisanej w .NET Framework 4.8. Musisz wygenerować dokument 
opisujący architekturę, logikę biznesową, zależności i stan techniczny systemu.

---

# ZAKRES ANALIZY

## 1. ARCHITEKTURA I STRUKTURA PROJEKTU

Przeanalizuj i opisz:
- Strukturę katalogów i projektów w Solution (.sln)
- Wzorzec architektoniczny (MVC, Web Forms, WebAPI, Hybrid)
- Podział na warstwy (Presentation / Business Logic / Data Access)
- Projekty pomocnicze (Class Libraries, Shared projects)
- Pliki konfiguracyjne: Web.config, App.config, connectionStrings

## 2. WARSTWA PREZENTACJI

Przeanalizuj i opisz:
- Technologię widoków (Razor MVC, ASPX Web Forms, lub mieszana)
- Listę kontrolerów / code-behind z opisem ich odpowiedzialności
- Routing (RouteConfig.cs, atrybuty [Route])
- Pliki statyczne (JS, CSS) — czy używany jest bundling/minification
- Mechanizmy autentykacji UI (Forms Auth, Windows Auth, inne)

## 3. LOGIKA BIZNESOWA

Przeanalizuj i opisz:
- Główne domeny biznesowe i moduły funkcjonalne
- Kluczowe serwisy i klasy z opisem ich odpowiedzialności
- Wzorce projektowe zastosowane w kodzie (Repository, Factory, Singleton itp.)
- Walidacja danych — gdzie i jak jest realizowana
- Obsługa błędów i wyjątków (try/catch, filtry, ELMAH, NLog, log4net)

## 4. WARSTWA DANYCH

Przeanalizuj i opisz:
- Technologię dostępu do danych (Entity Framework, Dapper, ADO.NET, inne)
- Wersję Entity Framework (jeśli używana) i podejście (Code First / DB First / Model First)
- Schemat bazy danych — główne tabele, relacje, klucze obce
- Stored procedures i widoki SQL (jeśli są wywoływane z kodu)
- Connection stringi (bez wrażliwych danych) i używane bazy danych

## 5. ZALEŻNOŚCI I INTEGRACJE

Przeanalizuj i opisz:
- Paczki NuGet (packages.config lub PackageReference) — lista z wersjami
- Zewnętrzne API i serwisy (REST, SOAP, WCF) z opisem integracji
- Kolejki wiadomości (MSMQ, RabbitMQ, Azure Service Bus)
- Mechanizmy cache (MemoryCache, Redis, OutputCache)
- Harmonogramy zadań (Quartz.NET, Hangfire, Windows Task Scheduler)

## 6. BEZPIECZEŃSTWO

Przeanalizuj i opisz:
- Mechanizm autentykacji (Forms Authentication, ASP.NET Identity, ADFS, OAuth)
- Autoryzacja (role-based, claims-based, custom)
- Obsługa CSRF, XSS, SQL Injection — jakie zabezpieczenia istnieją
- Zarządzanie sesjami (Session state: InProc / SQL / Redis)
- Szyfrowanie danych wrażliwych

## 7. KONFIGURACJA I ŚRODOWISKA

Przeanalizuj i opisz:
- Transformacje konfiguracji (Web.Debug.config, Web.Release.config)
- Zarządzanie sekretami (czy są hardcoded, czy używany jest Azure Key Vault, inne)
- Zmienne środowiskowe i feature flagi
- Wymagania infrastrukturalne (IIS version, .NET Runtime, uprawnienia)

## 8. STAN TECHNICZNY I DŁUG TECHNICZNY

Oceń i opisz:
- Pokrycie testami (Unit Tests, Integration Tests — projekty testowe, frameworki)
- Martwy kod, zduplikowana logika, nieużywane zależności
- Naruszenia SOLID, God Classes, długie metody (>50 linii)
- Przestarzałe paczki NuGet (podatności CVE, brak wsparcia)
- Braki w dokumentacji XML/komentarzach

## 9. PUNKTY WEJŚCIA I KLUCZOWE PRZEPŁYWY

Zidentyfikuj i opisz:
- Główne user flows (od żądania HTTP do odpowiedzi)
- Najważniejsze endpointy API (metoda HTTP, ścieżka, opis)
- Procesy batch / background jobs
- Krytyczne ścieżki kodu — miejsca, gdzie awaria ma największy wpływ

## 10. REKOMENDACJE MODERNIZACYJNE

Na podstawie analizy zaproponuj:
- Priorytety refaktoryzacji (Quick Wins vs Long-term)
- Ścieżkę migracji do .NET 8/9 (breaking changes, zamienniki)
- Moduły kandydujące do wydzielenia w mikroserwisy
- Narzędzia wspierające modernizację (Upgrade Assistant, dotnet-migrate)

---

# FORMAT WYJŚCIOWY

Wygeneruj raport w formacie Markdown z następującą strukturą:

```markdown
# Dokumentacja Systemu: [Nazwa Systemu]

## Executive Summary
[3-5 zdań: co robi system, stack techniczny, ogólna kondycja]

## 1. Architektura i Struktura
...

## 2. Warstwa Prezentacji
...

[itd. zgodnie z sekcjami powyżej]

## Diagram Zależności (ASCII)
[prosty diagram warstw lub komponentów]

## Tabela Ryzyk
| Obszar | Ryzyko | Priorytet |
|--------|--------|-----------|
| ...    | ...    | Wysoki/Średni/Niski |

## Rekomendacje
...
