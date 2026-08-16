# Le Dimore del Salento — site vitrine hébergement

Landing page one-page pour un hébergement de **Nosy Be**, publiée en sous-domaine de `rentanoo.com`.
Sert aussi de **base réutilisable** pour créer un site par hébergement → voir `TEMPLATE-nouvel-hebergement.md`.

## En bref
- **Live** : https://ledimoredelsalento.rentanoo.com
- **Repo** : https://github.com/techerchristopher-dotcom/le-dimore-del-salento (public, branche `main`)
- **Hébergeur** : GitHub Pages (branche `main`, racine) + fichier `CNAME`
- **Stack** : **un seul `index.html`** (HTML/CSS/JS vanilla, zéro dépendance, zéro build) + dossier `images/`. Fonts Google (Fraunces + Manrope).

## ⚠️ Gotchas à connaître
- **Dépôt git isolé** : le HOME `/Users/christopher` est lui-même un dépôt git géant. Ce dossier a son **propre `.git`** (`git init` local). Ne JAMAIS `git add` depuis le home. `.gitignore` = `.DS_Store`.
- **Chemin avec espace** : `.../1-DEV CLAUDE /site web client/dimore del salento` (espace après `CLAUDE`) → toujours quoter.
- **Images = WebP uniquement**. Ne pas remettre de gros JPEG. Cf. section Images.
- **i18n** : toute string visible traduisible doit avoir sa clé FR dans l'objet `T` du script (sinon elle reste en français en IT/EN).

## Structure
```
index.html            ← tout le site (markup + <style> + <script>)
CNAME                 ← domaine perso (ledimoredelsalento.rentanoo.com)
robots.txt            ← SEO : Allow all + lien vers le sitemap
sitemap.xml           ← SEO : 1 URL (la home), lastmod
.gitignore            ← .DS_Store
.claude/launch.json   ← serveur de preview local (python http.server)
images/               ← photos .webp + logo.png + og.jpg (partage 1200×630) + favicon.svg
```

## Sections de la page (dans l'ordre)
Header (logo+wordmark cliquable→haut, sélecteur langue FR·IT·EN, CTA) · Hero (carrousel + titre + tagline + CTA) · Bandeau atouts (5 familles) · Galerie bento · « Ce qui fait la différence » (4) · Appartement type · Le lieu (carte + Google Maps) · Hôte (Ciao c'est Alex) · Tarif · CTA final (#contact) · Partenaire Rentanoo + Footer.

## Config par-propriété (OÙ changer quoi)
Tout ce qui change d'un hébergement à l'autre est regroupé/repérable :
- **Contacts (JS, dans le `<script>`)** :
  - `var WA_NUM = "261373437912";` → numéro WhatsApp (format international sans `+`).
  - `var waMsg = { fr, it, en }` → message WhatsApp pré-rempli.
  - E-mail : `mailto:chrisrentanoo@gmail.com` (section CTA final `#contact`).
- **Plateformes (JS)** : `var AIRBNB`, `var BOOKING` ; Rentanoo est en dur dans le HTML (`rentanoo.com/hebergement/XXXX`, 3 emplacements, chip doré « Meilleur tarif »).
- **Google Maps** : lien `maps.app.goo.gl/...` sur le bouton « Voir sur Google Maps » (section Le lieu).
- **Textes** : titres, tarif (`dès 45 €`), descriptions, nom de l'hôte, etc. dans le markup.
- **Photos** : `images/*.webp` (hero-1..5, res-1..5, chambre, cuisine, sdb).
- **Domaine** : fichier `CNAME`.

Valeurs actuelles : WhatsApp `+261373437912` · e-mail `chrisrentanoo@gmail.com` · Airbnb `airbnb.fr/rooms/1724758128139677758` · Booking annonce « appart-neuf-climatise-piscine-nosy-be » · Rentanoo `D3C8CDA7` · Maps `evCBE1PbkaQD9aGw9` · tarif « dès 45 € ».

## Fonctionnalités & où elles vivent (dans le `<script>` en bas de `index.html`)
- **i18n FR/IT/EN** : objet `T` (clés = texte FR exact) ; `apply(lang)` fait de l'**auto-match par 1er nœud de texte** (préserve icônes/spans). Cas spéciaux par id : `hero-title` et `host-h2` (via `titleHtml`/`hostHtml`, innerHTML). Sélecteur = `.lang-btn` ; choix mémorisé (`localStorage dds_lang`) + détection navigateur.
- **Carrousel hero** : fondu enchaîné + Ken Burns, auto ~5,5 s, pause au survol des puces uniquement.
- **Lightbox** : clic sur toute `img.slot` → agrandissement (fermer = clic/Échap).
- **Header au scroll** : classe `.scrolled` + styles inline (fond clair, couleur encre) ; la pilule du sélecteur de langue s'adapte.
- **CTA → WhatsApp** : `apply()` réécrit les `href` de `a.h-cta, #nav-cta` à chaque changement de langue.
- **Plateformes** : câblées par texte (`Airbnb`/`Booking`) après `apply(initial)`.

## Développement local
```bash
# preview (défini dans .claude/launch.json : python http.server)
# via l'outil preview, ou :
cd "…/dimore del salento" && python3 -m http.server 4610
```
Vérifs utiles : pas de scroll horizontal mobile (375/768), images non cassées, liens WhatsApp/plateformes câblés, bascule FR/IT/EN.

## Images (recette d'optimisation — IMPORTANT)
Sources HEIC → WebP redimensionné (dossier `images/` cible ~2 Mo total).
```bash
cd images
# hero (plein écran) : bord long ≤1600, q80
for f in hero-1 hero-2 hero-3 hero-4 hero-5; do cwebp -quiet -q 80 -m 6 -resize 0 1600 "$f.jpg" -o "$f.webp"; done
# sections : bord long ≤1200, q78
for f in res-1 res-2 chambre-1 cuisine-1 sdb-2; do cwebp -quiet -q 78 -m 6 -resize 0 1200 "$f.jpg" -o "$f.webp"; done
```
(HEIC → JPG intermédiaire via `sips -s format jpeg …` si besoin ; `-resize 0 H` = cap la hauteur pour photos portrait.)
Dans le HTML : `<img class="slot" loading="lazy" decoding="async" src="images/x.webp">`. Logo = `logo.png` détouré (fond retiré via PIL flood/keying, ~600px).

## SEO (présent sur tous les sites — OBLIGATOIRE, cf. TEMPLATE §3bis)
Chaque site embarque, en plus de la base déjà saine (`<html lang="fr">`, `<title>`/`<meta description>` uniques et riches en mots-clés, **un seul `<h1>`** = `#hero-title`, WebP + lazy) :
- **`robots.txt`** (Allow all + `Sitemap:`) et **`sitemap.xml`** (1 URL = la home, `lastmod`) à la racine.
- **Bloc `<head>` SEO** (juste après la `<meta name="description">`) : `canonical`, `robots` (max-image-preview:large), `theme-color #16211F`, `geo.region/placename`, **Open Graph** (og:type/site_name/locale + it/en/url/title/description/**image**/image:width=1200/height=630/alt), **Twitter Card** (`summary_large_image`), **favicon** (`images/favicon.svg` — soleil+vagues bleu/or ; dimore garde son `logo.png`), `preconnect` vers `i.ytimg.com`, et surtout **JSON-LD `LodgingBusiness`** (name, description, url, image[], telephone, email, priceRange « À partir de X € la nuit », address PostalAddress {locality, region:Nosy Be, country:MG}, amenityFeature[], `sameAs` → Rentanoo + Airbnb).
- **`images/og.jpg` 1200×630 « marketing »** : photo hero + scrim + nom (Fraunces) + accroche + chip prix doré + eyebrow « RENTANOO · NOSY BE ». Générée via le canvas `_ogmaker.html` (cf. TEMPLATE §3bis), à supprimer avant commit.
- **Non fait volontairement** : `hreflang` (inutile, 1 seule URL, langues en JS) ; lat/lng exact et `aggregateRating` (ne pas inventer — à ajouter quand vrais avis / emplacement précis).
- **Sitemaps INDÉPENDANTS** par sous-domaine (1 sous-domaine = 1 propriété Search Console). Pour tout regrouper → propriété **« Domaine » `rentanoo.com`** (TXT DNS Hostinger). Gros levier off-page restant = **liens internes depuis `rentanoo.com`** vers chaque site vitrine.

## Déploiement
```bash
# repo ISOLÉ (jamais git add -A depuis le home) → ajouter les fichiers voulus
git add index.html robots.txt sitemap.xml images/*.webp images/og.jpg images/favicon.svg CNAME
git commit -m "…" && git push origin main   # Pages rebuild ~1-2 min
```
Vérifier live : `curl -s https://ledimoredelsalento.rentanoo.com/`.
Après mise en ligne : soumettre `…/sitemap.xml` dans **Search Console** + « Scrape Again » dans le **Facebook Sharing Debugger** (rafraîchit l'aperçu WhatsApp/FB).

### Domaine perso (sous-domaine rentanoo.com) — déjà configuré ici
1. DNS **Hostinger** (rentanoo.com y est géré) : enregistrement **CNAME** `ledimoredelsalento` → `techerchristopher-dotcom.github.io`.
2. Fichier `CNAME` (contenu = `ledimoredelsalento.rentanoo.com`) à la racine du repo.
3. `gh api -X PUT repos/OWNER/REPO/pages -f cname='…'` puis `-F https_enforced=true` (certificat auto).

## En attente / à finaliser
- Portrait d'**Alex** (placeholder — bloc commenté section « HÔTE », prévoir `images/alex.*`).
- ~~Open Graph + favicon~~ ✅ faits (cf. section SEO).
- Côté client : soumettre les sitemaps dans Search Console ; envisager la propriété « Domaine » `rentanoo.com` ; ajouter les liens internes rentanoo.com → sites vitrines.
