# Baza danych SQLite

Stan dokumentacji: **14.10.2026**. Baza działa lokalnie; nie jest częścią repozytorium dokumentacyjnego.

## Lokalizacja

macOS:

```text
~/Library/Application Support/AllegroParameterDB/database.sqlite
```

## Najważniejsze tabele

| Tabela | Klucz / rola |
| --- | --- |
| `categories` | Metadane kategorii (`category_id`, nazwa, rodzic, ścieżka) |
| `imported_categories` | Rejestr importowanych kategorii i statusów |
| `parameters` | Parametry przypisane do kategorii; klucz `(category_id, parameter_id)` |
| `dictionary_values` | Wartości słownikowe; klucz `value_id`, dodatkowo `parameter_id` i `value` |
| `dictionary_sets` | Unikalne zestawy wartości; klucz `set_id`, dodatkowo `parameter_id` |
| `dictionary_set_values` | Relacja zestaw–wartość; klucz `(set_id, value_id)` |
| `category_parameter_sets` | Przypisanie zestawu do parametru w kategorii; klucz `(category_id, parameter_id)` |
| `parameter_sync_status` | Ostatni wynik synchronizacji kategorii |
| `parameter_sync_runs` | Historia uruchomień synchronizatora |
| `parameter_sync_run_items` | Status poszczególnych kategorii w danym uruchomieniu |

## Przechowywanie parametrów

Tabela `parameters` zawiera m.in. identyfikator kategorii i parametru, nazwę, typ, informacje o wymaganiu wartości, ograniczeniach i znacznik ostatniej synchronizacji. Pole `raw_json` przechowuje metadane parametru **bez rozbudowanej listy `dictionary`**, dzięki czemu nie duplikuje słowników.

## Normalizacja słowników

1. Wartość słownikowa jest zapisywana w `dictionary_values`.
2. Zestaw dopuszczalnych identyfikatorów wartości otrzymuje stabilny `set_id`.
3. `dictionary_set_values` łączy zestaw z jego wartościami.
4. `category_parameter_sets` wskazuje, który zestaw obowiązuje dla konkretnej pary kategoria–parametr.

Identyfikator zestawu to SHA-256 tekstu utworzonego z `parameter_id`, znaku nowej linii i posortowanych identyfikatorów wartości połączonych znakami nowej linii. Identyczne zestawy dla tego samego parametru mogą być współdzielone między kategoriami. Parametry bez wartości słownikowych nie muszą mieć wpisu w `category_parameter_sets`.

### Odczyt wartości parametru w kategorii

```sql
SELECT dv.value_id, dv.value
FROM category_parameter_sets AS cps
JOIN dictionary_set_values AS dsv ON dsv.set_id = cps.set_id
JOIN dictionary_values AS dv ON dv.value_id = dsv.value_id
WHERE cps.category_id = '18711'
  AND cps.parameter_id = 'TU_WSTAW_ID_PARAMETRU'
ORDER BY dv.value;
```

## Potwierdzone statystyki

| Tabela | Rekordy |
| --- | ---: |
| `parameters` | 3 429 |
| `dictionary_values` | 4 332 |
| `dictionary_sets` | 58 |
| `dictionary_set_values` | 4 421 |
| `category_parameter_sets` | 1 763 |

257 kategorii ma zapisane parametry. Ostatni zgłoszony wynik `PRAGMA integrity_check`: `ok`.

## Kontrola integralności

W kliencie SQLite:

```sql
PRAGMA integrity_check;
SELECT COUNT(DISTINCT category_id) FROM parameters;
SELECT COUNT(*) FROM parameters;
SELECT COUNT(*) FROM dictionary_values;
SELECT COUNT(*) FROM dictionary_sets;
SELECT COUNT(*) FROM dictionary_set_values;
SELECT COUNT(*) FROM category_parameter_sets;
```

`integrity_check` weryfikuje spójność strukturalną SQLite; nie zastępuje kontroli semantycznej danych API.

## Historia optymalizacji

Pierwotny sposób przechowywania powielał duże słowniki pomiędzy kategoriami. Po migracji do współdzielonych zestawów, usunięciu nieużywanych tabel historycznych i wykonaniu `VACUUM` lokalny plik bazy zmniejszył się z około 205 MB do około 3 MB. Przed migracją wykonano kopię zapasową; **kopie i plik bazy pozostają lokalne**.
