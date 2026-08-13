# 🏗️ Template — créer un site pour un nouvel hébergement Rentanoo

Objectif : reproduire ce site (Le Dimore del Salento) pour **n'importe quel hébergement** de rentanoo.com,
publié sur son propre **sous-domaine** `NOM.rentanoo.com`.

Ce site (`index.html`) EST le template de base : on **duplique**, on remplace le contenu, on optimise les photos, on déploie.
Toute la config par-propriété est repérée dans `CLAUDE.md` (section « Config par-propriété »).

---

## 0. Infos à récupérer du client (checklist)
- [ ] **Nom** de l'hébergement + **slug** (ex. `le-dimore-del-salento`) → sert au repo ET au sous-domaine
- [ ] **Localisation** (quartier, île) + lien **Google Maps** (`maps.app.goo.gl/…`)
- [ ] **Photos** (HEIC/JPG) : extérieurs, piscine, chambres, cuisine, salle de bain… + éventuel **logo** + **portrait de l'hôte**
- [ ] **Prénom de l'hôte** + histoire courte (2-3 phrases)
- [ ] **WhatsApp** (format international, ex. `+261…`) + **e-mail** de contact
- [ ] **Tarif** (« dès X € la nuit »)
- [ ] **Caractéristiques** : nb d'appartements, surface, équipements (clim, wifi, parking…), distance plage/aéroport
- [ ] Liens plateformes : **Airbnb** (`airbnb.fr/rooms/ID`), **Booking** (URL annonce), **Rentanoo** (`rentanoo.com/hebergement/CODE`)

## 1. Créer le nouveau projet (dupliquer la base)
```bash
BASE="/Users/christopher/Desktop/1-DEV CLAUDE /site web client/dimore del salento"
DEST="/Users/christopher/Desktop/1-DEV CLAUDE /site web client/NOUVEAU-SLUG"
mkdir -p "$DEST" && cp "$BASE/index.html" "$DEST/" && cp "$BASE/.gitignore" "$DEST/"
mkdir -p "$DEST/.claude" "$DEST/images"
# repo git ISOLÉ (piège home-is-git) :
cd "$DEST" && git init -q && git checkout -q -b main
```
Copier aussi `CLAUDE.md`/`TEMPLATE-*.md` comme référence si utile.

## 2. Photos → WebP (le point « vitesse »)
Mettre les sources dans `images/` (convertir HEIC→JPG d'abord si besoin : `sips -s format jpeg src.heic --out x.jpg`).
```bash
cd images
# hero (fond plein écran) : 5 photos, bord long ≤1600, q80
for f in hero-1 hero-2 hero-3 hero-4 hero-5; do cwebp -quiet -q 80 -m 6 -resize 0 1600 "$f.jpg" -o "$f.webp"; done
# sections : bord long ≤1200, q78
for f in res-1 res-2 res-3 res-4 res-5 chambre-1 chambre-2 cuisine-1 sdb-2; do cwebp -quiet -q 78 -m 6 -resize 0 1200 "$f.jpg" -o "$f.webp"; done
rm -f *.jpg   # ne garder que les .webp (+ logo.png)
```
Logo : détourer le fond (PIL) + resize ~600px, garder PNG (transparence). Cible dossier `images/` : **~2 Mo**.

## 3. Remplacer le contenu dans `index.html`
Toutes les images sont déjà en `.webp` avec `loading="lazy"`. À adapter :
- **Textes** : titre hero, tagline, tarif (`dès X €`), descriptions, nom de l'hôte (`Ciao, c'est …` → id `host-h2` + `hostHtml`), bandeau 5 familles, « Le lieu », etc.
- **i18n** : pour CHAQUE texte FR modifié, mettre à jour la **clé + traductions** dans l'objet `T` (et `titleHtml`/`hostHtml` pour le titre/hôte). Sinon l'IT/EN reste en français.
- **Photos** : renommer/mapper les `.webp` aux bons emplacements (hero carrousel, galerie, différence, appartement, CTA bg).
- **Portrait hôte** : remplacer le bloc placeholder `.portrait-ph` (section HÔTE) par `<img class="slot" src="images/alex.webp" …>`.

## 4. Contacts & plateformes (dans le `<script>`)
```js
var WA_NUM = "261XXXXXXXXX";           // WhatsApp sans +
var waMsg  = { fr:"…", it:"…", en:"…" };// message pré-rempli
var AIRBNB = "https://www.airbnb.fr/rooms/ID";
var BOOKING= "https://www.booking.com/hotel/…";
```
- E-mail : remplacer `mailto:chrisrentanoo@gmail.com` (section CTA final).
- Rentanoo : remplacer `rentanoo.com/hebergement/CODE` (3 occurrences, chips dorés « Meilleur tarif »).
- Google Maps : remplacer le lien du bouton « Voir sur Google Maps » (section Le lieu).

## 5. Vérifs (preview local)
`.claude/launch.json` (python http.server sur un port libre), puis contrôler :
- Pas de scroll horizontal à **375** et **768** px (mobile-first).
- Images non cassées, `<title>`/meta OK.
- Liens WhatsApp / Airbnb / Booking / Rentanoo / Maps câblés.
- Bascule **FR / IT / EN** (aucune string oubliée).

## 6. Déployer + sous-domaine
```bash
git add -A && git commit -m "Site NOUVEAU-SLUG" 
gh repo create NOUVEAU-SLUG --public --source=. --remote=origin --push
```
Puis le sous-domaine `SLUG.rentanoo.com` :
1. **Hostinger** (DNS de rentanoo.com) → Edit DNS zone → **CNAME** : Name `SLUG`, Points to `techerchristopher-dotcom.github.io`.
2. Fichier `CNAME` à la racine = `SLUG.rentanoo.com` → commit+push.
3. Config Pages + HTTPS :
```bash
gh api -X PUT repos/techerchristopher-dotcom/NOUVEAU-SLUG/pages -f cname='SLUG.rentanoo.com'
# attendre le certificat (state "approved") puis :
gh api -X PUT repos/techerchristopher-dotcom/NOUVEAU-SLUG/pages -F https_enforced=true
```
4. Vérifier : `curl -s -o /dev/null -w "%{http_code}" https://SLUG.rentanoo.com/` → `200`.

> ⚠️ Poser le **DNS d'abord**, le fichier `CNAME` **ensuite** — sinon le site est injoignable le temps de la propagation.

## 7. Checklist « opérationnel »
- [ ] WhatsApp + e-mail cliquables · [ ] Airbnb/Booking/Rentanoo réels · [ ] Google Maps
- [ ] Photos WebP + lazy (dossier ≤ ~2,5 Mo) · [ ] FR/IT/EN complet
- [ ] Responsive OK · [ ] Sous-domaine HTTPS en ligne · [ ] Portrait hôte (si dispo)

---

### Idée d'évolution (si beaucoup d'hébergements)
Externaliser la config par-propriété dans un objet JS `CONFIG = {…}` en haut du `<script>` (nom, wa, mail, plateformes, maps, tarif, textes) pour ne modifier qu'**un seul bloc** par nouveau site. Refactor à faire une fois, gain de temps ensuite.
