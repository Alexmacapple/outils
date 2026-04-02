# Outils SNUM Miweb

Outils autonomes pour le traitement de contenus numériques accessibles.

---

## Extracteur d'images vidéo

**Fichier** : `video-extractor-local.html`

Extrait automatiquement les diapositives uniques d'une vidéo au format JPG. Traitement 100 % local (Canvas API), aucune donnée envoyée.

### Fonctionnalités

- Détection automatique des changements de slides par comparaison pixel
- Paramètres ajustables (intervalle, seuil de similarité, qualité JPG)
- Téléchargement individuel ou ZIP
- Accordéon pédagogique expliquant les paramètres
- Pause/reprise du traitement

### Conformité

- **DSFR** 1.11.2 (Design System de l'État)
- **RGAA** 4.1.2 (hiérarchie titres, landmarks, labels, fieldset, skip links, ARIA)
- **WCAG** 2.2 AA (contrastes, clavier, lecteur d'écran, live regions)
- **Sécurité** : CSP meta, SRI sur les 4 CDN, construction DOM programmatique (pas de innerHTML), `'use strict'`

### Utilisation

Ouvrir `video-extractor-local.html` dans un navigateur (Chrome recommandé). Fonctionne en `file://` ou servi en HTTP.

---

## Licence

Ce projet est distribué sous licence **GNU General Public License v3.0**.
Voir le fichier [LICENSE](LICENSE) pour le texte complet.

---

## Auteurs

- SNUM Miweb
- Assisté par Claude Code (Anthropic)
