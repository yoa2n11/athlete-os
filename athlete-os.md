# My Athlete OS

Dashboard — dernière mise à jour 2026-07-24.

## Objectif de l'année
**Marathon d'Amsterdam — dimanche 18 octobre 2026.**
- A : sub-2h37/2h38, relance sur le dernier tiers si les sensations sont bonnes
- B : sub-2h40 en pacing régulier (3:47/km verrouillé)
- C : sub-2h40 en mode gestion si ça se complique — sub-2h40 reste le plancher, pas un objectif "au mieux"
→ Détail complet : [races/2026-10-18-amsterdam-marathon-dossier.md](races/2026-10-18-amsterdam-marathon-dossier.md)

## Où j'en suis dans le plan
Phase actuelle (voir [training/plan.md](training/plan.md)) : **Phase 0 — fin de prépa Embrunman**, jusqu'au 15 août. Priorité triathlon, aucune séance dure ajoutée avant le bloc marathon.
Bloc marathon détaillé (séances semaine par semaine, 17 août → 18 octobre) : [training/2026-08-17-amsterdam-marathon-block.md](training/2026-08-17-amsterdam-marathon-block.md)

## Prochaines échéances
| Date | Course | Objectif |
|---|---|---|
| 15/08/2026 | Quart Embrunman (1,5km/44km/10km) | C — bonus, protéger la reprise du bloc marathon 2 jours après |
| 18/10/2026 | Marathon d'Amsterdam | A — sub-2h40 minimum |
| 15/11/2026 | Semi de Boulogne-Billancourt | B — après Amsterdam, hors bloc actuel |

Plan de course Embrunman : [races/2026-08-15-embrunman-race-plan.md](races/2026-08-15-embrunman-race-plan.md)
Plan fueling & pacing Amsterdam : [races/2026-10-18-amsterdam-fueling-pacing-plan.md](races/2026-10-18-amsterdam-fueling-pacing-plan.md)

## Dernière review hebdomadaire
[reviews/2026-07-13.md](reviews/2026-07-13.md) — sortie longue exécutée avec contrôle (bon signe), mais stress/sommeil perturbés ont fait grimper la fatigue (ATL) plus que prévu ; légère gêne ischio-jambier notée (même zone qu'une déchirure légère du 31 mai, résolue) à surveiller à la reprise post-Embrunman.

## Points de vigilance actifs
- **Volume course à pied** : jamais plus de ~10%/semaine de hausse en bloc marathon — historique confirmé (blessures 2022/2023 précédées de hausses de 18-47% en fin de prépa).
- **Trois zones à surveiller** : Achille (droit), TFL (gauche), ischio-jambier (nouveau, résolu mais réapparu brièvement le 19-20/07).
- **Reprise post-Embrunman (17/08)** : semaine 1 volontairement sans séance dure, même schéma de risque qu'après le Half Angers de mai.

## Compétences disponibles (.claude/skills/)
- [.claude/skills/plan-my-week/SKILL.md](.claude/skills/plan-my-week/SKILL.md) — construit la semaine à venir à partir du plan de phase + données réelles
- [.claude/skills/weekly-review/SKILL.md](.claude/skills/weekly-review/SKILL.md) — analyse une semaine passée (exécution, ressenti vs données, tendances)
- [.claude/skills/fuel-and-pace/SKILL.md](.claude/skills/fuel-and-pace/SKILL.md) — plan fueling/pacing chiffré pour une course donnée
- [.claude/skills/recovery-check/SKILL.md](.claude/skills/recovery-check/SKILL.md) — appel de forme quotidien (GO HARD/EASY/REST), programmé chaque matin

## Ma philosophie de coaching
Synthèse Lydiard (base aérobie) + Bakken (seuil brisé hebdo) + Bu (rigueur données), adaptée à mon budget temps réel et à mon historique de blessures. Détail complet et sources : [athlete-profile.md](athlete-profile.md#coaching-philosophy).

## Structure du projet
- `training/` — plan de phase (plan.md) et blocs/semaines datés
- `reviews/` — reviews hebdomadaires, jamais écrasées
- `races/` — dossiers de course et plans fueling/pacing
- `health/` — vide pour l'instant (pas encore construit)
- `.claude/skills/` — compétences réutilisables listées ci-dessus
- `athlete-profile.md` — profil, objectifs, philosophie de coaching, historique
- `CLAUDE.md` — instructions persistantes du projet
