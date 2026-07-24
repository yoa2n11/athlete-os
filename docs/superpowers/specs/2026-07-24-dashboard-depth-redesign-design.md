# Design — Refonte en profondeur du dashboard

Date : 2026-07-24
Statut : approuvé, en attente de plan d'implémentation

## Contexte et problème

Le dashboard (`dashboard.html`) a déjà une structure à onglets (Vue jour, Historique, Zones, Totaux, Compétences, Feuille de route) mais chaque onglet n'exploite qu'une fraction des données réellement disponibles dans le projet :

- **Zones** : uniquement course à pied, alors que le profil a maintenant un FTP vélo (~260W) et un CSS natation (~1:38/100m).
- **Historique** : une table plate allure/FC, sans type de séance ni détail structuré.
- **Totaux** : volume + dénivelé sur 8 semaines fixes, un seul total toutes activités confondues.
- **Compétences** : une liste de tuiles statiques, sans lien avec ce que chaque compétence a réellement produit.
- **Feuille de route** : une frise avec juste des dates, alors que les 5 fichiers de blocs (Amsterdam à Séville) contiennent maintenant un vrai volume semaine par semaine.
- **Aucune vue CTL/ATL** (forme/fatigue dans le temps), alors que ces chiffres sont au cœur du fonctionnement de `recovery-check` et de `weekly-review`.

## Contrainte technique à respecter partout

`dashboard.html` est un **fichier statique**, ouvert directement dans le navigateur (double-clic, sans serveur). Il est régénéré à la demande ("mets à jour mon dashboard"), pas connecté en direct à intervals.icu ni capable d'exécuter une compétence. Toute idée d'"action" sur le dashboard (relancer une compétence, rafraîchir une donnée) doit se traduire par du texte à copier-coller dans le chat, jamais par un vrai bouton fonctionnel.

## Ce qui change, onglet par onglet

### Vue jour
- Ajoute un lien direct vers le fichier du bloc actif (ex. bloc Amsterdam) sous la carte "Bloc marathon, phases".
- Ajoute un encart "forme dans 2 semaines" : lit le volume prévu du bloc en cours pour la semaine +2 et donne une phrase de contexte (ex. "tu seras en pleine phase spécifique, ~4h de course prévues").

### Historique
- Remplace la table plate par une liste où chaque sortie a : date, type de séance (facile / qualité / longue — déduit du contenu, ex. présence de blocs Z4 = qualité), distance, allure, FC, **et** un lien "voir le détail" quand la séance vient d'un fichier de semaine structuré (échauffement/corps/récup déjà écrits dans les fichiers `training/*-week-plan.md`).
- Les séances qui n'ont pas de détail structuré (juste tirées d'intervals.icu) restent affichées simplement, sans lien mort.

### Zones
- Trois sections au lieu d'une : **Course** (existant, inchangé), **Vélo** (nouveau : FTP 260W, 7 zones Coggan en watts), **Natation** (nouveau : CSS 1:38/100m, 5 zones de pace).
- Chaque nouvelle section porte la mention "estimé par intervals.icu à partir du profil de puissance/vitesse, pas un test de terrain dédié" — cohérent avec la note déjà ajoutée dans `athlete-profile.md`.

### Totaux
- Sélecteur de période : 8 / 12 / 26 semaines (au lieu de 8 fixes).
- Le graphique de volume devient des barres empilées par sport (course/vélo/natation) au lieu d'un seul total, pour voir la répartition en un coup d'œil.
- Nouvelle vue "Prévu vs réel" : pour chaque semaine de la période choisie qui est couverte par un fichier de bloc, affiche le volume réel à côté du volume cible du bloc — visualise directement si l'entraînement suit le plan long terme ou s'en écarte.

### Compétences
- Chaque tuile affiche, en plus de sa description : la date/résultat de sa dernière exécution connue (ex. "Dernier appel : GO HARD, 24/07" pour recovery-check ; "Dernière review : 13/07" pour weekly-review) et un lien vers le dernier fichier qu'elle a produit (reviews/, races/, health/).
- Pas de bouton "relancer" fonctionnel (contrainte technique ci-dessus) — à la place, une ligne de texte copiable du type `Demande : "lance weekly-review"`.

### Feuille de route
- Chaque arrêt de la frise affiche désormais le volume du bloc qu'il représente (ex. "Bloc Séville, 9 sem., 4h → 5h → 3h30"), tiré directement des fichiers de blocs réels plutôt que d'un simple label de dates.

### Nouveau : graphique CTL/ATL
- Placé sur la page Vue jour (sous la carte "Appel du jour", qui n'affiche aujourd'hui que 3 chiffres bruts) ou sur Totaux — décision au moment du plan d'implémentation, pas structurante pour la spec.
- Courbe à deux séries : CTL et ATL réels sur les dernières semaines (données intervals.icu déjà utilisées ailleurs), puis une **projection en pointillés** au-delà d'aujourd'hui, calculée à partir des volumes prévus des blocs à venir (approximation simple : la charge d'entraînement suit le volume prévu, pas un vrai modèle physiologique — à indiquer clairement comme une projection, pas une prédiction garantie).

## Sources de données par section

| Section | Source |
|---|---|
| Zones vélo/natation | `athlete-profile.md` (FTP, CSS — déjà ajoutés) |
| Historique détaillé | intervals.icu (activités) + fichiers `training/*-week-plan.md` pour le détail structuré quand il existe |
| Totaux prévu vs réel | fichiers de blocs (`training/*.md`) pour le prévu, intervals.icu pour le réel |
| Compétences, dernière exécution | `reviews/`, `races/`, `health/` (dates de fichiers + contenu), pas de log d'exécution dédié à créer pour cette spec |
| Feuille de route détaillée | fichiers de blocs (déjà tous écrits) |
| CTL/ATL projection | intervals.icu (réel) + volumes des fichiers de blocs (projection) |

## Hors périmètre

- Pas de vraie interactivité serveur (pas de bouton qui exécute une compétence) — contrainte du fichier statique, déjà actée.
- Pas de log d'exécution dédié pour les compétences (ex. un fichier `health/skill-runs.json`) — la "dernière exécution" affichée se déduit des fichiers déjà produits (dates de reviews/races), pas d'un nouveau mécanisme de tracking à construire.
- La projection CTL/ATL reste une approximation simple (volume prévu → charge estimée), pas un modèle de charge d'entraînement (TSS/IF) complet — hors périmètre pour cette spec.
- Le contenu des fichiers de blocs eux-mêmes ne change pas (déjà traité dans la spec précédente) — cette spec ne fait que les **afficher** mieux.

## Addendum 2026-07-24 — Détail blessures dans l'onglet Zones

Contexte : import d'un handoff externe (projet claude.ai/design "Athlete OS application", 6 maquettes React) proposant une page "Santé" dédiée. Décision : pas de nouvel onglet — les maquettes Planning et Santé sont fusionnées dans Vue jour et Zones respectivement. Planning est déjà entièrement construit dans Vue jour (cartes jour par jour, bilan hebdo) ; seul le détail blessures manquait.

Changement :
- Nouvelle section dans l'onglet **Zones**, sous les 3 tableaux de zones : une carte par zone à risque (Achille droit, TFL gauche, ischio-jambier) avec statut, historique réel, symptômes, monitoring — sourcé depuis `athlete-profile.md` (section "Constraints and health"), pas depuis le texte générique de la maquette.
- Note "pattern de blessure identifié" basée sur les déclencheurs confirmés par l'athlète (2026-07-23 : hausse de volume trop rapide en prépa marathon ; 2026-07-24 : reprise trop rapide après une grosse course).
- La carte compacte "Santé, zones à surveiller" de Vue jour reste inchangée mais pointe désormais vers cette nouvelle section (lien "voir détail").
- Toujours aucune base de données, aucun backend — fichier statique régénéré à la main, comme le reste du dashboard.
