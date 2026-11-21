<img width="960" height="600" alt="demo" src="https://github.com/user-attachments/assets/bbc7558d-b97d-4462-a210-43b1c80868f6" />


# Inspecteur-de-design
Extension Chrome pour designers et développeurs : au survol, elle affiche la typo, les couleurs et les espacements d'un élément, avec son box model. Elle calcule le contraste WCAG (AA/AAA), permet de figer et copier les valeurs, de naviguer dans le DOM au clavier et d'extraire la palette de la page en variables CSS. JavaScript, Manifest V3.

# Inspecteur de design

![Extension Chrome](https://img.shields.io/badge/Chrome-extension-6D5DFC?logo=googlechrome&logoColor=white)
![Manifest V3](https://img.shields.io/badge/Manifest-V3-6D5DFC)
![JavaScript](https://img.shields.io/badge/JavaScript-sans%20framework-F7DF1E?logo=javascript&logoColor=black)
![Dépendances](https://img.shields.io/badge/d%C3%A9pendances-0-2ea44f)

Extension Chrome pour designers et développeurs : au survol, elle affiche la typo, les couleurs et les espacements d'un élément, avec son box model. Elle calcule le contraste WCAG (AA/AAA), permet de figer et copier les valeurs, de naviguer dans le DOM au clavier et d'extraire la palette de la page en variables CSS.

![Démo : survol, figer et copier, navigation au clavier, contraste et palette de la page](docs/demo.gif)

## Fonctionnalités

- **Survol** : police, taille, graisse, interligne, couleurs (en hexadécimal), margin, padding, arrondi, et box model dessiné sur la page.
- **Contraste WCAG** : ratio entre le texte et le fond réellement visible, avec un verdict AA / AAA (critères repris par le RGAA).
- **Figer et copier** : un clic fige l'infobulle, un clic sur une valeur la copie.
- **Navigation au clavier** : ↑ pour remonter au parent, ↓ pour descendre au premier enfant.
- **Palette de la page** : les couleurs les plus utilisées, à copier une par une ou en variables CSS.
- **Permissions minimales** : l'extension n'accède qu'à l'onglet où on l'active (`activeTab`).

## Installation

1. Télécharger ou cloner ce dépôt.
2. Ouvrir `chrome://extensions` (fonctionne aussi dans Edge, Brave, Opera, Arc…).
3. Activer le **Mode développeur** (en haut à droite).
4. Cliquer sur **Charger l'extension non empaquetée** et choisir le dossier du projet (celui qui contient `manifest.json`).

## Utilisation

- Cliquer sur l'icône, puis sur **Activer l'inspecteur**, ou utiliser le raccourci **Alt+Shift+D**.
- Survoler n'importe quel élément de la page.
- **Clic** pour figer l'infobulle, puis clic sur une valeur pour la copier. Un nouveau clic sur la page la libère.
- **↑ / ↓** pour passer au parent ou au premier enfant de l'élément inspecté.
- **Échap** pour quitter. Le badge **ON** sur l'icône indique que l'inspecteur est actif dans l'onglet.
- La **palette** de la page s'affiche dans la popup : clic sur une pastille pour copier la couleur.
- Le raccourci se modifie dans `chrome://extensions/shortcuts`.

## Choix techniques

- **JavaScript sans framework ni étape de build** : l'extension se charge telle quelle dans Chrome, sans aucune dépendance.
- **Permissions minimales** (`activeTab` + `scripting`) : le script n'est injecté qu'à la demande, dans l'onglet où on active l'extension.
- **Shadow DOM** : l'infobulle est isolée du CSS de la page inspectée, et inversement.
- **Rendu fluide** : l'affichage est recalculé au plus une fois par image, avec `requestAnimationFrame`.
- **Messages entre les parties de l'extension** : `TOGGLE`, `GET_STATE`, `GET_PALETTE` et `STATE_CHANGED` relient la popup, le service worker et le script injecté.
- **Contraste selon WCAG 2** : fond réellement visible (superposition des fonds semi-transparents), luminance relative, seuils AA / AAA adaptés au gros texte. Les couleurs modernes comme `oklch()` sont converties grâce à un canvas.

## Architecture

```
inspecteur-design/
├── manifest.json             → nom, permissions, popup, raccourci clavier
├── background.js             → service worker : raccourci clavier + badge ON
├── lib/inspector-control.js  → injecte le script dans l'onglet et lui envoie les messages
├── content/inspector.js      → le cœur du projet : box model, infobulle, contraste, palette
├── popup/                    → la fenêtre qui s'ouvre au clic sur l'icône
├── icons/
└── docs/                     → GIF de démo et notes de développement
```

```
Popup (bouton, palette) ──┐
                          ├──► lib/inspector-control.js
Raccourci Alt+Shift+D ────┘        │ 1. injecte content/inspector.js dans l'onglet
  (background.js)                  │ 2. envoie TOGGLE ou GET_PALETTE
                                   ▼
                         content/inspector.js
                           │ STATE_CHANGED             │ réponse : couleurs de la page
                           ▼                           ▼
                  background.js → badge ON       popup → pastilles
```

## Journal de progression

| Étape | Contenu | Statut |
| --- | --- | --- |
| 1 | Base de l'extension : survol, infobulle, box model | ✅ 20/11/2026 |
| 2 | Figer avec un clic, copier les valeurs | ✅ 20/11/2026 |
| 3 | Navigation au clavier dans le DOM | ✅ 20/11/2026 |
| 4 | Palette de couleurs de la page | ✅ 20/11/2026 |
| 5 | Guide typographique de la page | ⏳ À venir |
| 6 | Contraste texte / fond (WCAG, RGAA) | ✅ 21/11/2026 |
| — | GIF de démo et README | ✅ 21/11/2026 |
| 7 | Passage à WXT, TypeScript et Firefox | ⏳ À venir |
| 8 | Publication sur le Chrome Web Store | ⏳ À venir |

### Étape 1 : la base de l'extension

- Extension Manifest V3 en JavaScript, sans outil de build : manifest, popup, service worker et content script.
- Au survol, une infobulle affiche la police, la taille, la graisse, l'interligne, les couleurs en hexadécimal, les margins, les paddings et l'arrondi. Le box model est dessiné par-dessus la page, avec les couleurs des DevTools.
- Activation depuis la popup ou avec **Alt+Shift+D**, badge **ON** sur l'icône, **Échap** pour quitter.
- Permissions `activeTab` + `scripting` : le script n'est injecté que dans l'onglet où on active l'extension, sans accès à tous les sites.
- Notions travaillées : content script et monde isolé, Shadow DOM pour isoler l'interface, `getComputedStyle()`, `getBoundingClientRect()`, `requestAnimationFrame`, messages entre les parties de l'extension.

### Étape 2 : figer et copier

- Un clic fige l'infobulle (contour violet) sans suivre les liens de la page. Un clic sur une valeur la copie, avec un retour « Copié ! ».
- Notions travaillées : écouteur `click` en phase de capture, `preventDefault()` et `stopPropagation()`, API Presse-papiers (`navigator.clipboard`).
- Difficulté rencontrée : le clic ne figeait rien, car la fonction `onClick` existait mais n'était jamais branchée à l'événement (il manquait le `addEventListener`). Autre piège : après une modification, il faut recharger l'extension **et** la page, sinon l'ancien script reste actif dans l'onglet.

### Étape 3 : navigation au clavier

- **↑** sélectionne le parent de l'élément inspecté, **↓** son premier enfant, sans faire défiler la page.
- Pour descendre, les éléments non affichés (comme `<head>`) sont ignorés grâce à `getClientRects()`.
- Combiné au clic pour figer, ça permet d'atteindre un élément recouvert par un autre.

### Étape 4 : palette de couleurs

- La popup affiche les 16 couleurs les plus utilisées dans la page (texte, fond, bordure), triées par fréquence.
- Clic sur une pastille pour copier la couleur, et bouton pour copier toute la palette en variables CSS (`:root { --color-1: … }`).
- La couleur de texte n'est comptée que pour les éléments qui contiennent eux-mêmes du texte, sinon l'héritage CSS fausse le résultat.
- Testé sur une page de 5 000 éléments : moins d'un dixième de seconde.

### Étape 5 : guide typographique (à venir)

- Lister les combinaisons police / taille / graisse utilisées dans la page, et repérer la police réellement affichée.

### Étape 6 : contraste WCAG

- Nouvelle ligne « Contraste » dans l'infobulle : ratio entre le texte et son fond, avec un verdict AA, AAA ou Échec, et des seuils adaptés au gros texte.
- Le fond réellement visible est calculé en remontant les parents et en superposant les fonds semi-transparents.
- Les couleurs modernes (`oklch()`…) sont converties grâce à un canvas de 1 pixel. Sur un fond en image ou en dégradé, le badge indique « À vérifier ».
- Résultats contrôlés avec des valeurs de référence : #767676 sur blanc donne 4,54:1 (AA), #777777 donne 4,47:1 (échec).

## Prochaines étapes

- Guide typographique de la page : polices, tailles et graisses utilisées.
- Passage à WXT et TypeScript, avec une version Firefox.
- Publication sur le Chrome Web Store.

## À propos

Projet personnel réalisé en parallèle de mon BUT MMI (Métiers du multimédia et de l'internet), parcours Développement web et dispositifs interactifs.

Les notes de développement (notions techniques, dépannage, feuille de route détaillée) sont dans [docs/notes-de-developpement.md](docs/notes-de-developpement.md).
