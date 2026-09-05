# Dark Matter — Dossier de traduction

Site statique qui compare le texte original des cartes à leurs traductions DE/FR.
`index.html` ne contient **aucune donnée** : il peut charger les JSON de deux façons.

## Deux façons d'alimenter le site

**1. Depuis un dépôt GitHub (recommandé)** — dans le panneau "⚙ Source des données"
en haut de la barre latérale, collez l'URL d'un dossier GitHub contenant vos exports
(ex. `https://github.com/votre-compte/votre-repo/tree/main/data`) et cliquez
*Charger*. La page :
- liste les fichiers `.json` de ce dossier via l'API GitHub,
- ouvre chacun et regarde sa structure pour deviner son rôle : celui dont `cards`
  est une liste = le projet de base ; ceux dont `cards` est un dictionnaire avec un
  champ `language` = des traductions (le nom de la langue est déduit du code
  `language`, pas du nom de fichier),
- charge tout automatiquement, sans que vous ayez à nommer les fichiers.

L'URL choisie est mémorisée (dans le navigateur) et ajoutée à l'adresse de la page
(`?source=...`), donc un lien que vous partagez rouvre directement la même source.

**Mettre à jour** = pousser (`git push`) de nouveaux exports JSON dans ce dossier
GitHub. Rien à réuploader sur Cloudflare : la page relit GitHub à chaque visite.
Comptez jusqu'à une minute avant que GitHub serve la nouvelle version (cache CDN).

Limite à connaître : l'API GitHub autorise 60 requêtes/heure par visiteur non identifié
(largement suffisant pour un usage perso, mais à savoir si plusieurs personnes
rechargent souvent). Le dépôt doit être public — les dépôts privés demandent un jeton
d'accès, non géré par cette version (dites-le-moi si vous en avez besoin).

**2. Fichiers déposés à côté de index.html (mode de secours)** — si vous cliquez
*Fichiers locaux* ou n'entrez aucune source GitHub, la page essaie de charger
`project.json`, `project_de.json`, `project_fr.json` dans son propre dossier. C'est le
mode utilisé par défaut si le dossier contient déjà ces fichiers (comme celui-ci).

## Héberger sur Cloudflare Pages (gratuit)

**Option A — glisser-déposer**
1. https://dash.cloudflare.com → *Workers & Pages* → *Create* → onglet *Pages* → *Upload assets*.
2. Donnez un nom au projet (ça devient `<nom>.pages.dev`).
3. Si vous utilisez le mode GitHub : glissez uniquement `index.html`. Si vous utilisez
   le mode fichiers locaux : glissez `index.html` + les 3 JSON.
4. *Deploy site*.

**Option B — via un dépôt Git**
1. Mettez ce dossier (au moins `index.html`) dans un dépôt GitHub/GitLab.
2. Cloudflare Pages → *Create* → *Connect to Git*, sélectionnez le dépôt.
3. Build command : vide. Output directory : `/`.
4. Chaque `git push` republie automatiquement.

Astuce : si vos JSON de traduction vivent déjà dans un dépôt Git (ce que permet le
mode GitHub ci-dessus), vous pouvez très bien héberger `index.html` seul sur
Cloudflare Pages et laisser toutes les données dans ce dépôt — les deux n'ont pas
besoin d'être au même endroit.

## Ajouter une langue (ex. espagnol)

**Mode GitHub** : rien à faire dans `index.html`. Déposez simplement `project_es.json`
dans le même dossier GitHub — son champ `"language": "es"` suffit à ce que la page le
détecte et affiche un onglet "ES" tout seul.

**Mode fichiers locaux** : ouvrez `index.html`, repérez ce bloc en haut du `<script>` :

```js
const LOCAL_PROJECT_FILE = "project.json";
const LOCAL_LANGS = [
  { code: "de", file: "project_de.json" },
  { code: "fr", file: "project_fr.json" }
];
```

et ajoutez une ligne, par exemple `{ code: "es", file: "project_es.json" }`, puis
déposez `project_es.json` à côté des autres.

## Tester en local avant de déployer

Comme la page charge les JSON via `fetch`, ouvrir `index.html` en double-cliquant
(`file://`) ne fonctionnera pas. Servez le dossier :

```
python3 -m http.server 8000
```

puis ouvrez `http://localhost:8000/index.html`.
