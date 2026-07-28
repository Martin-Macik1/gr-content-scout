# GR Content Scout

Týždenný agent, ktorý sleduje marketingový obsah konkurencie a dáva konkrétne
návrhy obsahu (blog, case studies) pre GoodRequest. Beží ako **Claude Code routine**
(cloud) a výstup posiela do Slack kanála `_gr-opportunity-scout`.

## Obsah repozitára
- `content-scout-task.md` - kompletný spec/prompt agenta (source of truth).
- `competitors.yaml` - zoznam sledovaných zdrojov + obsahové signály. Tu edituj konkurenciu.
- `reports/content-state.json` - stav medzi behmi (čo už bolo videné/navrhnuté). Prvý beh = baseline.
- `reports/YYYY-MM-DD-content.md` - plný týždenný report (vytvára agent každý beh).

## Nastavenie routine (raz)
1. Nahraj tento repozitár do GitHubu (`gr-content-scout`).
2. `claude.ai/code/routines` -> **New routine**.
3. **Instructions** vlož:
   ```
   Si Content Scout pre GoodRequest. Postupuj presne podľa content-scout-task.md v koreni repozitára.
   1. Načítaj content-scout-task.md, competitors.yaml a reports/content-state.json.
   2. Sprav týždennú obsahovú analýzu konkurencie podľa spec (potrebuješ web).
   3. Výstup POSTni do Slack kanála _gr-opportunity-scout (hlavný post + vlákno) cez Slack konektor.
   4. Ulož reports/{YYYY-MM-DD}-content.md, aktualizuj reports/content-state.json, commitni a pushni na main.
   5. Ak content-state.json neexistuje alebo je is_baseline=true = prvý beh = baseline, nič neoznačuj ako "nové" a napíš to v poste.
   Úspech = Slack post odoslaný a stav commitnutý. Ak web alebo Slack zlyhá, jasne to napíš do behu.
   ```
4. Model: **Opus**.
5. **Repository:** `gr-content-scout`.
6. **Trigger -> Schedule:** Weekly, pondelok 08:00 (lokálna zóna, prevedie sa automaticky).
7. **Run now** -> over baseline.

## Dve pasce (inak to potichu zlyhá)
- **Sieť:** default prostredie „Trusted" blokuje ľubovoľné domény. Agent ťahá weby
  konkurencie -> prepni **Network access na Full** (alebo Custom + domény z `competitors.yaml`).
  MCP konektory (Slack, Drive) idú cez servery Anthropicu, fungujú aj bez toho.
- **Stav:** Claude defaultne pushuje len do vetiev `claude/...`. Aby sa
  `reports/content-state.json` zapisoval do `main` (a agent si „pamätal"), zapni
  **Allow unrestricted branch pushes** pre `gr-content-scout`.

## Overenie po prvom behu
Zelený status = relácia len naštartovala a skončila bez infra chyby, NIE že úloha uspela.
Otvor transcript behu a over, že reálne prebehol Slack post aj commit `content-state.json`.

## Bonus
K routine sa dá pridať **API trigger** (`POST .../fire` s bearer tokenom) a spúšťať
ad-hoc beh napr. z n8n/HubSpotu mimo pondelka.

## Údržba
- Nového konkurenta pridáš do `competitors.yaml` (skopíruj blok, vyplň weby a priority).
- Po pár behoch doplň do zdrojov aj kľúčových ľudí na LinkedIn (často väčší engagement než firemný profil).
