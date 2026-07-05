---
name: zb
description: Kompleksowa metodologia Zapasu Bezpieczeństwa (ZB) 2.0 — audyt i budowa kalkulatorów Excel dla portfeli do 500+ SKU, 16 metod ZB (King, DDMRP, Croston/SBA, Fill Rate/ESC, kwantyl empiryczny, King z σ_err), automatyczne drzewo wyboru metody, ROP/S-level, korekta fill rate dostawcy (LT_eff), profile dostawców Turcja/Chiny/Europa. Użyj ZAWSZE gdy użytkownik pyta o zapas bezpieczeństwa, safety stock, ZB, ROP, punkt zamawiania, DDMRP, Croston, SBA, popyt przerywany, King's Formula, fill rate, LT_eff, poziom obsługi CSL, klasyfikację ABC/XYZ dla zapasów — a także gdy przesyła plik Excel z obliczeniami zapasów do sprawdzenia ("przeanalizuj plik", "oceń metodę", "spójrz okiem profesora logistyki"), chce zbudować kalkulator ZB dla wielu produktów, pyta "ile trzymać na magazynie" / "kiedy zamawiać", lub ma dostawców z długim czasem dostawy.
---

# Zapas Bezpieczeństwa (ZB) 2.0 — audyt i budowa kalkulatorów

Następca metodologii `zb` 1.x. Wypracowany na realnym projekcie: portfel 907 SKU,
dostawcy z Turcji/Chin/Europy, plik Excel z 56 tys. formuł, sześć iteracji recenzji
profesorskiej. Zawiera nie tylko wzory, ale i katalog błędów, które NAPRAWDĘ
wystąpiły — sprawdzaj je w każdym cudzym pliku.

## Filozofia

ZB chroni przed dwoma niepewnościami naraz: zmiennością popytu i zmiennością
czasu dostawy (LT). Zanim policzysz wzór — zrozum dane. Trzy pytania otwierające:
1. Ile tygodni niezerowych ma historia? (n_nz < 3 → żadna statystyka nie jest uczciwa)
2. Kto generuje ryzyko: popyt czy dostawca? (porównaj składniki LT×σ_D² vs D̄²×σ_LT²)
3. Czy ZB kosztuje mniej niż 30% wartości rocznej SKU? (jeśli nie — problem jest
   biznesowy: VMI/konsygnacja/eliminacja, nie wzór)

## Tryby pracy — rozpoznaj, który dotyczy użytkownika

**A. AUDYT istniejącego pliku** — użytkownik przesyła Excel z obliczeniami ZB.
**B. BUDOWA kalkulatora** — użytkownik ma dane (historia tygodniowa + parametry
dostawców) i chce narzędzie dla portfela.
**C. POJEDYNCZY SKU / pytanie metodyczne** — szybka analiza lub wybór metody.

We wszystkich trybach: jednostki! Ustal na starcie, czy popyt jest dzienny czy
tygodniowy i czy LT jest w dniach czy tygodniach. Mieszanie poziomów czasu to
najczęstsze źródło ZB zawyżonego/zaniżonego o rząd wielkości. Konwencja: historia
i σ_D tygodniowo, LT w dniach → w formułach LT/7.

## Tryb A — Audyt: obowiązkowa checklista

Wczytaj plik programowo (openpyxl, data_only=False ORAZ data_only=True — formuły
i wartości osobno). Sprawdź KAŻDY punkt — to realne błędy z praktyki:

1. **Porównanie tekstu z liczbą w IF** — np. `IF(T3>1.32,...)` gdzie T3 to litera
   klasy XYZ. W Excelu tekst > liczba jest zawsze PRAWDĄ → warunek martwy.
   Objaw: jedna gałąź nigdy nie występuje w wynikach (np. 685×SBA, 0×Croston).
2. **Nieistniejące funkcje** — `_xludf.*` (artefakt generatorów), POISSON.INV
   (nie istnieje w żadnym Excelu, także 365 — używaj CRITBINOM/BINOM.INV).
   IFERROR maskuje takie błędy do 0 — po cichu!
3. **Unarny minus przed potęgą** — `EXP(-A1^2/2)` liczy EXP(+A1²/2), bo w Excelu
   `-A1^2 = (-A1)²`. Zawsze pisz `EXP(-(A1^2)/2)`.
4. **CV liczony z tygodniami zerowymi** — zawyża zmienność, wpycha wszystko do
   klasy Z. Klasyfikacja XYZ i kryterium Syntetos-Boylan: CV²_nz TYLKO z okresów
   niezerowych. σ_D do wzorów ZB: z pełnej historii (to poprawne).
5. **Niespójna korekta fill rate** — albo wszędzie LT_eff = LT/FR (podejście A,
   zalecane), albo nigdzie; ZB/FR (podejście B) nie może współistnieć z A.
   ROP też musi używać LT_eff, inaczej pipeline niedoszacowany dla słabych dostawców.
6. **ISBLANK na kolumnie z formułą** — formuła zwracająca 0/"" nigdy nie jest
   "blank". Bramy warunkowe buduj na wartościach (n_nz ≥ 8), nie na ISBLANK.
7. **Błąd propagacji w AND** — `AND(ISNUMBER(X), ABS(N-X)>0.1)` wybucha #VALUE!
   gdy X="" — Excel NIE skraca obliczeń w AND. Używaj zagnieżdżonych IF.
8. **Filtr-zombie** — autofiltr z zapisanym kryterium + wiersze ukryte na poziomie
   wiersza; usunięcie filtra NIE odkrywa wierszy. Napraw: wyczyść filterColumn
   w XML/openpyxl i ustaw row_dimensions.hidden=False.
9. **Dane wejściowe-defaulty** — σ_LT=0 przy LT>30 dni, identyczne FR dla całego
   portfela, LT>180 dni — wyglądają na niewypełnione pola, nie pomiary. Flaguj.
10. **Newsvendor/DDMRP w złej roli** — Newsvendor tylko jednookresowo (sezon/promo);
    DDMRP to polityka sterowania (bufory, ADU), nie kolejny wzór na ZB — nigdy
    w autoselekcji, zawsze jako flaga "kandydat" do decyzji eksperta.

Wynik audytu podawaj jak recenzent: błędy krytyczne / niespójności metodyczne /
jakość danych / mocne strony — z liczbami (ile SKU dotkniętych, wpływ na kapitał).

## Drzewo wyboru metody (AUTO) — serce metodologii

Kolejność sprawdzeń jest istotna (od dyskwalifikujących do domyślnych):

```
1. n_nz < 3                        → MANUAL/MTO (żadna statystyka; krytyczne: ZB≈1 partia ẑ,
                                     niekrytyczne: ZB=0, MTO/VMI)
2. p̂ > 1,32 tyg (przerywany):
   a. λ = D̄×LT_eff < 5 ORAZ ẑ ≤ 2  → M8 Poisson (popyt jednostkowy!)
   b. klasa Z, n_nz ≥ 8, LT_eff ≤ 13 tyg → M15 kwantyl empiryczny
   c. CV²_nz > 0,49                → M13 SBA
   d. inaczej                      → M12 Croston
   (M3/M4/M5/M7/M10/M11 ZABRONIONE dla przerywanego)
3. regularny, klasa C:
   a. wartość ZB > 30% wartości rocznej → MTO/VMI (koszt)
   b. inaczej                      → M2 (20% popytu w LT_eff)
4. regularny, segment AX/AY/BX/BY:
   a. σ_err dostępne i σ_err/σ_D ≤ 0,8 → M16 King z σ_err (prognoza tłumaczy strukturę)
   b. inaczej                      → M5 King's Formula z LT_eff
```

Progi i uzasadnienia każdej gałęzi + wzory wszystkich 16 metod:
**czytaj `references/metody_zb_16.md`**.

Kolumnę AUTO stawiaj OBOK ręcznego wyboru użytkownika, nie zamiast — z alertem,
gdy różnica ZB przekracza 20%. Automat proponuje, człowiek decyduje.

## Alerty jakości (zawsze przy portfelu)

- 🔴 n_nz < 3 → decyzja manualna
- ⚠️ metoda normalna przy popycie przerywanym
- ⚠️ M15 przycięty (LT_eff > 13 tyg przy 26 tyg historii)
- 🟡 pokrycie ZB poza 2–20 tygodni
- 🔴 kapitał: wartość ZB > 30% wartości rocznej → VMI/konsygnacja/eliminacja;
  🟡 20–30% → obniż SL / prostsza metoda / MTO
- ℹ️ kandydat DDMRP (LT > 30 dni, segment AX/AY)
- ℹ️ FR odbiega od benchmarku kraju > 10 p.p.
- Δ ZB_auto vs wybrane > 20% → przegląd

## Tryb B — Budowa kalkulatora

Strukturę arkuszy, układ kolumn A–BE, pasma sekcji, grupowanie, ochronę
i wszystkie pułapki techniczne openpyxl: **czytaj `references/struktura_v36.md`**.

Zasady nienegocjowalne:
- Żywe formuły Excel, nie wartości z Pythona (wyjątki: kwantyl empiryczny M15
  i σ_err — oznacz jako STATYCZNE z instrukcją przeliczenia).
- Po każdym zapisie: przelicz przez LibreOffice (skrypt recalc) i wymagaj
  **0 błędów formuł**. Ustaw fullCalcOnLoad=True.
- Waliduj liczbowo: przelicz ręcznie w Pythonie King's dla 2–3 SKU i porównaj
  co do sztuki; sprawdź rozkład metod AUTO i ogony pokrycia.
- Ochrona arkuszy bez hasła: edytowalne tylko komórki wejściowe; filtrowanie,
  sortowanie i formatowanie dozwolone.
- Wersjonuj plik (v3.x) i prowadź arkusz CHANGELOG — także z listą rzeczy
  świadomie NIEwdrożonych i dlaczego.

## Tryb C — pojedynczy SKU

Workflow jak w zb 1.x: analiza historii (CV z niezerowych!), wybór σ_LT
(PERT gdy tylko min/avg/max; MAD gdy ≥10 obserwacji), metoda z drzewa AUTO,
korekta FR przez LT_eff, ROP = D̄×(LT_eff+T_review)+ZB, walidacja z historią
(czy ZB bywał przekraczany; +20–30% marginesu ponad obserwowane maksimum).

## Zasady recenzji profesorskiej (gdy oceniasz decyzje)

- Poprawność proceduralna ≠ poprawność merytoryczna: reguły mogły zadziałać,
  a wynik i tak jest szumem (n_nz=1: p̂ to cenzura, nie pomiar; CV²=0 to artefakt).
- Przy popycie grudkowym (partie ~stała wielkość co kilka tygodni) sensowna
  polityka to "trzymaj ≈1 partię + odnawiaj" — sprawdź, czy to nie cykliczne
  zamówienie jednego klienta → wtedy awizacja/MTO zamiast zapasu.
- DDMRP vs King: DDMRP celowo używa nominalnego DLT (zawodność dostawcy →
  Var_Factor, wg Ptak-Smith), King jawnie wycenia σ_LT i FR oraz gwarantuje
  mierzalny CSL. Porównania liczbowe traktuj jako sanity-check (±15%), nie test.
- KPI: dla A/B regularnych MAPE/Bias/RMSE; dla klasy Z — fill rate i kapitał,
  nie dokładność prognozy. KPI liczone z prognozy naiwnej oznaczaj jako
  PRZYKŁADOWE — do podmiany na błędy prognozy z ERP.
- 26 tygodni historii = założenie quasi-stacjonarności; sezonowość wymaga ≥104.

## Pliki referencyjne

- `references/metody_zb_16.md` — wzory M1–M16 (Excel + interpretacja), progi
  decyzyjne, tablica G(z), bibliografia. Czytaj przy obliczeniach i audycie wzorów.
- `references/struktura_v36.md` — wzorcowa struktura pliku (13 arkuszy, kolumny
  A–BE), ustawienia ochrony, pułapki openpyxl/LibreOffice. Czytaj przy budowie
  lub naprawie pliku.
