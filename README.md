# Allegro Parameter DB

Lokalny projekt do pobierania, przechowywania i aktualizowania parametrów kategorii Allegro za pośrednictwem API. Dokumentacja opisuje stan projektu z **14 października 2026 r.**

> **Status:** działająca synchronizacja parametrów i zoptymalizowana baza SQLite. Interfejs graficzny jest planowany. Repozytorium może zawierać wyłącznie dokumentację; kod, lokalna baza danych i dane produktów nie są częścią tej paczki.

## Co już działa

- Pobieranie parametrów kategorii z Allegro API z użyciem uwierzytelniania OAuth.
- Odczyt identyfikatorów kategorii z lokalnego pliku `kategorie.csv` i eliminacja duplikatów **w pamięci**.
- Zapisywanie parametrów, ograniczeń i dozwolonych wartości do SQLite.
- Współdzielenie identycznych zestawów wartości słownikowych między kategoriami.
- Synchronizacja brakujących kategorii, aktualizacja wszystkich, wznowienie aktualizacji, ponowienie błędów i synchronizacja jednej kategorii.
- Zapisywanie historii uruchomień i statusów.

## Stan danych (14.10.2026)

| Metryka | Wartość |
| --- | ---: |
| Unikalne kategorie w lokalnym CSV | 257 |
| Kategorie z zapisanymi parametrami | 257 |
| Rekordy parametrów | 3 429 |
| Unikalne wartości słownikowe | 4 332 |
| Współdzielone zestawy słowników | 58 |
| Relacje zestaw–wartość | 4 421 |
| Relacje kategoria–parametr–zestaw | 1 763 |
| `PRAGMA integrity_check` | `ok` |

Po normalizacji i oczyszczeniu baza zmniejszyła się z około **205 MB do około 3 MB** (wartości orientacyjne z lokalnych pomiarów).

## Uruchamianie (lokalna instalacja projektu)

Przykłady zakładają macOS, katalog projektu `~/Documents/Allegro-Parameter-DB`, aktywne środowisko `.venv`, poprawnie skonfigurowane dane API Allegro i dostępny lokalnie kod aplikacji.

```bash
cd ~/Documents/Allegro-Parameter-DB
source .venv/bin/activate
python -m app.services.sync_all_parameters --help
python -m app.services.sync_all_parameters --missing
```

Pozostałe polecenia i zachowanie trybów: [docs/SYNCHRONIZATION.md](docs/SYNCHRONIZATION.md).

## Dokumentacja

- [Struktura bazy danych](docs/DATABASE.md)
- [Synchronizacja — instrukcja i polecenia](docs/SYNCHRONIZATION.md)
- [Historia zmian](docs/CHANGELOG.md)

## Bezpieczeństwo i dane

Nie publikuj identyfikatorów i sekretów OAuth, tokenów, plików `.env`, prywatnych plików CSV ani bazy SQLite. Ta paczka zawiera **wyłącznie pliki Markdown**. Plik `kategorie.csv` nie jest dołączony.

## Planowane kierunki rozwoju

Przeglądarka kategorii i parametrów, wyszukiwanie parametrów pomiędzy kategoriami, podgląd wartości słownikowych oraz eksport danych do CSV.
