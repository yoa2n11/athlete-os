# Refonte en profondeur du dashboard — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Enrichir chaque onglet du dashboard (`dashboard.html`) avec les données réelles déjà disponibles mais pas encore affichées : zones multi-sports, historique détaillé, totaux par sport avec comparaison prévu/réel, compétences avec dernière exécution, feuille de route chiffrée, et un nouveau graphique CTL/ATL avec projection.

**Architecture:** Toutes les modifications se font dans le fichier statique unique `dashboard.html` (HTML/CSS/JS vanilla, pas de build, pas de dépendance externe). Chaque tâche ajoute ou enrichit une section existante sans casser les onglets déjà fonctionnels. Aucune "action" ne doit être un vrai bouton fonctionnel (fichier statique, pas de backend) — remplacé par du texte copiable.

**Tech Stack:** HTML/CSS/JS vanilla dans un seul fichier, ouvert en local (`file://`). Vérification = ouvrir le fichier dans le navigateur et lire/cliquer, pas de suite de tests automatisée.

## Global Constraints

- Ne jamais ajouter de bouton qui prétend exécuter une compétence — le fichier est statique, aucune action réelle possible depuis le navigateur.
- Toute donnée estimée (FTP, CSS, projection CTL/ATL) doit porter une mention explicite de son origine, jamais présentée comme une mesure de terrain.
- Les 5 fichiers de blocs (`training/2026-08-17-amsterdam-marathon-block.md` et les 4 nouveaux) sont la source de vérité pour les volumes "prévus" — ne jamais inventer un chiffre prévu qui n'y figure pas.
- Garder le thème sombre existant (variables CSS `--bg`, `--panel`, `--accent`, etc. déjà définies en haut du fichier) — toute nouvelle section réutilise ces variables, pas de nouvelles couleurs isolées.

---

### Task 1: Zones vélo et natation

**Files:**
- Modify: `dashboard.html` (page `#page-zones`, section CSS `.zones-table`)

**Interfaces:**
- Consumes: rien de dynamique — valeurs FTP (260W) et CSS (1:38/100m) écrites en dur dans le HTML, comme le sont déjà les zones course.
- Produces: rien consommé par une autre tâche.

- [ ] **Step 1: Lire la structure actuelle de la page Zones**

```bash
grep -n "page-zones" -A 30 "C:/OpenClaw/Athlete OS/dashboard.html"
```

- [ ] **Step 2: Ajouter les deux nouvelles sections après le tableau Course existant, à l'intérieur de `#page-zones`**

Insérer juste après la fermeture du `<div class="card zones-table">` existant (celui des zones course), toujours à l'intérieur de `.grid` :

```html
<div class="card zones-table">
  <h3>Zones vélo (FTP ~260W)</h3>
  <div class="zrow"><div class="zlabel z1">Z1</div><div>&lt; 143 W (&lt;55%)</div><div>Récupération</div></div>
  <div class="zrow"><div class="zlabel z2">Z2</div><div>143-195 W (55-75%)</div><div>Endurance</div></div>
  <div class="zrow"><div class="zlabel z3">Z3</div><div>195-234 W (75-90%)</div><div>Tempo</div></div>
  <div class="zrow"><div class="zlabel z4">Z4</div><div>234-273 W (90-105%)</div><div>Seuil</div></div>
  <div class="zrow"><div class="zlabel z5">Z5</div><div>273-312 W (105-120%)</div><div>VO2max</div></div>
  <div class="zrow"><div class="zlabel z4">Z6</div><div>312-390 W (120-150%)</div><div>Anaérobie</div></div>
  <div class="zrow"><div class="zlabel z5">Z7</div><div>&gt; 390 W (&gt;150%)</div><div>Neuromusculaire</div></div>
  <div class="zsub">FTP estimé par intervals.icu à partir du profil de puissance (2026-07-24), pas un test de terrain dédié (ramp test) — bon point de départ, à affiner.</div>
</div>

<div class="card zones-table">
  <h3>Zones natation (CSS ~1:38/100m)</h3>
  <div class="zrow"><div class="zlabel z1">Z1</div><div>Plus lent que 1:50/100m</div><div>Facile</div></div>
  <div class="zrow"><div class="zlabel z2">Z2</div><div>1:46-1:50/100m</div><div>Aérobie / soutenu</div></div>
  <div class="zrow"><div class="zlabel z3">Z3</div><div>1:36-1:40/100m</div><div>Seuil / CSS</div></div>
  <div class="zrow"><div class="zlabel z4">Z4</div><div>1:32-1:36/100m</div><div>VO2max / rapide</div></div>
  <div class="zrow"><div class="zlabel z5">Z5</div><div>Plus rapide que 1:32/100m</div><div>Sprint</div></div>
  <div class="zsub">CSS estimé par intervals.icu (2026-07-24), pas un vrai test CSS (2x400m/2x200m chronométrés) — bon point de départ, à affiner.</div>
</div>
```

- [ ] **Step 3: Vérifier dans le navigateur**

Ouvrir `dashboard.html`, cliquer sur l'onglet Zones, confirmer que 3 tableaux apparaissent maintenant (Course, Vélo, Natation) avec les bonnes valeurs.

- [ ] **Step 4: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add bike and swim zones to dashboard (FTP 260W, CSS 1:38/100m)"
```

---

### Task 2: Graphique CTL/ATL avec projection

**Files:**
- Modify: `dashboard.html` (page `#page-jour`, nouvelle carte ; CSS ; script)

**Interfaces:**
- Consumes: aucune donnée d'une autre tâche.
- Produces: rien consommé ailleurs.

- [ ] **Step 1: Ajouter la carte HTML dans `#page-jour`, après la carte `.phases` et avant `.sessions`**

```html
<div class="card" style="grid-column: span 12;">
  <h3>Forme / fatigue (CTL/ATL), 21 dernières semaines + projection</h3>
  <div id="chart-ctl-atl" style="height:180px; position:relative;"></div>
  <div class="zone-legend" style="margin-top:10px; border-top:none; padding-top:0;">
    <div class="lg"><span class="sw z2" style="background:var(--accent-2)"></span> CTL réel (forme)</div>
    <div class="lg"><span class="sw z4" style="background:var(--accent)"></span> ATL réel (fatigue)</div>
    <div class="lg"><span class="sw z1" style="background:var(--muted)"></span> Projection (pointillés, basée sur le volume prévu des blocs à venir)</div>
  </div>
  <div class="note" style="margin-top:8px">Projection = approximation simple (volume prévu × ratio charge/heure récent), pas un modèle physiologique complet — indicatif, pas une prédiction garantie.</div>
</div>
```

- [ ] **Step 2: Ajouter le CSS pour le graphique en ligne (SVG inline généré en JS)**

Ajouter dans le bloc `<style>` :

```css
#chart-ctl-atl svg { width: 100%; height: 100%; }
.ctl-line { fill: none; stroke: var(--accent-2); stroke-width: 2; }
.atl-line { fill: none; stroke: var(--accent); stroke-width: 2; }
.ctl-proj { fill: none; stroke: var(--accent-2); stroke-width: 2; stroke-dasharray: 4 4; opacity: 0.6; }
.atl-proj { fill: none; stroke: var(--accent); stroke-width: 2; stroke-dasharray: 4 4; opacity: 0.6; }
```

- [ ] **Step 3: Ajouter les données réelles et la fonction de rendu dans le script**

Ajouter avant la ligne `document.querySelectorAll(".nav-item").forEach(...)` :

```javascript
// CTL/ATL réel, 21 dernières semaines (intervals.icu, semaines ISO 2026)
const CTL_REAL = [45.4,49.6,47.1,47.0,49.2,51.2,50.5,54.9,60.0,62.7,65.4,66.4,62.9,59.9,60.7,59.2,58.8,62.6,62.3,65.0];
const ATL_REAL = [35.7,56.6,47.1,46.9,58.9,58.0,50.4,61.8,81.0,82.7,83.2,82.7,57.2,43.4,56.4,53.3,55.6,74.6,67.5,75.9];

// Projection : volume prévu (heures course) des blocs à venir, converti en charge
// approximative avec un ratio ~50 de charge par heure (dérivé des semaines réelles récentes,
// ex. semaine 9.2h -> charge 489 = ~53/h). 6 points de projection = 6 prochaines semaines du bloc Amsterdam.
const PLANNED_HOURS_NEXT = [2.5, 3.0, 3.5, 4.0, 4.3, 4.5]; // S1-S6 du bloc Amsterdam, training/2026-08-17-amsterdam-marathon-block.md
const LOAD_PER_HOUR = 50;
function projectCtlAtl(lastCtl, lastAtl, plannedHours) {
  let ctl = lastCtl, atl = lastAtl;
  const ctlProj = [ctl], atlProj = [atl];
  plannedHours.forEach(h => {
    const load = h * LOAD_PER_HOUR;
    ctl = ctl + (load - ctl) * (1 - Math.exp(-7/42));
    atl = atl + (load - atl) * (1 - Math.exp(-7/7));
    ctlProj.push(Math.round(ctl*10)/10);
    atlProj.push(Math.round(atl*10)/10);
  });
  return { ctlProj, atlProj };
}

function renderCtlAtlChart(id, ctlReal, atlReal, ctlProj, atlProj) {
  const el = document.getElementById(id);
  if (!el) return;
  const all = [...ctlReal, ...atlReal, ...ctlProj, ...atlProj];
  const max = Math.max(...all) * 1.1;
  const w = 1000, h = 180;
  const totalPoints = ctlReal.length + ctlProj.length - 1;
  const stepX = w / totalPoints;
  const toY = v => h - (v / max) * h;
  const toPoints = (arr, offset) => arr.map((v, i) => `${(offset+i)*stepX},${toY(v)}`).join(" ");
  const ctlRealPts = toPoints(ctlReal, 0);
  const atlRealPts = toPoints(atlReal, 0);
  const ctlProjPts = toPoints(ctlProj, ctlReal.length - 1);
  const atlProjPts = toPoints(atlProj, atlReal.length - 1);
  el.innerHTML = `<svg viewBox="0 0 ${w} ${h}" preserveAspectRatio="none">
    <polyline class="ctl-line" points="${ctlRealPts}"/>
    <polyline class="atl-line" points="${atlRealPts}"/>
    <polyline class="ctl-proj" points="${ctlProjPts}"/>
    <polyline class="atl-proj" points="${atlProjPts}"/>
  </svg>`;
}

const { ctlProj, atlProj } = projectCtlAtl(CTL_REAL[CTL_REAL.length-1], ATL_REAL[ATL_REAL.length-1], PLANNED_HOURS_NEXT);
renderCtlAtlChart("chart-ctl-atl", CTL_REAL, ATL_REAL, ctlProj, atlProj);
```

- [ ] **Step 4: Vérifier dans le navigateur**

Ouvrir `dashboard.html` sur l'onglet Vue jour, confirmer qu'un graphique avec deux lignes pleines (teal et orange) puis deux lignes en pointillés apparaît sous les phases, sans erreur dans la console (F12).

- [ ] **Step 5: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add CTL/ATL chart with plan-based projection to Vue jour"
```

---

### Task 3: Totaux — sélecteur de période, répartition par sport, prévu vs réel

**Files:**
- Modify: `dashboard.html` (page `#page-totaux`)

**Interfaces:**
- Consumes: rien d'une autre tâche.
- Produces: rien consommé ailleurs.

- [ ] **Step 1: Remplacer le contenu de `#page-totaux` par une version avec sélecteur de période**

Chercher le bloc `<div class="page" id="page-totaux">` et remplacer tout son contenu interne par :

```html
<div class="page" id="page-totaux">
  <div class="page-header"><h2>Totaux</h2>
    <select id="totaux-period">
      <option value="8">8 dernières semaines</option>
      <option value="12">12 dernières semaines</option>
      <option value="21">21 dernières semaines</option>
    </select>
  </div>
  <div class="grid">
    <div class="card" style="grid-column: span 12;">
      <h3>Volume hebdomadaire par sport (heures)</h3>
      <div class="chart" id="chart-sport-split" style="height:160px"></div>
      <div class="zone-legend" style="margin-top:10px; border-top:none; padding-top:0;">
        <div class="lg"><span class="sw" style="background:var(--accent)"></span> Course</div>
        <div class="lg"><span class="sw" style="background:var(--accent-2)"></span> Vélo</div>
        <div class="lg"><span class="sw" style="background:var(--warn)"></span> Natation</div>
      </div>
      <div class="note" id="totaux-gap-note" style="margin-top:8px"></div>
    </div>
    <div class="card" style="grid-column: span 12;">
      <h3>Prévu vs réel (semaines couvertes par un bloc)</h3>
      <table id="prevu-reel-table"></table>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Ajouter les données réelles par sport et par semaine dans le script**

Ajouter avant la fonction `renderChart` existante :

```javascript
// Volume hebdo par sport, semaines ISO 2026 (intervals.icu). 0 = pas de donnée fiable (source Strava masquée avant mi-avril).
const WEEKLY_SPORT = {
  labels: ["S7","S9","S10","S11","S12","S13","S14","S16","S17","S18","S19","S20","S23","S24","S25","S26","S27","S28","S29","S30"],
  course:   [0,   0,   0,   0,   0,   0,   0,   0,   1.6, 4.0, 2.3, 1.5, 0,   2.0, 2.9, 2.8, 3.9, 1.5, 4.5, 1.7],
  velo:     [0.9, 0.5, 1.1, 1.0, 1.0, 0.7, 1.2, 1.0, 5.3, 3.4, 3.8, 0.8, 1.5, 4.0, 2.2, 5.9, 2.8, 4.5, 4.1, 0.0],
  natation: [0,   0,   0,   0,   0,   0,   0,   0,   0,   1.6, 1.2, 0.8, 0,   0.9, 1.4, 0,   0.6, 1.2, 0,   0]
};

function renderSportSplit(id, periodWeeks) {
  const el = document.getElementById(id);
  if (!el) return;
  const n = WEEKLY_SPORT.labels.length;
  const start = Math.max(0, n - periodWeeks);
  const labels = WEEKLY_SPORT.labels.slice(start);
  const course = WEEKLY_SPORT.course.slice(start);
  const velo = WEEKLY_SPORT.velo.slice(start);
  const nat = WEEKLY_SPORT.natation.slice(start);
  const totals = labels.map((_, i) => course[i] + velo[i] + nat[i]);
  const max = Math.max(...totals, 1);
  el.innerHTML = labels.map((lab, i) => {
    const hC = Math.round((course[i] / max) * 130);
    const hV = Math.round((velo[i] / max) * 130);
    const hN = Math.round((nat[i] / max) * 130);
    return `<div class="bar-col">
      <div class="bar-val">${totals[i].toFixed(1)}h</div>
      <div style="display:flex; flex-direction:column; width:100%;">
        <div style="height:${hN}px; background:var(--warn); width:100%;"></div>
        <div style="height:${hV}px; background:var(--accent-2); width:100%;"></div>
        <div style="height:${hC}px; background:var(--accent); width:100%;"></div>
      </div>
      <div class="bar-lab">${lab}</div>
    </div>`;
  }).join("");
  const gapNote = document.getElementById("totaux-gap-note");
  if (gapNote) {
    gapNote.textContent = start < 8 ? "Semaines avant mi-avril 2026 : répartition par sport incomplète (données synchronisées via Strava à l'époque, masquées par leur API) — le total course/vélo/natation de ces semaines-là est sous-estimé." : "";
  }
}

// Volume prévu (course, heures) par semaine, tiré des fichiers de blocs
const PLANNED_VS_REAL = [
  { week: "S27 (13-19/07)", planned: null, actual: 3.9 },
  { week: "S28 (20-26/07)", planned: null, actual: 1.5 },
  { week: "S29 (27/07-02/08)", planned: 8.0, actual: null },
];
function renderPrevuReel(id) {
  const el = document.getElementById(id);
  if (!el) return;
  const rows = PLANNED_VS_REAL.map(r => {
    const p = r.planned !== null ? r.planned.toFixed(1) + "h" : "—";
    const a = r.actual !== null ? r.actual.toFixed(1) + "h" : "à venir";
    let delta = "—";
    if (r.planned !== null && r.actual !== null) {
      const pct = Math.round(((r.actual - r.planned) / r.planned) * 100);
      delta = (pct >= 0 ? "+" : "") + pct + "%";
    }
    return `<tr><td>${r.week}</td><td>${p}</td><td>${a}</td><td>${delta}</td></tr>`;
  }).join("");
  el.innerHTML = `<tr><th>Semaine</th><th>Prévu</th><th>Réel</th><th>Écart</th></tr>${rows}`;
}

renderSportSplit("chart-sport-split", 8);
renderPrevuReel("prevu-reel-table");
document.getElementById("totaux-period").addEventListener("change", e => {
  renderSportSplit("chart-sport-split", parseInt(e.target.value, 10));
});
```

- [ ] **Step 3: Nettoyer l'ancien graphique volume/dénivelé devenu obsolète**

Le HTML de `#page-totaux` remplacé à l'étape 1 n'a plus les éléments `id="chart-hours"` et `id="chart-vert"`. Dans le script, supprimer les lignes devenues mortes (elles ne provoquent pas d'erreur grâce au `if (!el) return;` de `renderChart`, mais autant nettoyer) :

```javascript
// À supprimer :
const hours = [1.5, 6.9, 7.1, 9.2, 8.0, 7.2, 9.2, 1.7];
const hoursLabels = ["S23","S24","S25","S26","S27","S28","S29","S30"];
const vert = [1039, 801, 713, 1327, 1006, 1245, 1248, 138];
// ...
renderChart("chart-hours", hours, hoursLabels, "hours", "h");
renderChart("chart-vert", vert, hoursLabels, "vertm", "");
```

Garder la fonction `renderChart` elle-même si rien d'autre ne l'utilise n'est pas nécessaire — elle peut rester inutilisée sans casser quoi que ce soit, mais la supprimer aussi si l'implémenteur préfère un fichier plus propre.

- [ ] **Step 4: Vérifier dans le navigateur**

Ouvrir l'onglet Totaux, confirmer que les barres empilées s'affichent avec 3 couleurs, que changer le sélecteur de période (8/12/21) met à jour le graphique, et que la note sur les données incomplètes apparaît en 21 semaines.

- [ ] **Step 5: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add period selector, sport split, and planned-vs-actual to Totaux"
```

---

### Task 4: Historique — type de séance et détail structuré

**Files:**
- Modify: `dashboard.html` (page `#page-historique`)

**Interfaces:**
- Consumes: rien d'une autre tâche.
- Produces: rien consommé ailleurs.

- [ ] **Step 1: Remplacer la table existante de `#page-historique` par une version enrichie**

Remplacer le contenu du `<table>` existant dans `#page-historique` par :

```html
<table>
  <tr><th>Date</th><th>Type</th><th>Distance</th><th>Allure</th><th>FC</th><th>Détail</th></tr>
  <tr><td>23/07</td><td>Facile</td><td>8,4 km</td><td>5:40/km</td><td><span class="hr-tag hr-z1">118</span></td><td>—</td></tr>
  <tr><td>20/07</td><td>Facile</td><td>10,0 km</td><td>5:15/km</td><td><span class="hr-tag hr-z1">123</span></td><td>—</td></tr>
  <tr><td>19/07</td><td>Longue</td><td>21,9 km</td><td>4:11/km</td><td><span class="hr-tag hr-z3">150</span></td><td>—</td></tr>
  <tr><td>18/07</td><td>Facile</td><td>9,3 km</td><td>5:29/km</td><td><span class="hr-tag hr-z1">121</span></td><td>—</td></tr>
  <tr><td>16/07</td><td>Facile</td><td>9,7 km</td><td>5:17/km</td><td><span class="hr-tag hr-z1">125</span></td><td>—</td></tr>
  <tr><td>13/07</td><td>Facile</td><td>14,5 km</td><td>5:25/km</td><td><span class="hr-tag hr-z2">128</span></td><td>—</td></tr>
  <tr><td>28/07 (prévu)</td><td>Qualité</td><td>~13,4 km</td><td>seuil 3:33/km</td><td>—</td><td><a href="training/2026-07-27-week-plan.md" style="color:var(--accent-2)">voir détail</a></td></tr>
</table>
<div class="note" style="margin-top:10px">Le type de séance ("Qualité"/"Longue") est déduit du contenu (bloc en zone seuil ou distance >18km) — un lien "voir détail" apparaît seulement pour les séances qui ont un fichier de semaine structuré (échauffement/corps/récup).</div>
```

- [ ] **Step 2: Vérifier dans le navigateur**

Ouvrir l'onglet Historique, confirmer que la colonne Type distingue bien Facile/Longue/Qualité, et que le lien "voir détail" sur la ligne du 28/07 ouvre bien `training/2026-07-27-week-plan.md`.

- [ ] **Step 3: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add session type and detail links to Historique"
```

---

### Task 5: Compétences — dernière exécution et action copiable

**Files:**
- Modify: `dashboard.html` (page `#page-competences`)

**Interfaces:**
- Consumes: dates réelles des fichiers `reviews/2026-07-13.md`, `training/2026-08-03-week-plan.md`, `races/2026-10-18-amsterdam-fueling-pacing-plan.md`.
- Produces: rien consommé ailleurs.

- [ ] **Step 1: Remplacer les 4 tuiles existantes dans `.skills-grid`**

```html
<div class="skills-grid">
  <div class="tile">
    <div class="name">plan-my-week</div>
    <div class="desc">Construit la semaine à venir à partir du plan de phase + données réelles.</div>
    <div class="desc" style="margin-top:8px; color:var(--text)">Dernière semaine construite : <a href="training/2026-08-03-week-plan.md" style="color:var(--accent-2)">03/08</a></div>
    <div class="desc" style="font-family:monospace; font-size:10.5px; margin-top:6px">Demande : "lance plan-my-week"</div>
  </div>
  <div class="tile">
    <div class="name">weekly-review</div>
    <div class="desc">Analyse une semaine passée: exécution, ressenti vs données, tendances.</div>
    <div class="desc" style="margin-top:8px; color:var(--text)">Dernière review : <a href="reviews/2026-07-13.md" style="color:var(--accent-2)">13/07</a></div>
    <div class="desc" style="font-family:monospace; font-size:10.5px; margin-top:6px">Demande : "lance weekly-review"</div>
  </div>
  <div class="tile">
    <div class="name">fuel-and-pace</div>
    <div class="desc">Plan fueling/pacing chiffré pour une course donnée.</div>
    <div class="desc" style="margin-top:8px; color:var(--text)">Dernier plan : <a href="races/2026-10-18-amsterdam-fueling-pacing-plan.md" style="color:var(--accent-2)">Amsterdam (90g/h glucides)</a></div>
    <div class="desc" style="font-family:monospace; font-size:10.5px; margin-top:6px">Demande : "lance fuel-and-pace pour [course]"</div>
  </div>
  <div class="tile">
    <div class="name">recovery-check</div>
    <div class="desc">Appel de forme quotidien (GO HARD/EASY/REST), programmé chaque matin à 7h07.</div>
    <div class="desc" style="margin-top:8px; color:var(--text)">Aucun flag notable pour l'instant (health/ vide) — signe que tout va bien, pas un manque de données.</div>
    <div class="desc" style="font-family:monospace; font-size:10.5px; margin-top:6px">Demande : "lance recovery-check"</div>
  </div>
</div>
```

- [ ] **Step 2: Vérifier dans le navigateur**

Ouvrir l'onglet Compétences, confirmer que les 4 tuiles affichent bien une info de dernière exécution et une ligne de commande copiable, et que les liens ouvrent les bons fichiers.

- [ ] **Step 3: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add last-execution info and copy-paste actions to skill tiles"
```

---

### Task 6: Feuille de route — volumes réels par bloc

**Files:**
- Modify: `dashboard.html` (page `#page-roadmap`)

**Interfaces:**
- Consumes: volumes min/pic/fin de chaque fichier de bloc (`training/2026-08-17-amsterdam-marathon-block.md` et les 4 nouveaux fichiers).
- Produces: rien consommé ailleurs.

- [ ] **Step 1: Lire les 5 fichiers de blocs pour confirmer les volumes à afficher**

```bash
grep -h "de course" "C:/OpenClaw/Athlete OS/training/2026-08-17-amsterdam-marathon-block.md" "C:/OpenClaw/Athlete OS/training/2026-10-19-recup-post-amsterdam.md" "C:/OpenClaw/Athlete OS/training/2026-11-02-bloc-semi-boulogne.md" "C:/OpenClaw/Athlete OS/training/2026-11-16-recup-base.md" "C:/OpenClaw/Athlete OS/training/2026-12-20-bloc-marathon-seville.md"
```

- [ ] **Step 2: Remplacer chaque `<div class="stop ...">` de la frise avec un résumé de volume**

Remplacer le contenu de `.roadmap-track` par :

```html
<div class="stop current"><div class="dot2"></div><div class="date">17/08 → 18/10</div><div class="label">Bloc marathon Amsterdam</div><div class="zsub">2h30 → 4h45 pic → 1h30</div><span class="obj a">A</span></div>
<div class="stop"><div class="dot2"></div><div class="date">19/10 → 01/11</div><div class="label">Récup post-Amsterdam</div><div class="zsub">1h → 2h30</div></div>
<div class="stop"><div class="dot2"></div><div class="date">02/11 → 15/11</div><div class="label">Semi Boulogne-Billancourt</div><div class="zsub">3h30 → 1h30</div><span class="obj b">B</span></div>
<div class="stop"><div class="dot2"></div><div class="date">16/11 → 20/12</div><div class="label">Récup + base légère</div><div class="zsub">1h30 → 3h30</div></div>
<div class="stop"><div class="dot2"></div><div class="date">21/12 → 21/02/27</div><div class="label">Bloc marathon Séville</div><div class="zsub">4h → 5h pic → 1h30</div></div>
<div class="stop goal-a"><div class="dot2"></div><div class="date">21/02/2027</div><div class="label">Marathon de Séville</div><span class="obj a">A</span></div>
```

- [ ] **Step 3: Vérifier dans le navigateur**

Ouvrir l'onglet Feuille de route, confirmer que chaque arrêt affiche maintenant une ligne de volume sous son label.

- [ ] **Step 4: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add real block volumes to roadmap timeline"
```

---

### Task 7: Vue jour — lien vers le bloc actif et projection à 2 semaines

**Files:**
- Modify: `dashboard.html` (page `#page-jour`, carte `.phases`)

**Interfaces:**
- Consumes: `training/2026-08-17-amsterdam-marathon-block.md` (bloc actif), `training/2026-08-03-week-plan.md` (semaine +2, ~7h cible, Annecy/Embrun).

- [ ] **Step 1: Ajouter un lien et un encart après la carte `.phases` existante**

Insérer juste après la fermeture de `<div class="card phases">...</div>` :

```html
<div class="card" style="grid-column: span 12;">
  <h3>Dans 2 semaines (03-09/08)</h3>
  <p style="margin:0; font-size:13px; color:var(--text)">Tu seras à Annecy puis en route vers Embrun, semaine à ~7h cible (-12% volontaire, on réduit avant le Quart Embrunman du 15/08 plutôt que d'ajouter du volume). <a href="training/2026-08-03-week-plan.md" style="color:var(--accent-2)">Voir le détail de cette semaine</a>.</p>
</div>
```

- [ ] **Step 2: Vérifier dans le navigateur**

Ouvrir l'onglet Vue jour, confirmer que le nouvel encart apparaît sous les phases du bloc marathon et que le lien fonctionne.

- [ ] **Step 3: Commit**

```bash
cd "C:/OpenClaw/Athlete OS"
git add dashboard.html
git commit -m "Add 2-week-ahead preview link to Vue jour"
```

---

### Task 8: Vérification finale et publication

**Files:**
- Read only: `dashboard.html`

- [ ] **Step 1: Parcourir les 6 onglets dans le navigateur**

Ouvrir `dashboard.html`, cliquer sur chaque onglet (Vue jour, Historique, Zones, Totaux, Compétences, Feuille de route), confirmer l'absence d'erreur JS dans la console (F12) et que chaque nouvelle section affichée dans les tâches précédentes est bien visible.

- [ ] **Step 2: Pousser sur GitHub si l'athlète le demande**

```bash
cd "C:/OpenClaw/Athlete OS"
git log --oneline -8
```
Attendre la confirmation explicite avant `git push origin main`.
