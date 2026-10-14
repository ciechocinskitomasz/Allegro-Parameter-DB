# Synchronizacja parametrów Allegro

Stan dokumentacji: **14.10.2026**.

## Wymagania

- Lokalny kod aplikacji i środowisko Python `.venv`.
- Działające uwierzytelnianie do Allegro API.
- Plik `kategorie.csv` w **głównym katalogu projektu**, z kolumną `ID KATEGORII` i separatorem `;`.
- Dostęp do lokalnej bazy SQLite.

Skrypt odczytuje identyfikatory z CSV i usuwa duplikaty w pamięci. **Nie zmienia pliku CSV ani produktów.**

## Przygotowanie terminala (macOS)

```bash
cd ~/Documents/Allegro-Parameter-DB
source .venv/bin/activate
```

Polecenia uruchamiaj z głównego katalogu projektu, mimo że skrypt znajduje się w `app/services/`.

## Polecenia

| Polecenie | Zastosowanie |
| --- | --- |
| `python -m app.services.sync_all_parameters` | Domyślnie pobiera brakujące kategorie |
| `python -m app.services.sync_all_parameters --missing` | Pobiera tylko kategorie bez zapisanych parametrów |
| `python -m app.services.sync_all_parameters --update` | Odświeża wszystkie kategorie z CSV; tworzy nowe uruchomienie |
| `python -m app.services.sync_all_parameters --update --resume` | Wznawia ostatnie niedokończone uruchomienie `--update` |
| `python -m app.services.sync_all_parameters --category 18711` | Aktualizuje jedną wskazaną kategorię |
| `python -m app.services.sync_all_parameters --failed` | Ponawia kategorie oznaczone jako `failed` |
| `python -m app.services.sync_all_parameters --help` | Wyświetla pomoc bez synchronizacji |

### Przykłady

```bash
# Nowe lub dotychczas niepobrane kategorie
python -m app.services.sync_all_parameters --missing

# Pełne odświeżenie danych
python -m app.services.sync_all_parameters --update

# Wznowienie ostatniej przerwanej/nieudanej pełnej aktualizacji
python -m app.services.sync_all_parameters --update --resume

# Test jednej kategorii
python -m app.services.sync_all_parameters --category 18711

# Ponowienie błędów
python -m app.services.sync_all_parameters --failed
```

## Działanie i postęp

- Domyślnie skrypt podejmuje maksymalnie **3 próby** na kategorię.
- Pomiędzy kategoriami odczekuje **0,5 sekundy**.
- Po błędach odczekuje przed kolejną próbą (3 s, następnie 6 s).
- Wyniki zapisuje w `parameter_sync_status`; historię pełnych i innych uruchomień w `parameter_sync_runs` i `parameter_sync_run_items`.
- `Ctrl+C` przerywa synchronizację; poprawnie zapisane wcześniej kategorie pozostają w bazie.
- Po przerwaniu `--update` użyj `--update --resume`. Po przerwaniu `--missing` ponownie uruchom `--missing`.

### Ograniczenia obecnej wersji

- `--missing` wykrywa zsynchronizowane kategorie na podstawie istnienia rekordów w `parameters`. Kategoria, dla której API zwróci **zero parametrów**, może zostać pobrana ponownie.
- `--resume` dotyczy ostatniego niedokończonego uruchomienia `--update`, a nie trybów `--missing` czy `--failed`.
- `--failed` wybiera kategorie z ostatnim statusem `failed`, które nadal występują w aktualnym CSV.
- Tryb `--category` przyjmuje numery także spoza CSV.
- Wynik kontroli `PRAGMA integrity_check = ok` nie oznacza automatycznie, że wszystkie dane Allegro są aktualne.

## Podział odpowiedzialności

- `app/services/parameter_sync.py` — pobranie i zapis parametrów **jednej** kategorii.
- `app/services/sync_all_parameters.py` — wybór kategorii, próby, statusy, historia i wznawianie.

## Rozwiązywanie problemów

**`ModuleNotFoundError: No module named 'app'`** — sprawdź, czy terminal znajduje się w głównym katalogu projektu, i uruchamiaj przez `python -m ...`.

**`Brak kolumny 'ID KATEGORII'`** — sprawdź nagłówek CSV i separator `;`.

**Błędy autoryzacji** — sprawdź lokalną konfigurację poświadczeń API; nie umieszczaj sekretów w repozytorium.

**`ImportError ... partially initialized module`** — sprawdź, czy `parameter_sync.py` nie został przypadkowo zastąpiony zawartością `sync_all_parameters.py`.
