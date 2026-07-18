# Zapas Bezpieczeństwa 2.0 — audyt i budowa kalkulatorów Excel
### Szkic ulepszonego README.md dla repozytorium `b-safety-stock-skill-inventrooy`
*(rekomendacja: zmień nazwę repo na `safety-stock-toolkit` lub `zb-skill-inventory` — obecna nazwa ma literówkę "inventrooy")*

![License](https://img.shields.io/github/license/ojekjojek-creator/b-safety-stock-skill-inventrooy)
![Claude Skill](https://img.shields.io/badge/Claude-Skill-D97757?logo=anthropic&logoColor=white)
![Domain](https://img.shields.io/badge/Domain-Supply%20Chain-blue)

## Co to jest

Skill Claude AI (i metodologia do ręcznego użycia w Excelu) do liczenia i **audytowania**
zapasu bezpieczeństwa (ZB) dla portfeli SKU. Zawiera 16 metod ZB (King, DDMRP,
Croston/SBA, Fill Rate/ESC, kwantyl empiryczny, King z σ_err), automatyczne drzewo
wyboru metody per SKU oraz katalog **prawdziwych błędów formuł Excela**, które
wystąpiły w praktyce (np. porównanie tekstu z liczbą w `IF`, nieistniejące funkcje
typu `POISSON.INV`, błędny unarny minus w `EXP(-A1^2/2)`).

## Dlaczego to działa lepiej

Zbudowane i zwalidowane na realnym projekcie: **907 SKU**, dostawcy z Turcji/Chin/Europy,
plik Excel z ~56 tys. formuł, sześć iteracji recenzji eksperckiej.

| Miara | Bez skilla | Ze skillem |
|---|---|---|
| Poprawność audytu formuł ZB | 51% | **100%** |

## Dla kogo

- Kierowników logistyki i zakupów budujących własne kalkulatory ZB/ROP w Excelu
- Osób sprawdzających cudzy plik Excel z obliczeniami zapasów ("czy te formuły są poprawne?")
- Użytkowników Claude/Claude Code chcących dodać domenową wiedzę supply chain jako skill

## Jak używać

### Jako skill Claude
```
1. Sklonuj repozytorium lub skopiuj katalog do ~/.claude/skills/
2. Zapytaj Claude: "przeanalizuj ten plik Excel z obliczeniami zapasu bezpieczeństwa"
   albo "zbuduj kalkulator ZB dla mojego portfela SKU"
3. Skill automatycznie rozpozna tryb: AUDYT / BUDOWA / POJEDYNCZY SKU
```

### Jako metodologia (bez AI)
Pliki w `references/` (`metody_zb_16.md`, `struktura_v36.md`) opisują wzory i strukturę
kalkulatora — można je stosować bezpośrednio jako checklistę przy budowie własnego
arkusza Excel.

## Struktura repozytorium

```
├── SKILL.md              # definicja skilla (frontmatter + metodologia)
├── references/
│   ├── metody_zb_16.md   # 16 metod obliczania zapasu bezpieczeństwa
│   └── struktura_v36.md  # struktura kalkulatora Excel
├── LICENSE               # MIT
└── README.md
```

## Metody objęte skillem

King's Formula, DDMRP, Croston/SBA (popyt przerywany), Fill Rate/ESC, kwantyl
empiryczny, King z korektą σ_err, korekta fill rate dostawcy (LT_eff), profile
dostawców Turcja/Chiny/Europa, klasyfikacja ABC/XYZ.

## Licencja

MIT — patrz [LICENSE](LICENSE).

## Kontakt

Pytania i propozycje współpracy: [ojekjojek@gmail.com](mailto:ojekjojek@gmail.com)
