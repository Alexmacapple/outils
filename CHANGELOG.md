# Changelog

## 1.0.0 — 2026-04-02

### Extracteur d'images vidéo

**Ajout initial** de l'outil d'extraction de diapositives uniques depuis une vidéo.

#### Fonctionnel
- Extraction par comparaison pixel avec seuil de similarité configurable
- Intervalle d'échantillonnage et qualité JPG ajustables
- Téléchargement individuel ou ZIP (JSZip)
- Pause/reprise du traitement
- Accordéon pédagogique DSFR sur les paramètres
- Alerte DSFR avec métadonnées vidéo (durée, poids, frames estimées)

#### Accessibilité
- DSFR 1.11.2 (header, footer, fieldset, accordion, alertes, grille responsive)
- RGAA 4.1.2 (hiérarchie titres, landmarks, labels, skip links, ARIA live regions)
- WCAG 2.2 AA (contrastes, clavier, lecteur d'écran, progressbar labellisé)
- Construction DOM programmatique (pas de innerHTML)

#### Sécurité
- Content Security Policy (meta)
- Subresource Integrity sur les 4 ressources CDN
- `'use strict'`, objet CONFIG, fonction escapeHTML
- ObjectURL révoquées, garde anti-race-condition, annulation extraction
