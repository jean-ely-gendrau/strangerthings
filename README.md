# Stranger Things Fan Lab

Mini site sur l'univers de Stranger Things pour apprendre la **réécriture d'URL** avec **Plesk / Apache** (`mod_rewrite`).

---

## Le principe

Sur un serveur web, les fichiers sont rangés dans des dossiers techniques (ici `/strangerthings/`).
Mais pour l'utilisateur, tu veux afficher des URLs propres et lisibles (ici `/hawkins/...`).

C'est exactement ce que fait la **réécriture d'URL** : elle traduit une URL "jolie" en chemin réel vers le bon fichier, sans que l'utilisateur ne s'en rende compte.

---

## Structure du projet

```
racine du vhost
├── .htaccess                       # les règles de réécriture
└── strangerthings/                 # les fichiers réels du site
    ├── assets/
    │   ├── style.css
    │   └── images/
    │       ├── hero-accueil.jpg      # hero accueil
    │       ├── hawkings_card.png    # illustration section code
    │       ├── carte-eleven.jpg
    │       ├── carte-groupe.jpg
    │       ├── carte-vecna.jpg
    │       ├── carte-hawkins.jpg
    │       ├── carte-upside-down.jpg
    │       └── carte-laboratoire.jpg
    ├── index.html                  # page d'accueil
    ├── personnages.html            # page personnages
    ├── lieux.html                  # page lieux & univers
    └── contact.html                # page contact
```

---

## Le fichier .htaccess

C'est le fichier clé. Il se place **à la racine du vhost** (pas dans le dossier `strangerthings/`).

Voici son contenu complet :

```apacheconf
# On active le moteur de réécriture d'Apache
RewriteEngine On

# CSS principal
# L'utilisateur demande : /hawkins/assets/style.css
# Le fichier réel est :   /strangerthings/assets/style.css
RewriteRule ^hawkins/assets/style\.css$ /strangerthings/assets/style.css [L]

# Favicon
RewriteRule ^hawkins/assets/favicon\.svg$ /strangerthings/assets/favicon.svg [L]

# Images
# Toutes les images dans /hawkins/assets/images/ pointent vers /strangerthings/assets/images/
RewriteRule ^hawkins/assets/images/(.+)$ /strangerthings/assets/images/$1 [L]

# Pages statiques
RewriteRule ^hawkins/?$                /strangerthings/index.html        [L]
RewriteRule ^hawkins/personnages/?$    /strangerthings/personnages.html  [L]
RewriteRule ^hawkins/lieux/?$          /strangerthings/lieux.html        [L]
RewriteRule ^hawkins/contact/?$        /strangerthings/contact.html      [L]
```

### Décortiquons une règle

Prenons cette ligne :

```apacheconf
RewriteRule ^hawkins/personnages/?$ /strangerthings/personnages.html [L]
```

| Partie | Rôle |
|--------|------|
| `RewriteRule` | Directive Apache qui définit une règle de réécriture |
| `^hawkins/personnages/?$` | L'URL que tu tapes dans ton navigateur (le `?` rend le `/` final optionnel) |
| `/strangerthings/personnages.html` | Le fichier réel que le serveur va charger |
| `[L]` | Flag "Last" — Apache arrête de chercher d'autres règles après celle-ci |

### Pourquoi réécrire aussi les assets (CSS, images, favicon) ?

La réécriture ne concerne pas que les pages HTML. Si ton HTML reference `/hawkins/assets/style.css`, le navigateur va demander ce fichier au serveur. Mais ce chemin n'existe pas physiquement — le vrai fichier est dans `/strangerthings/assets/style.css`.

Sans règle de réécriture pour les assets, tu obtiens une **erreur 404** sur le CSS, les images et le favicon : ton site s'affiche sans style, sans images et sans icône d'onglet.

Tu peux le vérifier toi-même :
1. Commente la règle du CSS dans le `.htaccess` (mets un `#` devant)
2. Recharge la page — tu verras le HTML brut, sans aucune mise en forme
3. Ouvre les DevTools (F12 > Console) — tu verras l'erreur 404 sur `style.css`

C'est la même logique pour les images (`(.+)$` capture n'importe quel nom de fichier dans le dossier) et le favicon.

### Ce que tu dois retenir

- `^` = début de l'URL, `$` = fin de l'URL. Ensemble, ils garantissent une correspondance **exacte**.
- `/?` = le slash final est **optionnel**. Donc `/hawkins/personnages` et `/hawkins/personnages/` fonctionnent tous les deux.
- `(.+)` = capture **un ou plusieurs caractères**. Ca permet de réécrire tout un dossier d'un coup sans lister chaque fichier.
- `[L]` = si cette règle matche, Apache **s'arrête là** et ne teste pas les règles suivantes.
- `RewriteEngine On` doit apparaître **une seule fois**, en haut du fichier.

---

## Comment tester

1. Place le dossier `strangerthings/` et le `.htaccess` à la racine de ton vhost sur Plesk.
2. Ouvre ton navigateur et va sur `ton-domaine.com/hawkins/`.
3. Navigue entre les pages — regarde l'URL dans la barre d'adresse : tu ne vois jamais `/strangerthings/`.
4. Ouvre les DevTools (F12 > Réseau) et observe les requêtes : le serveur sert bien les fichiers depuis `/strangerthings/`.

---

## Aller plus loin

Maintenant que tu as compris le principe, tu peux :

- Ajouter une nouvelle page (ex: `theories.html`) et écrire la règle de réécriture correspondante
- Tester ce qui se passe si tu **supprimes** une règle du `.htaccess`
- Regarder les **logs d'accès Apache** dans Plesk pour voir les requêtes réelles
- Explorer les autres flags disponibles : `[R=301,L]` pour une redirection, `[NC]` pour ignorer la casse
