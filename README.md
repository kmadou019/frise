# Frise des savants

Application web responsive pour noter des savants (musulmans ou autres) avec leur date de décès, classés automatiquement par ordre chronologique. Les données sont stockées directement dans un fichier du dépôt Git (`data/scholars.json`) et synchronisées automatiquement entre appareils, sans configuration une fois le déploiement fait.

## ⚠️ À lire avant de déployer

La synchronisation utilise un **jeton d'accès personnel GitHub écrit en dur dans `index.html`** (variable `CONFIG.token`), pour qu'aucun appareil n'ait besoin de le saisir. Conséquence importante :

- **L'URL du site déployé ne doit jamais être partagée ni rendue publique.** GitHub Pages et Vercel servent la page à quiconque connaît l'URL, même si le dépôt Git est privé — n'importe qui visitant le site peut voir le jeton via "Afficher le code source".
- N'utilise cette approche que pour un usage strictement personnel, avec une URL que tu ne communiques à personne.
- Le jeton doit être limité **uniquement à ce dépôt** et avoir seulement la permission d'écrire son contenu (voir ci-dessous) — jamais un jeton avec accès large à ton compte.

Si tu préfères ne pas exposer de jeton du tout, il existe une alternative plus sûre où seule la *lecture* est automatique et l'*écriture* nécessite de saisir un jeton une fois par appareil — demande cette variante si besoin.

## Mise en place

### 1. Créer un jeton d'accès personnel (fine-grained)

1. Va sur [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new).
2. **Repository access** → *Only select repositories* → choisis ce dépôt (`frise`).
3. **Permissions** → *Repository permissions* → **Contents : Read and write**. Aucune autre permission n'est nécessaire.
4. Génère le jeton et copie-le (il ne sera plus affiché ensuite).

### 2. Configurer `index.html`

Ouvre `index.html`, repère le bloc `CONFIG` en haut du `<script>` :

```js
const CONFIG = {
  owner: 'kmadou019',
  repo: 'frise',
  path: 'data/scholars.json',
  branch: 'main',
  token: 'REMPLACE_PAR_TON_TOKEN_GITHUB_ICI'
};
```

Remplace `token` par le jeton généré à l'étape 1. Adapte `owner`/`repo`/`branch` si besoin.

### 3. Déployer

Committe et pousse `index.html` (avec le jeton dedans) sur le dépôt, puis déploie :

- **GitHub Pages** : Settings → Pages → Source : branche `main`, dossier `/root`. ⚠️ Sur le plan GitHub Free, Pages ne fonctionne qu'avec un dépôt **public** — passer en privé nécessite GitHub Pro. Dans les deux cas, le site publié reste accessible publiquement à quiconque a l'URL.
- **Vercel** : Add New → Project → importe le dépôt → Deploy. Aucune configuration nécessaire.

À la première utilisation, le fichier `data/scholars.json` n'existe pas encore dans le dépôt : l'app démarre avec une liste vide et le crée automatiquement au premier ajout (premier commit).

## Fonctionnement de la synchronisation

- À l'ouverture, l'app lit `data/scholars.json` via l'API GitHub (Contents API) et l'affiche.
- Chaque ajout, suppression ou remise à zéro déclenche un commit automatique sur le dépôt via cette même API.
- Une copie est gardée en `localStorage` pour un affichage instantané et un fonctionnement hors ligne — en cas de perte de connexion, les modifications restent en local jusqu'au prochain succès de synchronisation.
- Sync "dernier écrit gagne" : si deux appareils modifient la liste hors ligne en même temps, la dernière sauvegarde écrase l'autre. Pour un usage personnel occasionnel, ce n'est en pratique pas gênant.
- Le bouton "Synchronisation" en haut à droite force une relecture manuelle du fichier distant.
