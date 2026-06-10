# Przekazanie systemu finansowego – notatki

## Kontekst systemu

System służy do zarządzania operacjami finansowymi. Obejmuje:

- Płatności masowe
- Mass collection
- Direct debit
- Zarządzanie klientami
- Księgowanie
- Raporty
- Integracje z systemami zewnętrznymi (EQ, Elixir)

-----

## Tematy do omówienia z analitykiem

### 1. Architektura i kontekst biznesowy

- Jaką rolę pełni system w ekosystemie firmy – czy to core systemu, czy satelita?
- Kto jest właścicielem biznesowym i kto podejmuje decyzje o zmianach?
- Jakie są plany rozwoju / znane długi techniczne?

### 2. Płatności masowe i mass collection

- Jaki jest cykl życia płatności – od inicjacji do rozliczenia?
- Jakie są statusy płatności i co wyzwala przejścia między nimi?
- Jakie są reguły biznesowe dla płatności odrzuconych, zwrotów, duplikatów?
- Czy są progi kwotowe, limity, wymagane autoryzacje?

### 3. Direct Debit

- Jak działa zarządzanie mandatami (tworzenie, modyfikacja, anulowanie)?
- Jaki jest harmonogram pobrań i jak obsługiwane są błędy (NSF, revoke)?
- Czy system obsługuje DD SEPA czy wyłącznie krajowe?

### 4. Integracje – EQ i Elixir

- Jak wygląda przepływ pliku z/do Elixiru (format MT, własny format, harmonogram sesji)?
- Co robi integracja z EQ – rozliczenia, raportowanie, obie?
- Jakie są scenariusze błędów po stronie integracji i jak są obsługiwane (retry, alerting, manual fix)?
- Czy integracje działają synchronicznie czy przez kolejkę/plik?

### 5. Księgowanie

- Jaki model księgowy jest stosowany – per transakcja, zbiorcze, dual-entry?
- Jak wygląda reconciliation – automatyczne czy manualne?
- Jakie są reguły dla korekt, storn, transakcji w toku?
- Czy system jest source of truth dla księgowości, czy synchronizuje się z innym systemem (ERP)?

### 6. Zarządzanie klientami

- Jakie dane klienta są istotne dla logiki płatności (segmenty, limity, zgody)?
- Jak działa onboarding/offboarding klienta w kontekście płatności?
- Czy są różne typy klientów z różnymi regułami?

### 7. Raporty

- Jakie raporty są krytyczne operacyjnie (dzienne, end-of-day, regulatoryjne)?
- Kto jest odbiorcą – operacje, finanse, compliance, zewnętrzne organy?
- Czy raporty są generowane automatycznie czy na żądanie?

### 8. Obsługa błędów i wyjątków

- Jakie są najczęstsze błędy produkcyjne i jak się je rozwiązuje?
- Czy są mechanizmy kompensacyjne (saga, manual override)?
- Kto ma dostęp do korekt manualnych i jak są auditowane?

### 9. Bezpieczeństwo i compliance

- Jakie regulacje obowiązują (KNF, PSD2, AML)?
- Jak działa autoryzacja operacji – role, 4-eyes principle?
- Gdzie są logi audytowe i jak długo są przechowywane?

### 10. Operacje i monitoring

- Jak wygląda typowy dzień operacyjny (sesje, cutoff, EOD)?
- Jakie są alerty i kto jest pierwszą linią wsparcia?
- Jakie masz SLA na przetwarzanie i dostępność systemu?

-----

## Plan spotkań kick-off (2 × 1h)

### Spotkanie 1 – „Biznes i przepływy”

|Czas     |Temat                                                          |
|---------|---------------------------------------------------------------|
|0:00–0:10|Kontekst systemu – rola w firmie, właściciel, historia, plany  |
|0:10–0:25|Cykl życia płatności – masowe, collection, direct debit        |
|0:25–0:40|Zarządzanie klientami – typy, reguły, onboarding               |
|0:40–0:55|Integracje EQ / Elixir – jak działają na co dzień, sesje, pliki|
|0:55–1:00|Pytania + ustalenie co analityk przygotuje na spotkanie 2      |

**Materiały do przygotowania przez analityka:**

- Diagram przepływu płatności
- Lista statusów płatności
- Schemat integracji

-----

### Spotkanie 2 – „Logika, błędy i operacje”

|Czas     |Temat                                                          |
|---------|---------------------------------------------------------------|
|0:00–0:15|Księgowanie – model, reconciliation, korekty, relacja z ERP/FK |
|0:15–0:30|Obsługa błędów – najczęstsze przypadki, manual fix, kompensacja|
|0:30–0:45|Raporty – krytyczne, harmonogram, odbiorcy                     |
|0:45–0:55|Bezpieczeństwo, role, compliance, monitoring i EOD             |
|0:55–1:00|Otwarte pytania + ustalenie dostępów, dokumentacji, kontaktów  |

**Materiały do przygotowania przez analityka:**

- Lista raportów z opisem
- Opis ról i uprawnień
- Znane problemy produkcyjne

-----

## Plan działania – pierwsze 2 miesiące

> Analityk dostępny na stałe. Deweloper dostępny przez 2 miesiące.

### Tygodnie 1–2 – intensywna eksploracja

- Spotkania z analitykiem **na żądanie**, krótko (15–30 min) gdy utkniesz
- Czytaj kod z deweloperem – poproś o **code walkthrough** kluczowych modułów (płatności, integracje)
- Deweloper jest najcenniejszy teraz – zna **dlaczego** coś jest zrobione tak a nie inaczej

### Tygodnie 3–8 – zanim deweloper odejdzie

Zaplanuj z nim dedykowane sesje na:

- [ ] Znane długi techniczne i “minowe pola” w kodzie
- [ ] Nieudokumentowane decyzje architektoniczne
- [ ] Scenariusze błędów których nigdy nie ma w dokumentacji
- [ ] Jak deployować i rollbackować – **krytyczne**

### Po odejściu dewelopera

- Analityk zostaje głównym źródłem wiedzy biznesowej
- Relacja z nim jest ważniejsza niż liczba spotkań

-----

## Kluczowe zasady

> **Dokumentuj wszystko co odkryjesz** – nie dla firmy, dla siebie.
> Za 6 miesięcy nie będziesz pamiętał dlaczego coś działa tak jak działa, a dewelopera już nie będzie.

- Nagrywaj spotkania (za zgodą uczestników)
- Zbieraj pytania na bieżąco podczas czytania kodu
- Ustal że przez pierwsze 2–3 tygodnie analityk jest dostępny na krótkie pytania
- Prowadź własne notatki/wiki

> Przy systemach finansowych z integracjami bankowymi **prawdziwe zrozumienie przychodzi po pierwszym incydencie produkcyjnym**, nie po spotkaniach.