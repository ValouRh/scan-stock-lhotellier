# Scan Stock Lhotellier — version GitHub Pages + Google Sheets

Application en 3 pages qui partagent la même feuille Google Sheets comme base de données, via un petit script Google gratuit (Apps Script) — pas de nouveau service tiers à créer :

- **`index.html`** — l'appli terrain : scan caméra en direct, panier multi-articles, validation de sortie. C'est le lien à partager avec les techniciens.
- **`historique.html`** — historique des sorties + statistiques (top articles, par magasin, par personne, évolution par mois), avec un statut « traité / non traité » à cocher au fur et à mesure de la saisie dans la GMAO. Accessible depuis l'icône 📦 en haut de `index.html`, protégée par un code (magasinier ou admin).
- **`admin.html`** — page d'administration : génération de QR codes pour de nouveaux articles (avec étiquette à télécharger et ajout automatique au référentiel), gestion du référentiel existant, gestion de la liste des magasins. Accessible depuis l'icône ⚙️ en haut de `index.html`, protégée par le code admin.

Contrepartie par rapport à une base "temps réel" façon Firebase : les mises à jour ne sont pas instantanées à la milliseconde près (l'appli interroge la feuille toutes les quelques secondes), ce qui reste largement suffisant pour un usage terrain.

---

## Étape 1 — Créer la feuille Google Sheets et le script (5 min)

1. Va sur https://sheets.google.com et crée une feuille vide. Nomme-la par exemple **Sorties de stock**.
2. Dans le menu, clique **Extensions** → **Apps Script**. Un éditeur de code s'ouvre dans un nouvel onglet.
3. Supprime tout le code par défaut (`function myFunction() {...}`) et colle à la place le contenu du fichier **`Code.gs`** fourni avec ces instructions.
4. Clique l'icône de sauvegarde (disquette) ou `Ctrl+S` / `Cmd+S`.
5. En haut à droite, clique **Déployer** → **Nouveau déploiement**.
6. Clique l'icône ⚙️ à côté de "Sélectionner le type" → choisis **Application Web**.
7. Renseigne :
   - **Exécuter en tant que** : Moi (ton compte Google)
   - **Qui a accès** : **Tout le monde**
8. Clique **Déployer**. Google demande d'autoriser le script à accéder à la feuille — accepte (c'est ton propre script, sur ta propre feuille).
9. Une URL apparaît, du type `https://script.google.com/macros/s/AKfycb.../exec`. **Copie cette URL**, tu en auras besoin à l'étape 3.

Les onglets "Scans" et "Referentiel" se créent automatiquement dans ton classeur au premier appel de l'application — pas besoin de préparer les colonnes toi-même. Il ne reste que le **référentiel initial** à importer (étape 1 bis ci-dessous), pour ne pas avoir à ressaisir tes 2600 articles à la main un par un depuis la page admin.

## Étape 1 bis — Importer ton référentiel d'articles existant (une seule fois)

1. Dans la même feuille Google Sheets, crée un nouvel onglet nommé exactement **`Referentiel`** (clic droit sur un onglet en bas → Insérer une feuille, puis renomme-la).
2. Dans cet onglet, sélectionne la cellule **A1**, puis **Fichier** → **Importer** → onglet **Importer** → dépose le fichier `referentiel_import.csv` fourni avec ces instructions.
3. Choisis **Insérer une nouvelle feuille** ? Non — choisis plutôt **"Remplacer la feuille actuelle"** (ou "Ajouter à la feuille actuelle" si l'onglet est vide), séparateur **virgule**, puis **Importer les données**.
4. Vérifie que la colonne A contient bien les références (en-tête `Reference` en A1) et la colonne B les désignations (en-tête `Designation` en B1), à partir de la ligne 2.

Une fois cet import fait, chaque nouvel article créé depuis la page `admin.html` s'ajoutera automatiquement à la suite de cet onglet — plus besoin d'y retoucher à la main.

## Étape 2 — Créer le dépôt GitHub

1. Va sur https://github.com et connecte-toi (ou crée un compte gratuit).
2. Clique **New repository**, nomme-le par exemple `scan-stock-lhotellier`, mets-le en **Public** (nécessaire pour GitHub Pages gratuit), puis **Create repository**.
3. Dans le dépôt vide, clique **uploading an existing file** (ou "Add file" → "Upload files").
4. Dépose les **trois fichiers** `index.html`, `historique.html` et `admin.html` (après avoir renseigné l'URL Apps Script dans chacun, étape 3 ci-dessous).
5. Valide (**Commit changes**).

## Étape 3 — Renseigner l'URL Apps Script dans les fichiers

1. Ouvre `index.html` avec un éditeur de texte.
2. Cherche, en haut du `<script>` :

   ```js
   var APPS_SCRIPT_URL = "REPLACE_ME";
   ```

3. Remplace `"REPLACE_ME"` par l'URL copiée à l'étape 1.9, par exemple :

   ```js
   var APPS_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbXXXXXXXXXXXX/exec";
   ```

4. Fais exactement la même chose dans `historique.html` et dans `admin.html` (même variable, tout en haut de leur `<script>`) — **les trois fichiers doivent pointer vers la même URL**.
5. Enregistre les trois fichiers, puis envoie-les sur GitHub (upload, ou modification directe dans l'éditeur GitHub en ligne).

## Étape 4 — Activer GitHub Pages

1. Dans le dépôt GitHub, va dans **Settings** → **Pages**.
2. Sous "Build and deployment", choisis **Source: Deploy from a branch**.
3. Branche : `main`, dossier : `/ (root)`. Clique **Save**.
4. Après 1–2 minutes, l'URL publique apparaît, du type :
   `https://<ton-nom-utilisateur>.github.io/scan-stock-lhotellier/`

C'est ce lien (vers `index.html`) que tu partages avec les techniciens. Les pages `historique.html` et `admin.html` sont accessibles depuis les deux icônes en haut de cette page (📦 magasinier, ⚙️ administration), chacune protégée par son propre code.

---

## Avantage : tu retrouves directement tes données dans Sheets

Chaque sortie de stock est une ligne dans l'onglet **Scans** de ta feuille, et chaque article du référentiel une ligne dans l'onglet **Referentiel** — tu peux les trier, filtrer, faire un TCD, les exporter, les partager avec l'équipe magasin, exactement comme n'importe quelle feuille Google Sheets. L'export CSV depuis `historique.html` reste disponible en plus, si besoin.

La liste des magasins n'est pas stockée dans une feuille visible (elle vit dans les "propriétés" du script), mais reste modifiable depuis `admin.html`.

## Mot de passe d'accès

Un seul mot de passe protège à la fois `historique.html` (icône 📦) et `admin.html` (icône ⚙️) : **`N1colas1809@`**. Il est redemandé à chaque ouverture de l'une de ces deux pages — rien n'est mémorisé dans le navigateur, donc il faut le ressaisir à chaque fois, y compris en revenant sur une page déjà ouverte plus tôt.

Pour le changer, cherche `APP_PASSWORD` en haut du `<script>` de `historique.html` **et** de `admin.html` — remplace la valeur dans les deux fichiers (la même dans les deux) puis republie sur GitHub.

## Générer un QR code pour un nouvel article

Depuis `admin.html`, onglet **Générer un QR code** :
1. Saisis la référence de l'article (c'est elle qui est encodée dans le QR code et lue au scan) et sa désignation.
2. Clique **Générer le QR code** — une étiquette apparaît avec le QR code et le texte "référence — désignation" en dessous.
3. Clique **Télécharger l'étiquette (PNG)** pour l'imprimer et la coller sur l'article.
4. Clique **Ajouter au référentiel** pour que la désignation apparaisse automatiquement dans le panier de tous les techniciens dès qu'ils scanneront cette référence — inutile de la ressaisir où que ce soit d'autre.

L'onglet **Référentiel** de la page admin permet de rechercher un article existant et de corriger sa désignation si besoin (bouton "Éditer", qui préremplit le générateur).

## Mettre à jour l'appli plus tard

Pour changer quoi que ce soit dans l'appli (texte, couleurs, magasins par défaut...), modifie le fichier concerné directement dans GitHub (bouton crayon ✏️) et valide — la page se met à jour en 1–2 minutes.

Pour changer la logique côté serveur (colonnes, règles de suppression...), modifie `Code.gs` dans l'éditeur Apps Script, sauvegarde, puis refais **Déployer → Gérer les déploiements → ✏️ (modifier) → Nouvelle version → Déployer** (l'URL reste la même, pas besoin de la remettre à jour dans les fichiers HTML).
