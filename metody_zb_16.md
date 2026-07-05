# 16 metod Zapasu Bezpieczeństwa — wzory, progi, interpretacja

Spis: [Notacja](#notacja) | [M1–M2 proste](#m1m2) | [M3–M5 normalne](#m3m5) |
[M6–M8 odporne](#m6m8) | [M9–M11 specjalne](#m9m11) | [M12–M13 przerywane](#m12m13) |
[M14–M16 zaawansowane](#m14m16) | [Progi decyzyjne](#progi) | [G(z)](#gz) | [Bibliografia](#biblio)

## Notacja {#notacja}
D̄ — średni popyt/tydz (z pełnej historii, z zerami); σ_D — odch. std popytu/tydz;
ẑ — śr. wielkość sprzedaży w tygodniach niezerowych; p̂ = liczba_tygodni / n_nz;
CV²_nz = (σ_nz/ẑ)² z tygodni NIEZEROWYCH; LT — czas dostawy [dni]; σ_LT [dni];
FR — fill rate dostawcy; LT_eff = LT/(7·FR) [tyg]; σ_LTw = σ_LT/7 [tyg];
z — kwantyl normalny dla CSL (NORMSINV); T — okres przeglądu [dni].
σ_LTD = √(LT_eff·σ_D² + D̄²·σ_LTw²) — odch. popytu w czasie dostawy.

## M1–M2: proste (klasa C) {#m1m2}
**M1 Heurystyczna**: ZB = stała (doświadczenie). Tylko C; zero podstaw statystycznych.
**M2 % popytu**: ZB = D̄ × LT_eff × PCT (A:30%, B:20%, C:10–20%). Nie uwzględnia zmienności.

## M3–M5: rodzina normalna (popyt regularny!) {#m3m5}
**M3 zmienny popyt, LT stały**: ZB = z·σ_D·√LT_eff. Niedoszacowuje gdy σ_LT>0.
**M4 popyt stały, LT zmienny**: ZB = z·D̄·σ_LTw. Tylko CV<0,1.
**M5 King's Formula ★**: ZB = z·√(LT_eff·σ_D² + D̄²·σ_LTw²) = z·σ_LTD.
Złoty standard dla AX/AY/BX/BY. Excel:
`=ROUND(Z*SQRT(LT_eff*σD^2+D^2*σLTw^2),0)`.
Diagnoza składników: A=LT_eff·σ_D² (popyt), B=D̄²·σ_LTw² (dostawca); B/(A+B)>70%
→ rozmawiaj z dostawcą, nie z prognostą. Klasyczna wersja pomija składnik
interakcji σ_D²·σ_LTw² (niedoszacowanie 3–10% — dopuszczalne, odnotuj).

## M6–M8: odporne / liczydłowe {#m6m8}
**M6 Max-Avg**: ZB = D_max·LT_max/7 − D̄·LT/7. Bez podstaw stat., zwykle mocno
przeszacowuje — tylko jako kolumna ostrzegawcza, nie do autoselekcji.
**M7 MAD**: ZB = z·1,25·MAD·√LT_eff. Odporna na outliery (1,25=σ/MAD dla normalnego).
**M8 Poisson**: λ = D̄·LT_eff. λ≥5: ZB = z·√λ. λ<5: ZB = CRITBINOM(100000; λ/100000; CSL) − λ.
UWAGA: POISSON.INV nie istnieje w Excelu (także 365) — tylko CRITBINOM/BINOM.INV.
Warunek sensowności: popyt JEDNOSTKOWY (ẑ ≤ 2 szt). Przy partiach ẑ≈20 Poisson
drastycznie zaniża.

## M9–M11: specjalne {#m9m11}
**M9 Square Root Law**: ZB_new = ZB_base·√(N_after/N_before). Metoda PORTFELOWA
(konsolidacja magazynów) — bez sensu per-SKU.
**M10 Newsvendor**: ZB = NORMSINV(CR)·σ_D·√LT_eff, CR = C_short/(C_short+C_hold).
JEDNOOKRESOWY (sezon, promo, kolekcja) — nie do SKU odnawialnych.
**M11 DDMRP Red Zone**: ZB = D̄·(LT/7)·LTF·(1+VF). Nominalny DLT (celowo bez LT_eff
— zawodność dostawcy → VF, wg Ptak-Smith). Pełny bufor = Green(max(D̄·T; D̄·DLT·LTF))
+ Yellow(D̄·DLT) + Red; TOR=Red, ROP=TOY. To POLITYKA sterowania, nie wzór —
w tabeli tylko jako porównanie + flaga "kandydat DDMRP" (LT>30 dni, AX/AY).
Kalibracja odwrotna do ZB statystycznego: LTF = min(1, max(0,2; 0,5·(1+σ_LT/LT))),
VF = ZB/(D̄·LT/7)/LTF − 1 ograniczone [0;2]. VF na limicie 2 = SKU poza profilem DDMRP.

## M12–M13: popyt przerywany (p̂ > 1,32 tyg) {#m12m13}
**M12 Croston**: prognoza = ẑ/p̂; ZB = z·√(LT_eff/p̂ · 2·ẑ²). ROP = (ẑ/p̂)·LT_eff + ZB.
**M13 SBA (Syntetos-Boylan)**: prognoza = (1−α/2)·ẑ/p̂ (α≈0,1); ZB jak Croston;
ROP ze skorygowaną stopą. Wybór: CV²_nz > 0,49 → SBA; ≤ 0,49 → Croston.
KRYTYCZNE: kryterium na CV²_nz z WIELKOŚCI niezerowych — nie na literze XYZ,
nie na CV z zerami. Wzór ZB to heurystyka (√(2ẑ²·LT/p̂) bez oparcia w literaturze)
— dla klasy Z preferuj M15, gdy brama pozwala.

## M14–M16: zaawansowane {#m14m16}
**M14 Fill Rate / ESC**: cel = P2 (fill rate), nie CSL. ESC = (1−FR_cel)·Q,
Q = D̄·T/7. G_req = ESC/σ_LTD; z z odwróconej funkcji straty (tablica G(z));
ZB = z·σ_LTD. Przy dużych Q często NIŻSZE niż CSL 97% — realna oszczędność
kapitału dla klasy A. FR_cel per klasa (A 99%, B 97%, C 95%) + nadpisanie per SKU.
**M15 Kwantyl empiryczny (Willemain)**: okno w = round(LT_eff) tyg (cap: połowa
historii); sumy kroczące popytu w oknie; ZB = kwantyl_CSL(sumy) − w·D̄, min 0.
Nieparametryczny — najlepszy dla klasy Z. Brama: n_nz ≥ 8 i LT_eff ≤ 13 tyg
(przy 26 tyg historii). Wartości statyczne — przeliczaj przy nowej historii.
**M16 King z σ_err**: M5 z σ_D→σ_err (RMSE błędu prognozy). Teoretycznie poprawne
wejście: ZB chroni przed BŁĘDEM PROGNOZY, nie surową zmiennością. Dla sezonowych
usuwa systematyczne zawyżenie. W AUTO: segmenty AX/AY/BX/BY, σ_err>0,
σ_err/σ_D ≤ 0,8 (prognoza musi realnie tłumaczyć ≥20% zmienności). Z prognozą
naiwną MA-4 rzadko się aktywuje — nabiera mocy po podpięciu RMSE z ERP/APS.

## Progi decyzyjne — tabela zbiorcza {#progi}
| Próg | Wartość | Rola |
|---|---|---|
| p̂ | > 1,32 tyg | popyt przerywany → M12/M13/M15/M8 |
| CV²_nz | > 0,49 | SBA zamiast Crostona (Syntetos-Boylan 2005) |
| n_nz | < 3 | MANUAL/MTO — statystyka zabroniona |
| n_nz | ≥ 8 | brama wiarygodności M15 |
| λ=D̄·LT_eff | < 5 i ẑ≤2 | Poisson dokładny (CRITBINOM) |
| σ_err/σ_D | ≤ 0,8 | M16 zamiast M5 |
| XYZ | CV_nz ≤0,5 / ≤1,0 / >1,0 | X / Y / Z |
| ABC | 80% / 95% wartości | A / B / C (Pareto) |
| pokrycie | 2–20 tyg | poza → alert |
| kapitał | >30% / 20–30% wart. rocznej | alert 🔴 / 🟡 |
| FR vs benchmark kraju | >10 p.p. | weryfikacja danych |
| DDMRP kandydat | LT>30 dni, AX/AY | flaga informacyjna |

## Funkcja straty G(z) dla M14 {#gz}
G(z) = φ(z) − z·(1−Φ(z)). Excel (legacy, przenośne):
`=EXP(-(A2^2)/2)/SQRT(2*PI())-A2*(1-NORMSDIST(A2))`
UWAGA na unarny minus: `-(A2^2)`, nigdy `-A2^2`! Tablica z=0,00…4,00 co 0,01;
odwrócenie: `INDEX(z; MATCH(G_req; kolumna_G; -1))` (kolumna malejąca).
G_req > G(0)=0,3989 → z<0 → ZB=0 (IFERROR).

## Bibliografia {#biblio}
Silver, Pyke & Peterson (1998) Inventory Management…, Wiley — M5, M14 (rozdz. 7);
Croston (1972) Oper. Res. Q. — M12; Syntetos & Boylan (2005) IJF — M13 i CV² 0,49;
Willemain, Smart & Schwarz (2004) IJF 20(3) — M15; Ptak & Smith (2016) DDMRP — M11;
Arrow, Harris & Marschak (1951) Econometrica — M10; Maister (1976) — M9;
Axsäter (2006) Inventory Control — M8; Hopp & Spearman (2001) Factory Physics — CSL vs FR;
Hadley & Whitin (1963) — Periodic Review; APICS CPIM (2022) — definicje.
