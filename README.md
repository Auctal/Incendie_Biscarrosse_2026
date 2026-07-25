# Biscarrosse — Carte collaborative de suivi de l'incendie

Site statique (HTML/CSS/JS) affichant une carte Leaflet avec trois couches :
zone touchée par l'incendie, bâtiments détruits confirmés, bâtiments apparaissant
intacts — plus un formulaire permettant aux internautes de faire remonter des
informations (texte + photo/vidéo).

⚠️ **Important : les fichiers GeoJSON fournis dans `/data` sont des exemples
fictifs** (marqués `"exemple": true`) qui servent uniquement à vérifier que la
carte s'affiche correctement. Remplacez-les par vos vraies données avant toute
mise en ligne publique — ne diffusez jamais une information non vérifiée comme
si elle était confirmée.

## 1. Structure du projet

```
biscarrosse-incendie/
├── index.html              page principale
├── css/style.css            styles
├── js/app.js                 logique carte + formulaire
├── data/
│   ├── incendie.geojson             (Incendie.geojson)
│   ├── batiments_detruits.geojson   (Batiment_detruit.geojson)
│   └── batiments_intacts.geojson    (Batiment_intact.geojson)
└── README.md
```

## 2. Mettre à jour les données

Remplacez le contenu des 3 fichiers GeoJSON dans `/data` par vos vraies
données. Gardez les mêmes noms de fichiers (ou modifiez `DATA_URLS` dans
`js/app.js` si vous préférez renommer).

Champs `properties` reconnus par la carte, pour chaque *feature* :

| Champ              | Utilisé pour                         | Exemple                          |
|---------------------|---------------------------------------|-----------------------------------|
| `adresse` / `nom`   | Titre du popup                        | `"12 rue des Pins"`              |
| `description`       | Texte descriptif                      | `"Toiture effondrée, murs debout"`|
| `source`             | Origine de l'information              | `"Vidéo Facebook, 24/07"`        |
| `media_url`          | Lien vers la photo/vidéo source       | `"https://.../photo1.jpg"`       |
| `media_type`         | `"image"` ou `"video"`                | `"image"`                        |
| `date_confirmation`  | Date de constat                       | `"2026-07-24"`                   |

La couche incendie (`incendie.geojson`) doit contenir un ou plusieurs
**polygones** (le périmètre du feu). Les couches bâtiments doivent contenir des
**points** (un point = un bâtiment). Vous pouvez créer/éditer ces fichiers
facilement avec [geojson.io](https://geojson.io) ou QGIS.

### Hébergement des photos/vidéos (`media_url`)

Ce site n'héberge pas lui-même les fichiers média des couches GeoJSON : il
affiche simplement une URL. Solutions simples et gratuites :
- déposer les fichiers dans un dossier `assets/` de ce projet et les servir en
  local (ex. `media_url: "assets/photo1.jpg"`) ;
- ou les héberger sur un service externe (Imgur, un dépôt GitHub, Google
  Drive en partage public, Cloudinary, etc.) et coller le lien direct.

## 3. Fonctionnement du formulaire de contact

Le formulaire (`index.html`, section « Faire une remontée de terrain ») envoie
directement un email — **avec la pièce jointe (photo/vidéo)** — à
**incendie.biscarrosse@proton.me**, sans ouvrir la messagerie de la personne.
Cela fonctionne sans backend ni compte à créer, grâce au service gratuit
[FormSubmit.co](https://formsubmit.co), compatible avec GitHub Pages :

```html
<form id="contact-form"
      action="https://formsubmit.co/incendie.biscarrosse@proton.me"
      method="POST" enctype="multipart/form-data">
```

### ⚠️ Étape unique d'activation (à faire une seule fois)

FormSubmit exige une confirmation avant de transmettre les emails à une
adresse pour la première fois :
1. Ouvrez le site en ligne et envoyez un premier signalement de test via le
   formulaire.
2. FormSubmit envoie un email de confirmation à
   **incendie.biscarrosse@proton.me** avec un lien à cliquer
   (« Activate form »).
3. Une fois ce lien cliqué, tous les signalements suivants sont transmis
   automatiquement à cette adresse, avec leur pièce jointe.

### Changer l'adresse de destination

Modifiez à la fois l'attribut `action` du formulaire (dans `index.html`) et
l'attribut `data-contact-email` (utilisé pour le message de secours en cas
d'échec réseau) :

```html
<form id="contact-form"
      action="https://formsubmit.co/nouvelle-adresse@exemple.com"
      data-contact-email="nouvelle-adresse@exemple.com" ...>
```

### Limites de FormSubmit (offre gratuite)
- 10 Mo maximum par envoi (toutes pièces jointes cumulées).
- Pas de tableau de bord : les emails partent directement en boîte de
  réception, rien n'est stocké côté FormSubmit.
- Anti-spam basique inclus (champ « honeypot » caché déjà présent dans le
  formulaire via `_honey`).

### Si l'envoi automatique échoue

Le formulaire retente une seule fois puis, en cas d'échec (connexion coupée,
service indisponible…), affiche un lien de secours qui ouvre la messagerie de
la personne avec un email pré-rempli à destination de la même adresse
(sans pièce jointe automatique dans ce cas de secours).

### Aller plus loin

D'autres services équivalents existent si vous préférez (Formspree, Web3Forms,
Google Forms en `<iframe>`…) — ils demandent en général un compte et ont des
limites différentes sur les pièces jointes. Pour les utiliser à la place,
remplacez l'attribut `action` du formulaire et adaptez le gestionnaire
`submit` dans `js/app.js` en conséquence.

Quelle que soit la solution retenue, pensez à relire les signalements avant de
les intégrer aux fichiers GeoJSON publiés.

## 4. Déployer le site

Le site est 100% statique : aucun serveur backend n'est requis pour
l'affichage de la carte (seul le formulaire a besoin d'un service externe,
cf. ci-dessus).

**Déploiement le plus simple : GitHub Pages**
1. Créez un dépôt GitHub et poussez-y ce dossier.
2. Dans les réglages du dépôt → *Pages*, activez la publication sur la branche
   principale.
3. Le site sera accessible via une URL du type
   `https://votre-compte.github.io/nom-du-depot/`.

**Alternatives** : Netlify ou Vercel (glisser-déposer le dossier sur leur
interface suffit), ou tout hébergement statique classique.

## 5. Mentions légales à conserver

Le site inclut déjà, en dur dans `index.html` :
- un bandeau rappelant qu'il s'agit d'une carte **collaborative et non
  officielle**, à but informatif ;
- un avertissement explicite sur la couche « bâtiments intacts » (évaluation
  fondée uniquement sur l'aspect extérieur visible en photo/vidéo, sans valeur
  d'expertise ni de confirmation officielle) ;
- une clause de non-responsabilité quant à l'exactitude des données ;
- les numéros d'urgence (18 / 112) et un renvoi vers les sources officielles
  (préfecture, mairie, SDIS).

Merci de conserver ces mentions telles quelles, et de les adapter si le cadre
juridique ou les informations officielles évoluent.

## 6. Aller plus loin (optionnel)

- Ajouter une couche « routes fermées » ou « points d'évacuation » en suivant
  le même modèle GeoJSON.
- Ajouter un horodatage automatique de mise à jour en committant les
  changements de GeoJSON avec un message de date.
- Mettre en place une modération (ex. un tableur partagé ou une base légère)
  entre la réception des signalements du formulaire et leur publication sur la
  carte, pour vérifier chaque source avant diffusion.
