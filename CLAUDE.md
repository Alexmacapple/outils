# Outils Miweb

**Depot** : git@github.com:Alexmacapple/outils.git (public, SSH)

---

## Structure

```
outils/
├── extracteur-slides-video.html   # Outil standalone extraction slides
├── CHANGELOG.md                    # Historique des versions
├── README.md                       # Documentation utilisateur
├── LICENSE                         # GPL v3
└── CLAUDE.md                       # Ce fichier
```

---

## Conventions

- **Langue** : francais (code source et variables en anglais)
- **Nommage** : kebab-case, pas d'accents dans les noms de fichiers
- **Commits** : francais, forme nominale, style conventionnel
- **Licence** : GPL v3 sur tous les fichiers

---

## Standards techniques

Chaque outil de ce depot doit respecter :

- **DSFR** : Design System de l'Etat (version 1.11 minimum)
- **RGAA** 4.1.2 : conformite accessibilite secteur public
- **WCAG** 2.2 AA : contrastes, clavier, lecteur d'ecran
- **Securite** : CSP, SRI sur les CDN, pas de innerHTML avec donnees non fiables, `'use strict'`

---

## Workflow

- Un outil = un fichier HTML standalone (pas de build, pas de dependances locales)
- Les CDN externes doivent avoir un attribut `integrity` (SRI)
- Tester en `file://` ET en HTTP avant de pousser
- Valider avec axe-core (0 violation) avant chaque release

---

## Outils existants

| Fichier | Description | Version |
|---------|-------------|---------|
| `extracteur-slides-video.html` | Extraction de diapositives uniques depuis une video | 1.0.0 |
