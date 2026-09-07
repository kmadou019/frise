# Frise des savants

Application web responsive pour noter des savants (musulmans ou autres) avec leur date de décès, classés automatiquement par ordre chronologique. Les données sont stockées directement dans un fichier du dépôt Git (`data/scholars.json`) et synchronisées automatiquement entre appareils, sans configuration une fois le déploiement fait.

## ⚠️ À lire avant de déployer

La synchronisation utilise un **jeton d'accès personnel GitHub**, pour qu'aucun appareil n'ait besoin de le saisir. Ce jeton n'est **jamais commité dans le dépôt** : il est stocké comme secret chiffré GitHub Actions et injecté dans `index.html` uniquement au moment du déploiement, par le workflow `.github/workflows/deploy.yml`. Le fichier source (celui que tu vois et modifies dans le dépôt) garde toujours le texte `REMPLACE_PAR_TON_TOKEN_GITHUB_ICI` à la place — GitHub bloquerait de toute façon tout push contenant un vrai jeton (*push protection*).

Cela dit, une fois le site déployé, le jeton réel est présent dans le code source **de la page publiée** (HTML/JS servis au navigateur) :

- **L'URL du site déployé ne doit jamais être partagée ni rendue publique.** GitHub Pages sert la page à quiconque connaît l'URL, même si le dépôt Git est privé — n'importe qui visitant le site déployé peut voir le jeton via "Afficher le code source".
- N'utilise cette approche que pour un usage strictement personnel, avec une URL que tu ne communiques à personne.
- Le jeton doit être limité **uniquement à ce dépôt** et avoir seulement la permission d'écrire son contenu (voir ci-dessous) — jamais un jeton avec accès large à ton compte.

Si tu préfères ne pas exposer de jeton du tout, il existe une alternative plus sûre où seule la *lecture* est automatique et l'*écriture* nécessite de saisir un jeton une fois par appareil — demande cette variante si besoin.

## Mise en place

### 1. Créer un jeton d'accès personnel (fine-grained)

1. Va sur [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new).
2. **Repository access** → *Only select repositories* → choisis ce dépôt (`frise`).
3. **Permissions** → *Repository permissions* → **Contents : Read and write**. Aucune autre permission n'est nécessaire.
4. Génère le jeton et copie-le (il ne sera plus affiché ensuite).

### 2. Enregistrer le jeton comme secret GitHub Actions

1. Dans le dépôt : **Settings → Secrets and variables → Actions → New repository secret**.
2. Nom : `SYNC_TOKEN`. Valeur : colle le jeton généré à l'étape 1.
3. Enregistre. Le jeton n'apparaît plus jamais en clair nulle part dans le dépôt.

Tu n'as **rien à modifier dans `index.html`** — le placeholder `REMPLACE_PAR_TON_TOKEN_GITHUB_ICI` doit rester tel quel dans le code source.

### 3. Activer GitHub Pages via Actions

1. **Settings → Pages → Source : "GitHub Actions"** (pas "Deploy from a branch").
2. ⚠️ Sur le plan GitHub Free, Pages ne fonctionne qu'avec un dépôt **public** — passer en privé nécessite GitHub Pro. Dans les deux cas, le site publié reste accessible publiquement à quiconque a l'URL.

### 4. Déployer

Committe et pousse (le fichier ne contient jamais de secret, donc rien ne bloque le push). Le workflow `.github/workflows/deploy.yml` se déclenche automatiquement à chaque push sur `main` : il copie `index.html`, y injecte le vrai jeton depuis le secret `SYNC_TOKEN`, puis publie le résultat sur GitHub Pages. Tu peux aussi le relancer manuellement depuis l'onglet **Actions → Déploiement GitHub Pages → Run workflow**.

À la première utilisation, le fichier `data/scholars.json` n'existe pas encore dans le dépôt : l'app démarre avec une liste vide et le crée automatiquement au premier ajout (premier commit, effectué directement par l'app via l'API GitHub — ce commit-là ne repasse pas par le workflow de déploiement, il est ignoré par celui-ci pour éviter un redéploiement inutile à chaque ajout).

## Fonctionnement de la synchronisation

- À l'ouverture, l'app lit `data/scholars.json` via l'API GitHub (Contents API) et l'affiche.
- Chaque ajout, suppression ou remise à zéro déclenche un commit automatique sur le dépôt via cette même API.
- Une copie est gardée en `localStorage` pour un affichage instantané et un fonctionnement hors ligne — en cas de perte de connexion, les modifications restent en local jusqu'au prochain succès de synchronisation.
- Sync "dernier écrit gagne" : si deux appareils modifient la liste hors ligne en même temps, la dernière sauvegarde écrase l'autre. Pour un usage personnel occasionnel, ce n'est en pratique pas gênant.
- Le bouton "Synchronisation" en haut à droite force une relecture manuelle du fichier distant.

## Affichage et zoom

- **Téléphone** (largeur < 768px) : frise verticale.
- **Tablette et ordinateur** (≥ 768px) : frise horizontale, un bloc par siècle, défilement latéral si besoin.
- Boutons `−` / `+` au-dessus de la frise pour zoomer/dézoomer (70 % à 200 %), niveau mémorisé par appareil.
