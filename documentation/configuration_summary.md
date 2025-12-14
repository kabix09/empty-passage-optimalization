# Raport: Testy konfiguracji modelu minimalizacji pustych przewozów

## Wprowadzenie
Celem testów było zbadanie wpływu różnych ustawień parametrów `max_idle_hours`, `min_empty_km` oraz `max_empty_km` na rozwiązanie modelu MIP dla minimalizacji pustych przewozów.

Parametry decyzyjne modelu:
- `max_idle_hours` – maksymalna liczba godzin postoju pojazdu,
- `min_empty_km` – minimalna odległość trasy pustej,
- `max_empty_km` – maksymalna odległość trasy pustej.

Status rozwiązania CPLEX:
- **Optimal** – rozwiązanie optymalne znalezione,
- **Infeasible** – brak rozwiązania spełniającego ograniczenia.

---

## Tabela wyników testów

| Nr testu | max_idle_hours | min_empty_km | max_empty_km | Status rozwiązania | Minimalna suma pustych km | Uwagi |
|----------|----------------|--------------|--------------|------------------|---------------------------|-------|
| 1 | 36 | 50 | 700 | Optimal | 29758.37 | Fill empty km → 29054 km |
| 2 | 18 | 50 | 400 | Infeasible | - | Brak rozwiązania |
| 3 | 18 | 50 | 700 | Infeasible | - | Brak rozwiązania |
| 4 | 20 | 10 | 700 | Optimal | 32547.42 | Fill empty km → 32547.42 km |
| 5 | 48 | 50 | 1200 | Optimal | 29537.61 | Fill empty route → 29537.61 km |

---

## Analiza wyników

1. **Wzrost `max_idle_hours`** pozwala na większą elastyczność w układaniu tras, co w testach 1 i 5 skutkuje znalezieniem rozwiązania optymalnego.
2. **Zbyt małe `max_idle_hours` i ograniczenie `max_empty_km`** (testy 2 i 3) prowadzi do braku możliwego rozwiązania – model staje się niespełnialny.
3. **Zmniejszenie `min_empty_km`** (test 4) pozwala uzyskać największą minimalną sumę pustych km (32547 km), co wskazuje na większą możliwość wypełnienia krótszych odcinków.
4. **Wypełnianie pustych tras (`fill empty km/route`)** wpływa na końcowy wynik, ale nie zmienia statusu rozwiązania.

---

## Wnioski

- Model jest wrażliwy na kombinacje parametrów `max_idle_hours` i `max_empty_km`.
- Zbyt rygorystyczne ograniczenia mogą prowadzić do sytuacji niespełnialnych.
- W praktyce warto testować kilka konfiguracji parametrów i stosować wypełnianie pustych tras, aby uzyskać rzeczywistą minimalizację pustych km.

