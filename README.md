# Outils SNUM Miweb

Outils autonomes pour le traitement de contenus numériques accessibles.

---

## Extracteur d'images video

**Fichier** : `video-extractor-local.html`

Extrait automatiquement les diapositives uniques d'une video au format JPG. Traitement 100 % local (Canvas API), aucune donnee envoyee.

### Fonctionnalites

- Detection automatique des changements de slides par comparaison pixel
- Parametres ajustables (intervalle, seuil de similarite, qualite JPG)
- Telechargement individuel ou ZIP
- Accordeon pedagogique expliquant les parametres
- Pause/reprise du traitement

### Conformite

- **DSFR** 1.11.2 (Design System de l'Etat)
- **RGAA** 4.1.2 (hierarchie titres, landmarks, labels, fieldset, skip links, ARIA)
- **WCAG** 2.2 AA (contrastes, clavier, lecteur d'ecran, live regions)
- **Securite** : CSP meta, SRI sur les 4 CDN, construction DOM programmatique (pas de innerHTML), `'use strict'`

### Utilisation

Ouvrir `video-extractor-local.html` dans un navigateur (Chrome recommande). Fonctionne en `file://` ou servi en HTTP.

---

## Licence

Ce projet est distribue sous licence **GNU General Public License v3.0**.
Voir le fichier [LICENSE](LICENSE) pour le texte complet.

---

SNUM Miweb
