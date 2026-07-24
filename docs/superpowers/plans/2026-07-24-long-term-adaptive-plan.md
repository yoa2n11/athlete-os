# Plan long terme adaptatif — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remplacer la feuille de route macro par des fichiers de blocs détaillés jusqu'à Séville (21/02/2027), avec un mécanisme dans `weekly-review` qui ajuste les blocs futurs selon les écarts prévu/réel.

**Architecture:** Une nouvelle compétence `.claude/skills/long-term-plan/SKILL.md` génère 4 fichiers de blocs dans `training/` (récup post-Amsterdam, bloc Semi Boulogne, récup+base, bloc Séville) à partir d'un template de phase généralisé. `weekly-review` gagne une étape finale qui recalcule les semaines restantes du bloc en cours si l'écart dépasse 15%. `plan.md` devient un index pointant vers tous les blocs.

**Tech Stack:** Markdown uniquement (pas de code exécutable) — ce sont des fichiers de compétences et de plans lus par Claude Code. Pas de suite de tests automatisée ; la vérification de chaque tâche consiste à relire le fichier produit et confirmer les valeurs attendues.

## Global Constraints

- Aucune donnée inventée : chaque chiffre de volume vient de la formule du template (courbe dégressive + coupure), jamais d'une estimation à la louche — voir doc/superpowers/specs/2026-07-24-long-term-adaptive-plan-design.md.
- Les dates de bloc sont des ancrages fixes (dates de course réelles), jamais recalculées.
- `.claude/skills/` est gitignoré dans ce projet — les commits sur les fichiers de compétences resteront locaux, seuls les fichiers `training/*.md` et `plan.md` seront poussés sur GitHub si demandé.
- Un flag fatigue/blessure actif bloque toujours une hausse de volume proposée par le calcul.

---

### Task 1: Créer la compétence `long-term-plan`

**Files:**
- Create: `.claude/skills/long-term-plan/SKILL.md`

**Interfaces:**
- Consumes : lit `athlete-profile.md` (baseline de volume réel), `training/plan.md` (dates de course, contexte), le dernier fichier de bloc existant pour connaître le point de départ.
- Produces : fichiers `training/YYYY-MM-DD-<nom-bloc>.md`, un par bloc, au même format que `training/2026-08-17-amsterdam-marathon-block.md`.

- [ ] **Step 1: Écrire le fichier de compétence**

Contenu exact à écrire dans `.claude/skills/long-term-plan/SKILL.md` :

```markdown
---
name: long-term-plan
description: Use when building or rebuilding a long-term training block (a phase of several weeks tied to a specific race), or when a weekly-review adjustment needs to cascade into future blocks.
aliases: [long-term-plan]
---

# Long Term Plan

## Overview

Builds the detailed weekly structure for a future training block (volume, session types, target intensity) from a generalized template, using real starting data — never a guessed number. Used once to build every block through the athlete's last known race, and again only when a weekly-review adjustment is too large to absorb inside the current block.

## The generalized block template

**Phase split (blocks of 6 weeks or more)**, proportions taken from the Amsterdam block (1 récup + 2 base + 3 spécifique + 1 pivot + 2 affûtage over 9 weeks):
- Récup: ~11% of block length (min 1 week) — only if the block starts right after a race. Skip if the prior block already absorbed the recovery.
- Base: ~22% of block length
- Spécifique: ~33% of block length
- Pivot: ~11% of block length (min 1 week)
- Affûtage: remainder (min 2 weeks)

**Short blocks (under 6 weeks)**: use a simplified 3-phase template instead — Récup (0-1 week, skip if not needed) → Construction (base + spécifique merged) → Affûtage (1-2 weeks). For blocks of 2 weeks (a short sharpening block leveraging fitness from a very recent block), use just Construction (1 week) → Affûtage/race week (1 week), no récup phase.

**Volume progression (running-only hours, matching the Amsterdam block convention)**: starts from the last known *real* weekly running volume, never a theoretical number. Build weeks follow a degressive growth curve: the first build week grows ~18-20% over the starting baseline, decreasing roughly linearly to ~4-5% by the last build week before the pivot. Insert one down week (-20 to -30%) around the middle of the build phase if there are 4+ build weeks; skip it if there aren't enough weeks. The pivot week is the peak (highest volume, longest race-pace test). Affûtage follows the same shape as the Amsterdam block: roughly -30% then a further sharp cut in race week (~-55 to -60% off the pre-taper peak).

**Session structure per phase type** (same as the Amsterdam block, reused as-is):
- Récup: no hard sessions.
- Base: easy running + optional light VMA/hills only in the first 1-2 weeks of the block.
- Spécifique: 1 broken-threshold session/week (Bakken-style, e.g. 4-5x6-7min at threshold pace with jog recovery) + a long run with a goal-race-pace block that grows week over week.
- Pivot: the block's hardest/longest long run, the last real test before taper.
- Affûtage: sharply reduced volume, a few short touches at goal pace to keep the feel.

For a half-marathon target (e.g. Semi Boulogne-Billancourt), the long run's goal-pace block uses the half-marathon target pace instead of marathon pace — if that pace isn't fixed yet with the athlete, say so explicitly in the file rather than inventing one, and default the session to "long run with a moderate tempo finish, pace to confirm."

For a transitional block with no race at its end (e.g. "récup + base légère"), skip Spécifique/Pivot entirely — just Récup then Base, volume climbing back toward the athlete's stated regular baseline (6-8h/week total, all sports).

## Procedure

1. Read athlete-profile.md for the current stated regular volume baseline and injury history/watch zones.
2. Read training/plan.md for the block's fixed start/end dates (race calendar) and read the most recent existing block file to find the real running-hours figure to start the new block's calculation from.
3. Determine which phase template applies (full 5-phase, short 3-phase, 2-week sharpening, or transitional recovery+base) based on the block's length and whether it ends in a race.
4. Compute the week-by-week volume using the degressive growth curve and down-week rule above. Show the resulting number and the % change for every week — never skip stating the math.
5. Assign session structure per week from the phase-type table above.
6. Write the block file to `training/YYYY-MM-DD-<nom-bloc>.md` (block start date, kebab-case name), in the same format as `training/2026-08-17-amsterdam-marathon-block.md`: objective line, zone reference, week-by-week sections with volume/session detail and a one-line "why", safety rules section at the end.
7. Update `training/plan.md`'s block table to link to the new file.

## Quick reference

| Block length | Template |
|---|---|
| 2 weeks, ends in race | Construction (1wk) → Affûtage/race week (1wk), no récup |
| 3-5 weeks | Récup (0-1wk) → Construction → Affûtage (1-2wk) |
| 6+ weeks, ends in race | Full 5-phase split (Récup/Base/Spécifique/Pivot/Affûtage) |
| No race at the end | Récup → Base only, volume climbs to stated regular baseline |
```

- [ ] **Step 2: Vérifier le fichier**

Lire `.claude/skills/long-term-plan/SKILL.md` et confirmer qu'il contient bien les 4 sections (Overview, generalized block template, Procedure, Quick reference) et le frontmatter `name`/`description`/`aliases`.

- [ ] **Step 3: Commit (local uniquement, .claude/ est gitignoré)**

```bash
cd "C:/OpenClaw/Athlete OS"
git status --short .claude/skills/long-term-plan/
```
Attendu : `.claude/` est listé comme ignoré si on tente `git add` — c'est normal, ne pas forcer l'ajout.

---

### Task 2: Mettre à jour la compétence `weekly-review` avec l'étape d'ajustement

**Files:**
- Modify: `.claude/skills/weekly-review/SKILL.md`

**Interfaces:**
- Consumes: le fichier de bloc en cours (produit par Task 1), le % d'écart calculé entre volume réel et prévu de la semaine revue.
- Produces: fichier de bloc mis à jour (semaines restantes recalculées) si le seuil de 15% est dépassé ; appel à la compétence `long-term-plan` si cascade nécessaire.

- [ ] **Step 1: Lire le fichier actuel pour ne pas écraser le contenu existant**

```bash
cat "C:/OpenClaw/Athlete OS/.claude/skills/weekly-review/SKILL.md"
```

- [ ] **Step 2: Ajouter la nouvelle étape à la section Procedure, juste avant l'étape 5 (Write the review)**

Insérer ce texte juste avant l'étape numérotée qui commence par "**Write the review**" :

```markdown
4b. **Check the long-term plan adjustment.** Compute the % delta between this week's actual volume and its planned target from the current block file. If |delta| ≤ 15%, no change — note it as normal variance. If |delta| > 15% (alone, or two weeks running), redistribute the current block's remaining planned volume across its remaining weeks (same degressive shape, phase dates unchanged) and update the block file's remaining weeks in place. If the delta is too large to absorb within the current block without breaking the ~10%/week single-week guardrail, invoke the `long-term-plan` skill to cascade the recompute into the following block(s), and say so explicitly in the review — never adjust silently. An active fatigue/injury flag always blocks an upward adjustment regardless of the math.
```

- [ ] **Step 3: Mettre à jour la table "Quick reference" en ajoutant une ligne**

Ajouter à la table existante :

```markdown
| Long-term adjustment | >15% delta redistributes the current block's remaining weeks; bigger deltas cascade via long-term-plan, always stated explicitly |
```

- [ ] **Step 4: Vérifier**

Relire le fichier modifié et confirmer que l'étape 4b apparaît bien avant l'étape d'écriture de la review, et que la ligne de la quick reference a été ajoutée sans supprimer les lignes existantes.

---

### Task 3: Générer le fichier de bloc "Récup post-Amsterdam"

**Files:**
- Create: `training/2026-10-19-recup-post-amsterdam.md`

**Interfaces:**
- Consumes: date de fin du bloc Amsterdam (18/10/2026, course), baseline de reprise.
- Produces: fichier lu par `plan-my-week` quand ces semaines arriveront, et par Task 6 (mise à jour de plan.md).

- [ ] **Step 1: Écrire le fichier avec ce contenu exact**

```markdown
# Récup post-Amsterdam — 19/10 au 01/11/2026

Bloc de transition, pas de course à la fin — objectif unique : récupérer du marathon avant d'attaquer le Semi Boulogne-Billancourt. Template "récup → base" simplifié (voir .claude/skills/long-term-plan/SKILL.md).

## Semaine 1 — 19 au 25 octobre
- ~1h de course à pied, tout Z1, aucune séance structurée
- Pourquoi : le marathon du 18/10 est une épreuve exigeante pour les jambes, la priorité absolue est la récupération, pas le maintien de forme
- Vélo/natation très légers si les jambes le permettent, sinon repos actif

## Semaine 2 — 26 octobre au 01 novembre
- ~2h30 de course à pied (+150% vs S1, mais S1 était volontairement quasi nulle — pas un vrai saut de charge)
- Reprise Z1-Z2 progressive, aucune séance de qualité
- Pourquoi : reconstruire un socle avant le sharpening court du Semi Boulogne qui démarre le 02/11

## Règles de sécurité
- Aucune séance dure sur ce bloc entier
- Si gêne quelconque (Achille/TFL/ischio) : rallonger la récup d'une semaine plutôt que de forcer le Semi Boulogne
```

- [ ] **Step 2: Vérifier**

```bash
cat "C:/OpenClaw/Athlete OS/training/2026-10-19-recup-post-amsterdam.md"
```
Confirmer que les deux semaines et la règle de sécurité apparaissent.

- [ ] **Step 3: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add training/2026-10-19-recup-post-amsterdam.md
git commit -m "Add recovery block after Amsterdam marathon"
```

---

### Task 4: Générer le fichier de bloc "Semi Boulogne-Billancourt"

**Files:**
- Create: `training/2026-11-02-bloc-semi-boulogne.md`

**Interfaces:**
- Consumes: fin du bloc précédent (Task 3, baseline ~2h30 running), date de course fixe (15/11/2026).
- Produces: fichier lu par `plan-my-week` pour ces 2 semaines.

- [ ] **Step 1: Écrire le fichier avec ce contenu exact**

```markdown
# Bloc Semi Boulogne-Billancourt — 02 au 15 novembre 2026

Objectif B. Bloc très court (2 semaines) qui s'appuie sur la forme marathon Amsterdam déjà construite — pas un nouveau bloc de fond, template "construction courte → affûtage" (2 semaines, pas de phase récup séparée, voir .claude/skills/long-term-plan/SKILL.md).

Allure cible semi : pas encore fixée avec l'athlète — séances ci-dessous en "tempo modéré", à préciser une fois l'allure définie.

## Semaine 1 — 02 au 08 novembre — Sharpening
- ~3h30 de course à pied (+40% vs les ~2h30 de fin de récup — jump volontaire car on part d'une base artificiellement basse, pas d'une charge de référence)
- Une séance tempo modéré : 15min Z2 + 20min tempo (allure semi à confirmer) + 10min Z2
- Dim : sortie longue ~14-16km avec 20min à allure semi (à confirmer) en fin de sortie
- Pourquoi : rappeler la vitesse spécifique sans reconstruire un vrai bloc, la forme vient d'Amsterdam

## Semaine 2 — 09 au 15 novembre — Semaine de course
- ~1h30 de course à pied avant course (-57%)
- Jeu ou Ven : 15-20min très facile + quelques accélérations courtes
- **Dimanche 15 novembre : Semi de Boulogne-Billancourt — objectif B**

## Règles de sécurité
- Aucune séance dure après mardi de la semaine de course
- Si les jambes sont encore lourdes de la récup Amsterdam, réduire la séance tempo plutôt que la couper entièrement
```

- [ ] **Step 2: Vérifier**

```bash
cat "C:/OpenClaw/Athlete OS/training/2026-11-02-bloc-semi-boulogne.md"
```

- [ ] **Step 3: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add training/2026-11-02-bloc-semi-boulogne.md
git commit -m "Add Semi Boulogne-Billancourt sharpening block"
```

---

### Task 5: Générer les fichiers "Récup + base légère" et "Bloc marathon Séville"

**Files:**
- Create: `training/2026-11-16-recup-base.md`
- Create: `training/2026-12-20-bloc-marathon-seville.md`

**Interfaces:**
- Consumes: fin du bloc Semi Boulogne (Task 4, baseline ~1h30 running), date de course Séville fixe (21/02/2027).
- Produces: fichiers lus par `plan-my-week` pour ces semaines.

- [ ] **Step 1: Écrire `training/2026-11-16-recup-base.md` avec ce contenu exact**

```markdown
# Récup + base légère — 16 novembre au 20 décembre 2026

Bloc de transition, pas de course à la fin — objectif : revenir progressivement vers la base habituelle (6-8h/semaine toutes activités) avant le démarrage du bloc marathon Séville le 21/12. Template "récup → base" (voir .claude/skills/long-term-plan/SKILL.md).

## Semaine 1 — 16 au 22 novembre
- ~1h30 de course à pied, Z1, récup post-Semi Boulogne
- Pourquoi : le semi du 15/11 reste une course, pas un footing — vraie récup avant de reconstruire

## Semaine 2 — 23 au 29 novembre
- ~2h30 de course à pied (+67%, saut normal en sortie de récup profonde)
- Toujours Z1-Z2, aucune séance de qualité

## Semaine 3 — 30 novembre au 06 décembre
- ~3h de course à pied (+20%)
- Reprise progressive du rythme habituel (nat/vélo en plus si les jambes le permettent)

## Semaine 4 — 07 au 13 décembre
- ~3h20 de course à pied (+11%)
- Toujours aucune séance dure, priorité à la régularité

## Semaine 5 — 14 au 20 décembre
- ~3h30 de course à pied (+5%)
- Base atteinte pour démarrer le bloc marathon Séville le lendemain (21/12)

## Règles de sécurité
- Aucune séance dure sur ce bloc entier
- Si fatigue ou gêne persistante fin novembre, prolonger d'une semaine avant le bloc Séville plutôt que de forcer la date
```

- [ ] **Step 2: Écrire `training/2026-12-20-bloc-marathon-seville.md` avec ce contenu exact**

```markdown
# Bloc marathon Séville — 21 décembre 2026 au 21 février 2027

Objectif A, dernier objectif A de la séquence. Bloc de 9 semaines, même structure que le bloc Amsterdam (voir training/2026-08-17-amsterdam-marathon-block.md), avec un ajustement : pas de phase Récup séparée ici, puisque le bloc précédent (récup + base légère) vient déjà de l'assurer — la semaine supplémentaire est absorbée en Base (3 semaines au lieu de 2) plutôt que de forcer une semaine sans séance dure qui ne serait pas justifiée.

Zones de référence (voir athlete-profile.md) — mêmes zones que le bloc Amsterdam, à recalibrer si les données de fin d'année le justifient.

AS-Séville = allure spécifique marathon cible pour Séville — à définir avec l'athlète avant le démarrage réel du bloc (pas encore fixée aujourd'hui, objectif temps non discuté).

## Semaine 1 (21-27 déc) — Base 1
- ~4h de course à pied (+14% vs ~3h30 de fin de bloc précédent)
- Facile, VMA/côtes légères optionnelles autorisées (première et dernière semaine où c'est le cas dans ce bloc)

## Semaine 2 (28 déc-3 jan) — Base 2
- ~4h30 de course à pied (+12%)
- Facile, dernière touche VMA/côtes optionnelle si besoin

## Semaine 3 (4-10 jan) — Base 3
- ~5h de course à pied (+11%)
- Premier seuil brisé de reprise, format léger : 3x6' seuil R2'
- Dim : longue avec 4x10' à AS-Séville (à confirmer) en fin de sortie

## Semaine 4 (11-17 jan) — Spécifique 1, semaine de coupure
- ~3h45 de course à pied (-25%, coupure volontaire avant de repartir en construction)
- Séance seuil maintenue mais raccourcie, longue easy sans bloc spécifique cette semaine

## Semaine 5 (18-24 jan) — Spécifique 2
- ~4h20 de course à pied (+15%)
- Seuil brisé 4x7' R1:30
- Dim : longue avec 3x15' à AS-Séville

## Semaine 6 (25-31 jan) — Spécifique 3
- ~4h45 de course à pied (+10%)
- Seuil progressif 5x7' R1:30
- Dim : longue avec 3x20' à AS-Séville

## Semaine 7 (1-7 fév) — Pivot, semaine pivot (charge max)
- ~5h de course à pied (+5%), sortie longue la plus dure du bloc
- Dim : 28-30km avec 2x25' à AS-Séville + finale, dernier gros test avant l'affûtage

## Semaine 8 (8-14 fév) — Affûtage 1
- ~3h30 de course à pied (-30%)
- Une séance courte avec quelques km à AS-Séville pour garder les sensations

## Semaine 9 (15-21 fév) — Semaine de course
- ~1h30 de course à pied max avant course, tout Z1
- **Dimanche 21 février 2027 : Marathon de Séville — objectif A**

## Règles de sécurité
- Aucune séance dure les 48h suivant une sortie longue >2h
- Si douleur Achille/TFL/ischio : couper la séance d'intensité prévue, garder seulement du footing facile
- Renfo hebdo maintenu tout le bloc
```

- [ ] **Step 3: Vérifier les deux fichiers**

```bash
cat "C:/OpenClaw/Athlete OS/training/2026-11-16-recup-base.md"
cat "C:/OpenClaw/Athlete OS/training/2026-12-20-bloc-marathon-seville.md"
```
Confirmer que les 5 semaines du premier fichier et les 9 semaines + règles de sécurité du second sont bien présentes, et que les pourcentages d'une semaine à l'autre correspondent aux volumes indiqués (ex. 3h30 → 4h = +14%).

- [ ] **Step 4: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add training/2026-11-16-recup-base.md training/2026-12-20-bloc-marathon-seville.md
git commit -m "Add recup+base transition block and Seville marathon block"
```

---

### Task 6: Mettre à jour `plan.md` pour pointer vers tous les blocs

**Files:**
- Modify: `training/plan.md`

**Interfaces:**
- Consumes: les 4 nouveaux fichiers de blocs (Tasks 3-5) et le fichier Amsterdam existant.
- Produces: `plan.md` reste la référence lue en premier par `plan-my-week` et `weekly-review`.

- [ ] **Step 1: Lire le fichier actuel**

```bash
cat "C:/OpenClaw/Athlete OS/training/plan.md"
```

- [ ] **Step 2: Remplacer la ligne de la feuille de route macro**

Chercher la ligne qui commence par `Détail semaine par semaine des phases 1 à 5 déjà écrit dans` et la section "Feuille de route macro" plus bas dans le fichier. Remplacer tout le contenu de la section `## Feuille de route macro` par :

```markdown
## Feuille de route détaillée — jusqu'à Séville (21 février 2027)

Chaque bloc a maintenant son fichier détaillé (volume semaine par semaine, structure de séances), construit avec la compétence `long-term-plan`. Plus de tableau macro seul — voici l'index :

| Bloc | Dates | Fichier |
|---|---|---|
| Bloc marathon Amsterdam | 17/08 → 18/10/2026 | [2026-08-17-amsterdam-marathon-block.md](2026-08-17-amsterdam-marathon-block.md) |
| Récup post-Amsterdam | 19/10 → 01/11/2026 | [2026-10-19-recup-post-amsterdam.md](2026-10-19-recup-post-amsterdam.md) |
| Bloc Semi Boulogne-Billancourt | 02/11 → 15/11/2026 | [2026-11-02-bloc-semi-boulogne.md](2026-11-02-bloc-semi-boulogne.md) |
| Récup + base légère | 16/11 → 20/12/2026 | [2026-11-16-recup-base.md](2026-11-16-recup-base.md) |
| Bloc marathon Séville | 21/12/2026 → 21/02/2027 | [2026-12-20-bloc-marathon-seville.md](2026-12-20-bloc-marathon-seville.md) |

Les semaines restantes de chaque bloc sont recalculées automatiquement par `weekly-review` si l'écart entre volume prévu et réel dépasse 15% une semaine donnée (voir `.claude/skills/weekly-review/SKILL.md`) — ce tableau reste la source de vérité pour les dates, le détail vit dans chaque fichier.
```

- [ ] **Step 3: Vérifier**

```bash
grep -A 10 "Feuille de route détaillée" "C:/OpenClaw/Athlete OS/training/plan.md"
```
Confirmer que les 5 lignes du tableau apparaissent avec les bons chemins de fichiers.

- [ ] **Step 4: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add training/plan.md
git commit -m "Point plan.md at detailed block files through Seville"
```

---

### Task 7: Vérification finale de cohérence

**Files:**
- Read only: tous les fichiers créés/modifiés dans les tasks 1-6.

- [ ] **Step 1: Vérifier la continuité des dates entre blocs**

Confirmer qu'il n'y a pas de trou ni de chevauchement : Amsterdam se termine 18/10 → récup commence 19/10 → récup finit 01/11 → Semi Boulogne commence 02/11 → finit 15/11 → récup+base commence 16/11 → finit 20/12 → Séville commence 21/12 → finit 21/02/2027.

- [ ] **Step 2: Vérifier la continuité des volumes entre blocs**

Confirmer que le volume de fin d'un bloc correspond au point de départ annoncé du bloc suivant (ex. récup+base finit à ~3h30, le bloc Séville S1 part bien de "+14% vs ~3h30").

- [ ] **Step 3: Pousser sur GitHub si l'athlète le demande**

```bash
cd "C:/OpenClaw/Athlete OS"
git log --oneline -8
```
Attendre la confirmation explicite de l'athlète avant `git push` (ne jamais pousser automatiquement).
