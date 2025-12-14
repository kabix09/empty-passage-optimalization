# Ewaluacja rozwiązań modelu trasowania ciągników

Poniżej znajduje się chronologiczny opis ewaluacji i rozwoju modelu, wraz z głównymi etapami i problemami, które zostały zidentyfikowane w trakcie ewaluacji rozwiązania.

---

## 1. Początkowe rozwiązania i obserwacje

- Pierwotny model skupiał się tylko na przypisaniu każdej trasy do dokładnie jednego traktora (`y[(t,r)] == 1`) i opcjonalnych pustych przejazdach (`x[(t, fr, to)]`).
- Problem: solver mógł ustawić wszystkie `x = 0`, bo nie było wymuszenia, że jeśli traktor wykonuje trasę, to musi mieć poprzedni pusty dojazd lub start.
- Konsekwencja: model dawał "0 km pustych", co nie odpowiadało rzeczywistości flotowej.

### Wnioski
- Potrzebne są dodatkowe ograniczenia przepływu, aby wymusić realistyczne sekwencje tras.

---

## 2. Chain maximalization (pierwszy wariant)

- Wprowadzono nagrodę `CHAIN_REWARD` (duża wartość np. 10 000) w funkcji celu: minimalizacja `(empty_km - CHAIN_REWARD) * x`.
- Cel: zachęcić solver do tworzenia jak największej liczby połączonych tras.

### Obserwacje
- Solver tworzył wiele chainów, ale sumaryczne puste km były bardzo duże (np. 140 tys. km vs oczekiwane 30 tys.).
- Problem: model maksymalizował liczbę chainów kosztem minimalizacji pustych km.

### Wnioski
- Nagroda `CHAIN_REWARD` była zbyt duża w stosunku do maksymalnych dopuszczalnych pustych przejazdów.
- Konieczne było wprowadzenie mechanizmu kontrolującego minimalizację pustych km.

---

## 3. Wariant z EPS (soft minimalization)

- Zamiast dużej nagrody, wprowadzono małe EPS (0.001) w funkcji celu: `empty_km * x - EPS * x`.
- Cel: dwupoziomowy, soft:
  - Krok A: maksymalizacja liczby chainów (EPS)
  - Krok B: minimalizacja pustych km (hard)

### Obserwacje
- Problem EPS był tylko "tie-breakerem", ale bez wymuszenia startów (`s`) solver nadal mógł ustawić `x = 0`.
- Model nie dawał realistycznych wyników.

### Wnioski
- EPS nie był skuteczny w warunkach, gdzie puste dojazdy nie były wymuszone.
- Potrzebne było wprowadzenie zmiennej `s[(t,r)]` oraz constraintu startu albo pustego dojazdu.

---

## 4. Single-stage realistic model (obecny)

- Wprowadzono zmienną `s[(t,r)]`:
  - `s = 1` jeśli traktor zaczyna od trasy r
- Constraint: `s + Σ x_incoming == y` wymusza:
  - każda trasa jest albo startem, albo ma dokładnie jeden pusty dojazd
- Funkcja celu: minimalizacja pustych km `Σ empty_km * x` (bez EPS)
- Dodatkowe ograniczenia:
  - Każda trasa przypisana dokładnie do jednego traktora (`Σ_t y == 1`)
  - Każdy traktor może zacząć tylko raz (`Σ_r s <= 1`)
  - Flow conservation: maksymalnie jeden pusty wyjazd z trasy
  - Big-M dla ograniczeń czasowych

### Efekty
- Model stał się w pełni realistyczny:
  - każda trasa wykonana
  - każdy traktor zaczyna maksymalnie raz
  - solver MUSI używać pustych przejazdów, minimalizując ich sumaryczne km
- EPS i CHAIN_REWARD nie są już potrzebne.

### Podsumowanie ewaluacji

| Etap | Cel | Problem | Wniosek |
|------|-----|--------|---------|
| 1. Początkowe | Przypisanie tras | x = 0, brak realistycznych chainów | Potrzebne ograniczenia przepływu |
| 2. Chain maximalization | Zachęta do chainów | Sumaryczne puste km za duże | Nagroda zbyt duża, wymaga kontrolowania pustych km |
| 3. EPS / soft | Tie-breaker dla chainów | x nadal może być 0 | EPS nie wystarcza bez zmiennej startu |
| 4. Single-stage | Minimalizacja pustych km + start | Realistyczne chainy, minimalizacja pustych km | Obecny finalny model, EPS i nagroda nie są potrzebne |

---

## Wnioski końcowe

- Obecny model jest **produkcyjnie gotowy**:
  - minimalizuje puste km
  - wymusza realistyczne sekwencje tras
  - eliminuje nienaturalne rozwiązania typu x=0 lub teleportacje
- Dalsza optymalizacja:
  - ograniczenie liczby kandydatów (`empty_candidates_df`) dla szybkości solve
  - ewentualne zmniejszenie M w Big-M dla stabilności
  - ewaluacja teoretycznego lower bound pustych km

