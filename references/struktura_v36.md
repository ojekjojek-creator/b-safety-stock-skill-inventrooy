# Wzorcowa struktura kalkulatora ZB (v3.6) + pułapki techniczne

Spis: [Arkusze](#arkusze) | [WYNIKI kolumny](#kolumny) | [Porządek wizualny](#wizual) |
[Ochrona](#ochrona) | [Pułapki openpyxl/LibreOffice](#pulapki) | [Workflow budowy](#workflow)

## Arkusze (13) {#arkusze}
1. **CHANGELOG** (pierwszy!) — historia wersji + decyzje świadomie NIEwdrożone z uzasadnieniem
2. **INSTRUKCJA** — workflow 8 kroków, legenda kolorów (niebieskie=wejście, szare=formuła)
3. **PARAMETRY** — kalkulator 1 SKU, jednostki DZIENNE (ostrzeżenie w nagłówku!),
   sekcje: popyt, LT, CSL (z=NORMSINV), profil dostawcy (FR, LT_eff), DDMRP,
   Periodic Review, Newsvendor, SRL, Croston, FR_cel A/B/C dla M14
4. **METODY_ZB** — 16 metod z żywymi formułami + segment + uwagi
5. **DDMRP_PELNY** — 3 strefy, TOG/TOY/TOR + adnotacja "nominalny DLT celowo"
6. **CROSTON_SBA** — metody 12–13 z kryterium CV²
7. **MACIERZ_ABC_XYZ** — 3×3, rekomendowana metoda + SL per segment
8. **FILL_RATE** — benchmarki krajów, kalkulator LT_eff, Periodic Review S
9. **WALIDACJA** — 4 testy: pokrycie, dominujący składnik, realność (pojemność/kapitał), FR
10. **DANE_WEJSCIOWE** — do 500+ SKU: kod, nazwa, cena, LT, σ_LT, LT_max, FR,
    T01–T26 (historia tygodniowa), kraj (lista rozwijana) + FR/σ_LT sugerowane (VLOOKUP z KRAJE)
11. **WYNIKI_500** — silnik (patrz niżej)
12. **BIBLIOGRAFIA**; 13. **Z_TABLE** (ukryty, G(z)); 14. **KRAJE** (benchmarki + disclaimer
    "full-truck, bez peak season"); 15. **KPI_PROGNOZY** (MAPE/Bias/MAE/RMSE per SKU,
    oznaczone PRZYKŁADOWE gdy z prognozy naiwnej)

## WYNIKI — układ kolumn (drabina zależności) {#kolumny}
| Blok | Kolumny | Zawartość |
|---|---|---|
| PRODUKT | A–D | nr, kod, nazwa, cena |
| POPYT | E–K | D̄, σ_D, CV, D_max, ẑ, p̂, Przerywany? (p̂>1,32) |
| DOSTAWCA | L–P | LT, σ_LT, FR, LT_eff[tyg]=L/(7·FR), σ_LT[tyg] |
| KLASYFIKACJA | Q–V | wart. roczna, kum% (SUMPRODUCT-ranking), ABC, XYZ (z CV_nz!), segment, z-score |
| DECYZJA | W–AC | metoda wybrana, ZB, ROP=D̄·LT_eff+ZB, S-level=D̄·(LT_eff+T/7)+ZB, pokrycie, wartość ZB, priorytet |
| METODY M1–M12 | AD–AO | kolumny porównawcze (zwinięte) |
| KALIBRACJA DDMRP | AP–AS | LTF_cal, VF_cal, ZB_DDMRP_cal, status ±15% vs ZB (sanity-check, nie test!) |
| POMOCNICZE | AT–AW | σ_nz, CV²_nz, σ_LTD, σ_err (statyczna, zwinięte) |
| NOWE MODELE | AX–AZ | M14 ESC, M15 empiryczny (statyczny), M16 King σ_err |
| ★ AUTO | BA–BE | n_nz (COUNTIF>0), Metoda AUTO, ZB AUTO, Alerty, FR_cel nadpisanie |

Formuły wzorcowe (wiersz 3 ↔ DANE wiersz 5):
- σ_nz: `SQRT(MAX(0,(SUMPRODUCT((rng>0)*rng^2)-n_nz*ẑ^2)/MAX(1,n_nz-1)))`
- kum% ABC bez sortowania: `(SUMPRODUCT((Q>Qi)*(B<>"")*Q)+Qi)/SUMPRODUCT((B<>"")*Q)`
- Metoda AUTO i Alerty: pełne drzewo w SKILL.md; FR-vs-benchmark przez zagnieżdżone IF
  (NIE w AND — patrz pułapki)
- Komentarze (openpyxl Comment) przy nagłówkach BB/BD/AS: interpretacja MANUAL/MTO,
  kolejność działań na flagach, charakter statusu DDMRP

## Porządek wizualny {#wizual}
- Wiersz 1: scalone pasma sekcji z pastelowymi wypełnieniami (po unmerge starego tytułu)
- Grupowanie kolumn: `ws.column_dimensions.group('AD','AS',hidden=True)` — bloki
  porównawcze zwinięte, przyciski +/− dla użytkownika
- Freeze panes na pierwszych kolumnach; autofiltr na PEŁNĄ szerokość (A2:BE…),
  bez zapisanych kryteriów; wszystkie wiersze odkryte

## Ochrona {#ochrona}
Wszystkie arkusze `protection.sheet=True` BEZ hasła; odblokowane (locked=False)
wyłącznie wejścia: DANE B:AI, PARAMETRY komórki nie-formułowe w C, WYNIKI kolumna
FR_cel, WALIDACJA benchmarki, KRAJE wartości. Zezwól: autoFilter=False, sort=False,
formatColumns/Rows/Cells=False (w openpyxl False = DOZWOLONE). Hasło w pliku
z changelogiem to iluzja — kontrola wersji jest w organizacji, nie w Excelu.

## Pułapki openpyxl / LibreOffice {#pulapki}
1. read_only=True + losowy dostęp .cell() = O(n) na komórkę → minuty zamiast sekund.
   Normalny load + iter_rows: cały plik w ~2 s.
2. Po zapisie openpyxl brak cached values → `wb.calculation.fullCalcOnLoad=True`
   i przelicz LibreOffice (skrypt recalc z skilla xlsx); wymagaj 0 błędów.
3. Skaner błędów łapie literalne "#NAME?" w TEKŚCIE komórki — nie pisz nazw błędów
   w opisach changeloga.
4. MergedCell: zapis do środka scalonego zakresu = AttributeError; pisz do kotwicy
   albo unmerge.
5. Funkcje 2010+: zapisuj przez prefiks _xlfn albo używaj legacy (NORMSDIST,
   NORMSINV, CRITBINOM) — legacy działa wszędzie.
6. Emoji w nagłówkach/kryteriach filtrów działają, ale zapisany filterColumn
   z emoji + ukryte wiersze = filtr-zombie (patrz checklista audytu).
7. Walidacja list (DataValidation) inline: `'"A,B,C"'` — bez przecinków w nazwach opcji.

## Workflow budowy {#workflow}
1. Wczytaj dane → oceń jakość (σ_LT=0? FR identyczne? LT>180?) → zaraportuj
2. Zbuduj arkusze z żywymi formułami; statyczne tylko M15/σ_err (oznacz!)
3. recalc → 0 błędów → walidacja liczbowa w Pythonie (King co do sztuki dla 2–3 SKU)
4. Rozkład metod AUTO + ogony pokrycia + suma alertów kapitałowych → raport
5. Ochrona, pasma, grupy, autofiltr → recalc ponownie → publikacja + CHANGELOG
