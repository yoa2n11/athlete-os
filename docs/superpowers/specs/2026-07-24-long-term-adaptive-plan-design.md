# Design — Plan long terme adaptatif

Date : 2026-07-24
Statut : approuvé, en attente de plan d'implémentation

## Contexte et problème

Le système actuel construit le plan un bloc à la fois, juste avant qu'il ne démarre (philosophie "juste-à-temps" de `plan-my-week` et du bloc Amsterdam). Ça a bien fonctionné pour le bloc en cours, mais ça laisse deux trous :

1. **Aucune vision d'ensemble** : la feuille de route dans `plan.md` n'est qu'un tableau de phases avec des dates, sans volume ni structure de séances — impossible de voir où on va vraiment jusqu'à Séville.
2. **Pas d'évolution automatique** : les retours quotidiens (recovery-check) et hebdomadaires (weekly-review) sont aujourd'hui de l'analyse pure — ils ne modifient jamais le plan des semaines suivantes.

L'athlète veut explicitement changer cette philosophie : construire tous les blocs jusqu'à Séville (21 février 2027) maintenant, avec des chiffres réels calculés (pas inventés), et faire en sorte que les retours hebdomadaires fassent évoluer ce plan long terme au fil de l'eau.

## Architecture retenue

**Un fichier détaillé par bloc, `plan.md` en index.** Prolonge la convention déjà en place (le bloc Amsterdam est déjà un fichier séparé référencé depuis `plan.md`).

Fichiers à créer (en plus du bloc Amsterdam existant, inchangé) :
- `training/2026-10-19-recup-post-amsterdam.md`
- `training/2026-11-02-bloc-semi-boulogne.md`
- `training/2026-11-16-recup-base.md`
- `training/2026-12-20-bloc-marathon-seville.md`

`plan.md` garde son rôle d'index : objectifs, contexte de données, tableau macro des phases, liens vers chaque fichier de bloc. Il n'a plus besoin de porter le détail des phases lui-même — chaque fichier de bloc devient la source de vérité pour sa période.

## Le template de bloc généralisé

Le bloc Amsterdam (déjà construit) sert de référence : 9 semaines, structure Récup → Base → Spécifique → Pivot → Affûtage. On généralise cette structure pour qu'elle s'applique à n'importe quel bloc futur, plutôt que de la reconstruire à la main à chaque fois.

**Découpage des phases (blocs de 6 semaines ou plus)**, proportions tirées du bloc Amsterdam (1 récup + 2 base + 3 spécifique + 1 pivot + 2 affûtage sur 9 semaines) :
- Récup : ~11% de la durée du bloc (mini 1 semaine)
- Base : ~22% de la durée du bloc
- Spécifique : ~33% de la durée du bloc
- Pivot : ~11% de la durée du bloc (mini 1 semaine)
- Affûtage : le reste (mini 2 semaines)

**Blocs courts (moins de 6 semaines, ex. Semi Boulogne)** : le découpage à 5 phases produit des fractions dégénérées. Template simplifié à 3 phases : Récup (0-1 semaine, sautée si le bloc précédent vient déjà de finir sur une récup) → Construction (base + spécifique fusionnés) → Affûtage (1-2 semaines).

**Progression du volume à l'intérieur d'un bloc** : pas un taux fixe — une courbe dégressive, comme observée dans le bloc Amsterdam (+20%, +17%, +14%, +8%, +5%, +4% d'une semaine construction à l'autre). Règle : le taux de croissance de la première semaine de construction tourne autour de 18-20%, puis décroît de façon quasi linéaire jusqu'à 4-5% à la dernière semaine avant le pivot. Une coupure de -20 à -30% est insérée tous les 3-4 semaines de construction (sautée si le bloc n'a pas assez de semaines de construction pour ça). Le point de départ du volume est toujours le dernier volume hebdomadaire **réel** connu au moment du calcul, jamais un chiffre théorique.

**Structure des séances par type de phase** (reprend ce qui a été fait pour Amsterdam, généralisé) :
- Récup : aucune séance dure.
- Base : facile + VMA/côtes légères optionnelles en tout début de bloc seulement.
- Spécifique : 1 séance seuil brisée/semaine (format Bakken, comme construit pour Amsterdam) + sortie longue avec bloc à l'allure cible de la course qui grossit semaine après semaine.
- Pivot : la sortie longue la plus dure du bloc, dernier vrai test avant l'affûtage.
- Affûtage : volume en forte baisse, quelques touches d'allure cible pour garder les sensations.

Pour un bloc semi (Boulogne-Billancourt), l'allure cible dans la longue est l'allure semi (à définir avec l'athlète le moment venu, pas encore fixée), pas l'allure marathon.

Le bloc "Récup + base légère" (16/11 → 19/12) est un cas particulier : ce n'est pas un bloc construit autour d'une course, donc pas de phase spécifique/pivot — juste Récup puis Base, volume qui remonte doucement vers la base habituelle de l'athlète (6-8h/semaine) avant que le vrai bloc Séville démarre.

## Mécanisme d'ajustement (weekly-review)

Ajustement **hebdomadaire uniquement** — recovery-check (quotidien) continue de flagger/alerter mais ne modifie jamais le plan.

Après chaque semaine réelle, weekly-review calcule l'écart en % entre le volume réel et le volume prévu pour cette semaine :

- **Écart ≤ 15%** (dans un sens ou l'autre) : bruit normal de semaine, aucun changement au plan.
- **Écart > 15%**, sur une semaine isolée ou deux semaines consécutives : les semaines **restantes du bloc en cours** sont recalculées — le volume total restant prévu pour le bloc est redistribué sur les semaines qui restent, avec la même courbe dégressive, sans changer les dates de début/fin du bloc (les dates de course sont des ancrages fixes, jamais déplacés).
- **Écart trop important pour être absorbé** dans le bloc en cours sans dépasser la règle des ~10%/semaine sur une seule semaine (ex. plusieurs semaines d'arrêt) : ça cascade sur le(s) bloc(s) suivant(s) — recalcul des fichiers de blocs futurs. Ce cas est **toujours signalé explicitement** à l'athlète avec la raison, jamais appliqué silencieusement.
- **Flag fatigue/blessure actif** (ATL très au-dessus de CTL, ou zone à surveiller signalée) : bloque toute augmentation de volume proposée par le calcul, quel que soit le résultat de la formule — le flag prime toujours.

## Changements de compétences

**Nouvelle compétence `long-term-plan`** :
- Construit ou reconstruit les fichiers de blocs listés ci-dessus à partir des règles ci-dessus.
- Utilisée maintenant pour construire tout d'un coup jusqu'à Séville, puis réutilisée uniquement en cas de cascade majeure (le cas rare ci-dessus).
- Lit : `athlete-profile.md`, `plan.md`, le dernier volume réel connu (intervals.icu), les fichiers de blocs déjà existants (pour connaître le point de départ). Écrit : les fichiers de blocs listés ci-dessus.

**`weekly-review` mise à jour** :
- Nouvelle étape finale : calcule l'écart prévu vs réel de la semaine revue, applique l'ajustement au bloc en cours si le seuil de 15% est dépassé (édite directement les semaines restantes du fichier de bloc concerné), déclenche une cascade vers `long-term-plan` si nécessaire.
- Dit toujours explicitement ce qui a changé et pourquoi dans la review elle-même.

**`plan-my-week` inchangée dans l'esprit** :
- Continue de lire le fichier de bloc de la phase en cours pour construire la semaine — sauf qu'il y a désormais toujours un vrai fichier de bloc à lire, même loin dans le temps, plus de trou comme celui rencontré avant le démarrage du bloc Amsterdam.

## Ce qui se passe immédiatement

Une fois ce plan implémenté, la compétence `long-term-plan` est invoquée une première fois pour construire les 4 fichiers de blocs manquants (récup post-Amsterdam, bloc Semi Boulogne, récup+base, bloc Séville), en utilisant les vraies données d'aujourd'hui comme point de départ. Le bloc Amsterdam existant n'est pas retouché.

## Hors périmètre

- Les zones/allures pour le Semi Boulogne-Billancourt (allure cible semi) ne sont pas encore fixées avec l'athlète — à définir plus tard, pas un blocage pour construire la structure du bloc maintenant.
- Rien au-delà de Séville (21/02/2027) : pas de course connue après, donc pas de plan à construire, cohérent avec YAGNI.
- Les améliorations du dashboard (CTL/ATL graphique, historique détaillé, zones multi-sports, totaux enrichis, page compétences) font l'objet d'une spec séparée, pas traitées ici.

## Format des fichiers de bloc

Chaque bloc de la feuille de route est un fichier Markdown autonome, lisible par `plan-my-week` et `long-term-plan`.

- Titre : nom du bloc et dates exactes de début et de fin.
- Objectif : phrase courte décrivant l'objectif spécifique du bloc.
- Semaine par semaine : pour chaque semaine, indiquer le volume cible de course à pied, le type principal de séances, une ligne "Pourquoi" et les points de vigilance.
- Sécurité : une section finale de règles d'arrêt ou de réduction de charge.
- Transparence : toute valeur non encore fixée par l'athlète (allure semi, allure Séville, etc.) doit être signalée comme "à confirmer".

## Règles de calcul et invariants

- Le point de départ de chaque nouveau bloc est le dernier volume hebdomadaire réel connu, jamais une estimation CSV arbitraire.
- Les dates de début/fin de bloc restent fixes : elles sont ancrées aux jalons de course ou à la transition planifiée, et ne sont jamais reculées/avancées par le recalcul adaptatif.
- La progression du volume est une courbe dégressive : forte croissance initiale puis réduction du rythme d'augmentation, avec un down-week planifié au milieu du build si le bloc compte 4 semaines de construction ou plus.
- Les écarts réels/prévus ≤ 15% sont traités comme variance normale et n'entraînent aucun changement automatique.
- Les écarts > 15% déclenchent un recalcul des semaines restantes du bloc courant si le bloc peut les absorber sans dépasser ~10% d'ajustement d'une semaine à l'autre. Si le bloc ne peut pas absorber l'écart, la cascade se propage vers les blocs futurs.
- Un flag fatigue/blessure actif arrête toute augmentation de volume, même si les calculs demandent une hausse.

## Spécification de `plan.md`

Le fichier `training/plan.md` devient un index macro, pas un plan détaillé.

- Il liste les blocs et leurs dates.
- Il inclut un court résumé de l'objectif général jusqu'à Séville.
- Il renvoie par lien vers chaque fichier de bloc détaillé.
- Il conserve le contexte de données utilisé pour la construction du plan long terme, y compris la date de la dernière donnée réelle prise en compte.

## Critères d'acceptation

- `training/plan.md` référence explicitement tous les blocs jusqu'à Séville et ne contient plus le détail exact de chaque semaine.
- Les 4 fichiers suivants sont présents et structurés selon le template :
  - `training/2026-10-19-recup-post-amsterdam.md`
  - `training/2026-11-02-bloc-semi-boulogne.md`
  - `training/2026-11-16-recup-base.md`
  - `training/2026-12-20-bloc-marathon-seville.md`
- Le document de spécification décrit le mécanisme adaptatif de `weekly-review` avec seuil de 15% et cascade explicite.
- Toute donnée inconnue est marquée comme « à confirmer » et n'est pas inscrite comme un chiffre définitif.
- Les changements de volume se basent toujours sur des chiffres calculés, jamais sur des valeurs inventées.

## Suivi et relecture

- Une fois les fichiers créés, relire chaque bloc pour vérifier que le volume cible, la structure de phase et les règles de sécurité sont cohérents avec le template général.
- Vérifier que le texte explicite les points d'arrêt : course cible, déconnexion entre performance réelle et plan, et fatigue/bles­sure.
- Si un bloc est reconstruit après une review, noter clairement dans le bloc que la version est issue d'un ajustement adaptatif, avec la raison courte et l'impact sur les semaines restantes.
