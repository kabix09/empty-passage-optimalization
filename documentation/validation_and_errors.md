# Walidacja wyników i analiza błędów

## Opis problemu

Zdarza się, że DataFrame z nowymi zestawami tras (wynik optymalizacji) zawiera **mniej rekordów** niż wejściowy układ tras.
Taka sytuacja **wymaga walidacji**, ponieważ model powinien znaleźć rozwiązanie obejmujące **wszystkie przejazdy wejściowe**, dla których minimalizowane są puste przebiegi.

Jeżeli którakolwiek z tras wejściowych zostanie pominięta, wynik należy uznać za **nieprawidłowy**.

---

## Możliwe przyczyny rozbieżności

### 1. Łączenie tras w jedną pełną jazdę (agregacja)

Solver (np. Solera / CPLEX) może zredukować liczbę połączeń poprzez **połączenie kilku krótszych tras w jedną dłuższą trasę**.

**Przykład:**

- Wejście:  
  - A → B  
  - B → C  
- Po optymalizacji:  
  - A → C  

Jeżeli pusta droga między B i C została uwzględniona w jednym przejeździe, liczba tras w wyniku maleje, **ale ładunek nadal jest realizowany poprawnie**.

➡️ **Taki przypadek nie jest błędem**, pod warunkiem że wszystkie trasy wejściowe są logicznie odwzorowane w rozwiązaniu.

---

### 2. Pominięcie tras pustych

Solver może traktować **puste przejazdy** jako osobną kategorię i **nie uwzględniać ich w zestawieniu nowych połączeń**.

Jeżeli:

- wejściowe dane zawierają zarówno trasy pełne, jak i puste,
- a wynikowy DataFrame zawiera tylko trasy z ładunkiem,

to liczba rekordów w wyniku może być mniejsza niż liczba wejściowa.

➡️ **Nie jest to błąd**, o ile puste przejazdy są poprawnie rozliczone w innym zestawieniu lub metryce.

---

### 3. Trasy niezrealizowane lub odrzucone przez model ❗

W niektórych przypadkach solver może **odrzucić część tras**, np. z powodu:

- braku możliwego okna czasowego,
- konfliktów czasowych między trasami,
- ograniczeń floty lub pojazdów.

➡️ **Jest to błąd krytyczny**, jeśli którakolwiek trasa wejściowa:

- nie została przypisana do żadnego przewoźnika,
- i nie została uwzględniona w żadnej formie w rozwiązaniu.

---

## Walidacja ręczna (wymagana)

Należy zawsze sprawdzić:

1. Czy **wszystkie identyfikatory tras wejściowych** występują w danych wynikowych  
   (np. porównanie indexów `full_number`).
2. Czy brakujące trasy:
   - zostały połączone z innymi (agregacja), czy
   - zostały całkowicie pominięte.
3. Czy liczba tras pominiętych = 0.

Jeżeli **choć jedna trasa wejściowa nie występuje w rozwiązaniu**, wynik należy oznaczyć jako **niepoprawny**.

---

## Wniosek

Różnica w liczbie tras **nie zawsze oznacza błąd**, ale:

- każda trasa wejściowa **musi być możliwa do odtworzenia** w wyniku,
- brak jawnego przypisania trasy oznacza **nieprawidłowe rozwiązanie optymalizacji**.
