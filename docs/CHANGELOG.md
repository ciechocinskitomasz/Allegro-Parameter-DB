# Historia zmian — Allegro Parameter DB

Dokument przedstawia osiągnięte etapy, a nie pełną historię commitów Git.

## 2026-10-14 — Zakończenie etapu bazy parametrów

### Dodano

- Odczyt kategorii z lokalnego `kategorie.csv` i deduplikację identyfikatorów bez zmiany danych źródłowych.
- Synchronizację parametrów kategorii przez Allegro API.
- Tryby synchronizacji: `--missing`, `--update`, `--update --resume`, `--category`, `--failed`.
- Rejestrowanie statusów, historii uruchomień i postępu poszczególnych kategorii.
- Ponawianie nieudanych zapytań oraz obsługę przerwania `Ctrl+C`.

### Zmieniono

- Znormalizowano słowniki parametrów i wprowadzono współdzielone zestawy wartości.
- Ograniczono duplikowanie danych w `raw_json`.
- Zmniejszono lokalny rozmiar bazy z około 205 MB do około 3 MB.

### Zweryfikowano

- 257 kategorii z zapisanymi parametrami.
- 3 429 rekordów parametrów, 4 332 unikalne wartości, 58 zestawów słownikowych.
- `PRAGMA integrity_check`: `ok`.
- Synchronizację testową kategorii `18711`: 19 parametrów, 4 054 wartości słownikowe, bez błędów.

### Dokumentacja

- Opis struktury bazy, poleceń synchronizatora i zasad bezpieczeństwa danych.

## Kolejne planowane prace

- Przeglądarka kategorii i parametrów.
- Wyszukiwanie parametrów oraz wartości słownikowych.
- Eksport danych do CSV.
- Dalsze testy odporności synchronizatora i walidacji danych.
