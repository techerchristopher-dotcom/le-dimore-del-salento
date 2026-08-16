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
- **Portrait hôte** : remplacer le bloc placeholder `.portrait-ph` (section HÔTE) par `<img class="slot" src="images/chris.webp" …>`. Photo de Chris = bucket Supabase **`photo fondateur/photo techer christopher .png`**. ⚠️ **Nettoyage contour** : la photo fournie a un **anneau blanc + coins sombres** incrustés → les supprimer (PIL : masque cercle `R≈558` centré, flou gaussien ~10, composite sur un fond = couleur moyenne d'un patch intérieur gris ; cf. `chris-v2.webp`). Afficher en `max-width:410px` (sinon trop grosse). **Anti-cache** : si on remplace une image existante, changer le **nom de fichier** (ex. `-v2`) sinon le navigateur garde l'ancienne.
- **Logo Rentanoo (partenaire)** : source = bucket **`email-asset/logo rentanoo 2.png`** (recadrer les marges blanches → `images/rentanoo-logo.webp`, ~520px). Placer dans la **section Partenaire** (en tête, `height:34px`) et dans le **footer** (« Site propulsé par [logo] », `height:17px`). Fond blanc du logo → se fond dans les sections claires.
- **Logo du site (si fourni)** : emblème rond → header (`.hdr-logo`, **version inversée sur le hero ↔ couleur au scroll** via `#site-header.scrolled .logo-color`) + footer ; favicon assorti.

## 3bis. SEO (OBLIGATOIRE — même recette pour tous les sites)
> Remplacer partout `SLUG` (ex. `appart-ambatoloaka`), `NOM`, `LOCALITE` (Madirokely/Andilana/Ambatoloaka), `PRIX` (25/45/85), `CODE` (Rentanoo), `IDAIRBNB`, `AAAA-MM-JJ` (date du jour).

**a) `robots.txt`** (racine) :
```
User-agent: *
Allow: /

Sitemap: https://SLUG.rentanoo.com/sitemap.xml
```

**b) `sitemap.xml`** (racine) :
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://SLUG.rentanoo.com/</loc>
    <lastmod>AAAA-MM-JJ</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

**c) `images/favicon.svg`** (soleil + vagues, charte marque — sauf si le site a un vrai logo, garder `logo.png`) :
```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="14" fill="#1785C0"/>
  <circle cx="32" cy="27" r="11" fill="#F2C230"/>
  <path d="M6 44c6 0 6-4 12-4s6 4 12 4 6-4 12-4 6 4 12 4" fill="none" stroke="#fff" stroke-width="4" stroke-linecap="round"/>
  <path d="M6 53c6 0 6-4 12-4s6 4 12 4 6-4 12-4 6 4 12 4" fill="none" stroke="#fff" stroke-width="4" stroke-linecap="round" opacity=".55"/>
</svg>
```

**d) Bloc `<head>` SEO** — coller **juste après la `<meta name="description">`** (adapter les valeurs) :
```html
<link rel="canonical" href="https://SLUG.rentanoo.com/">
<meta name="robots" content="index, follow, max-image-preview:large, max-snippet:-1">
<meta name="theme-color" content="#16211F">
<meta name="author" content="Rentanoo">
<meta name="geo.region" content="MG">
<meta name="geo.placename" content="LOCALITE, Nosy Be">
<link rel="icon" type="image/svg+xml" href="images/favicon.svg">
<link rel="preconnect" href="https://i.ytimg.com">

<meta property="og:type" content="website">
<meta property="og:site_name" content="Rentanoo · Nosy Be">
<meta property="og:locale" content="fr_FR">
<meta property="og:locale:alternate" content="it_IT">
<meta property="og:locale:alternate" content="en_US">
<meta property="og:url" content="https://SLUG.rentanoo.com/">
<meta property="og:title" content="NOM — ACCROCHE · Nosy Be">
<meta property="og:description" content="… (reprendre/raccourcir la meta description) …">
<meta property="og:image" content="https://SLUG.rentanoo.com/images/og.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:image:alt" content="NOM — LOCALITE, Nosy Be">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="NOM — ACCROCHE · Nosy Be">
<meta name="twitter:description" content="… (idem) …">
<meta name="twitter:image" content="https://SLUG.rentanoo.com/images/og.jpg">

<script type="application/ld+json">
{"@context":"https://schema.org","@type":"LodgingBusiness","name":"NOM","description":"…","url":"https://SLUG.rentanoo.com/","image":["https://SLUG.rentanoo.com/images/og.jpg","https://SLUG.rentanoo.com/images/hero-1.webp"],"telephone":"+261373437912","email":"chrisrentanoo@gmail.com","priceRange":"À partir de PRIX € la nuit","currenciesAccepted":"EUR","address":{"@type":"PostalAddress","addressLocality":"LOCALITE","addressRegion":"Nosy Be","addressCountry":"MG"},"areaServed":"Nosy Be","amenityFeature":[{"@type":"LocationFeatureSpecification","name":"Wifi","value":true},{"@type":"LocationFeatureSpecification","name":"Cuisine équipée","value":true}],"sameAs":["https://rentanoo.com/hebergement/CODE","https://www.airbnb.fr/rooms/IDAIRBNB"]}
</script>
```
> Valider le JSON-LD (parse strict) avant deploy : `python3 -c "import re,json;[json.loads(b) for b in re.findall(r'ld\+json\">(.*?)</script>',open('index.html').read(),16)]"`.

**e) Image de partage `images/og.jpg` 1200×630 « marketing »** (nom + accroche + prix incrustés sur la photo hero) :
Créer un fichier temporaire `_ogmaker.html` à la racine, avec un `<canvas width=1200 height=630>` et un `window.__drawOG(site)` qui : charge `images/hero-1.webp` (`await img.decode()`), attend les polices (`document.fonts.load(...)` avec race timeout 4 s), dessine la photo en **cover**, un **scrim** gauche+bas `rgba(9,26,32,…)`, l'eyebrow doré `RENTANOO · NOSY BE` (Manrope 600 20px, letterSpacing 4px), le **nom** (Fraunces 300 66px blanc + liseré doré), l'**accroche** (Manrope 500 30px), un **chip prix doré** `dès PRIX € / nuit` (Manrope 700 30px, fond `#F2C230`, texte `#16211F`), puis `return canvas.toDataURL('image/jpeg',0.85)`.
Procédure : `python3 -m http.server` → ouvrir `_ogmaker.html` dans le navigateur → **réveiller l'onglet par un screenshot** (sinon rAF/polices throttlés en arrière-plan) → appeler `window.__drawOG(...)` → récupérer le dataURL → décoder en `images/og.jpg` via python (`base64.b64decode`) → **supprimer `_ogmaker.html`** avant commit.
> Version rapide (photo brute, sans texte) si pressé : `dwebp hero-1.webp -o /tmp/x.png && sips --resampleWidth 1200 /tmp/x.png --out /tmp/y.png && sips -c 630 1200 /tmp/y.png --out /tmp/z.png && sips -s format jpeg -s formatOptions 82 /tmp/z.png --out images/og.jpg`.

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
# repo ISOLÉ d'abord (piège home-is-git) : git init puis add SPÉCIFIQUE (jamais -A depuis le home)
git init -q && git symbolic-ref HEAD refs/heads/main
git add index.html robots.txt sitemap.xml .gitignore .claude/launch.json images/*.webp images/og.jpg images/favicon.svg
git commit -m "Site NOUVEAU-SLUG"
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
- [ ] **SEO (§3bis)** : robots.txt + sitemap.xml · bloc `<head>` OG/Twitter/JSON-LD/canonical/favicon · **og.jpg 1200×630** · JSON-LD validé
- [ ] Après deploy : sitemap soumis dans Search Console · « Scrape Again » Facebook Debugger

---

### Idée d'évolution (si beaucoup d'hébergements)
Externaliser la config par-propriété dans un objet JS `CONFIG = {…}` en haut du `<script>` (nom, wa, mail, plateformes, maps, tarif, textes) pour ne modifier qu'**un seul bloc** par nouveau site. Refactor à faire une fois, gain de temps ensuite.
