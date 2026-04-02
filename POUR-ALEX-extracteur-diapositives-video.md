# Ce qu'on a appris en construisant l'extracteur de diapositives video

> Retour pedagogique sur la session du 2 avril 2026
> De "ca timeout apres 339 secondes" a un outil de production conforme DSFR/RGAA/WCAG/CSP, publie sur GitHub.

---

## 1. Approche choisie

On est partis d'un fichier HTML existant qui fonctionnait... sauf sur les longues videos. Le reflexe aurait ete de simplement augmenter le timeout. On a plutot choisi de comprendre pourquoi c'etait lent, et de corriger la cause racine au lieu du symptome.

Le raisonnement initial etait simple : si une video de 19 minutes genere un timeout a 339 secondes, ce n'est pas le timeout qui est trop court — c'est le traitement qui est trop lent. En remontant le fil, on a trouve trois coupables empiles : `requestIdleCallback` qui attendait que le navigateur soit libre (il ne l'est jamais pendant un traitement intensif), un echantillonnage de pixels trop dense, et un timeout global fixe au lieu d'un timeout d'inactivite.

C'est comme reparer une fuite d'eau : on ne met pas un seau plus grand, on trouve le tuyau perce.

---

## 2. Alternatives ecartees

**Augmenter le timeout** : la solution de facilite. On aurait pu mettre 30 minutes et prier. Rejete parce que ca masque le vrai probleme — le traitement de chaque frame prend trop longtemps.

**Web Workers pour la comparaison de pixels** : techniquement elegant, ca aurait deplace la comparaison hors du thread principal. Rejete parce que la comparaison synchrone avec 1 500 pixels prend moins d'une milliseconde — le gain ne justifie pas la complexite.

**Convertir les dataURL en Blob pour la memoire** : la revue de code l'a identifie comme limitation. On a choisi de ne pas le faire parce que ca aurait necessite une refonte du systeme ZIP et du telechargement, pour un gain pertinent seulement sur des videos de plus d'une heure avec des centaines de slides uniques. Le rapport effort/impact n'etait pas la.

**Serveur HTTP local** au lieu de `file://` : quand les `blob:null` posaient probleme, on a envisage de demander a l'utilisateur de lancer `python3 -m http.server`. Rejete parce que l'outil doit rester un fichier qu'on ouvre en double-cliquant, sans prerequis.

---

## 3. Architecture et articulation

Le fichier est un monolithe HTML de 1 469 lignes. C'est voulu — un outil standalone n'a pas besoin d'architecture modulaire.

L'ordre des corrections a suivi une logique de couches :

1. **Fonctionnel** d'abord : le timeout, le `requestIdleCallback`, la comparaison synchrone. Si l'outil ne marche pas, le reste est inutile.
2. **Typographie et orthographe** : les espaces insecables, les unites francaises. Ca parait cosmetique mais ca conditionne la credibilite de l'outil aupres des utilisateurs internes.
3. **Accessibilite** : titres, landmarks, ARIA, labels, fieldset. C'est structurel — ca se corrige mieux avant d'ajouter des fonctionnalites.
4. **DSFR** : header, footer, grille, accordeon, tokens CSS. Le habillage institutionnel.
5. **Securite** : innerHTML, CSP, SRI. La couche de durcissement.
6. **Bug file://** : le debug du `blob:null` est venu en dernier parce qu'il est apparu seulement apres les corrections de securite (la revocation d'ObjectURL a revele un probleme latent).

Chaque couche dependait de la precedente. On ne peut pas auditer les contrastes DSFR si le DSFR JS ne charge pas. On ne peut pas tester l'extraction si le timeout l'empeche de finir.

---

## 4. Outils et methodes

**axe-core** (via accesslint MCP) : audit automatique a chaque modification. Zero violation du debut a la fin — mais axe-core ne detecte que 30-40 % des problemes d'accessibilite. Les vrais problemes (hierarchie de titres, `aria-labelledby` coherent, contrastes dark mode) ont ete trouves par inspection manuelle et par les outils de l'utilisateur.

**Chrome DevTools** (via MCP) : indispensable pour le debug du `blob:null`. Les tests de seek programmatique depuis la console ont prouve que le probleme n'etait pas le blob URL mais la video en lecture pendant les seeks.

**Tanaguru / validateur W3C** (cote utilisateur) : c'est la que les vrais problemes sont apparus — le `role="banner"` redondant, le `src=""` invalide, le `label` sur un input `display:none`, le contraste des boutons disabled. axe-core n'avait rien vu.

**Revue de code par sous-agent** : deux passes (qualite + securite Codex). La premiere a identifie les innerHTML, les ObjectURL non revoquees, la race condition. La seconde a verifie que tout etait corrige et a ajoute CSP/SRI.

La lecon : un seul outil d'audit ne suffit jamais. C'est la combinaison axe-core + validateur W3C + Tanaguru + inspection manuelle qui donne une couverture correcte.

---

## 5. Compromis

**`'unsafe-inline'` dans la CSP** : on a du l'accepter pour les styles (le DSFR les requiert) et les scripts (fichier standalone, pas de build pour generer des hashes). C'est un compromis de securite assume — la CSP verrouille les sources mais n'elimine pas completement le risque XSS inline. En production sur un serveur, on externaliserait les scripts.

**Dark mode supprime** : le DSFR 1.11.2 en dark mode produit des contrastes de 1.5-2:1 sur tous les textes. Plutot que de surcharger chaque token CSS du DSFR (fragile et non maintenable), on a force le mode clair. L'utilisateur perd une fonctionnalite, mais gagne la conformite WCAG.

**dataURL au lieu de Blob** : chaque image extraite est une chaine base64 en memoire. Pour 100 images, c'est ~50 Mo. On a accepte cette limitation parce que la conversion en Blob aurait casse le telechargement ZIP et l'affichage dans la galerie. Le commentaire dans le code documente la limitation.

**`!important` sur les boutons disabled** : le DSFR definit les couleurs disabled avec une specificite elevee. Pour atteindre 4.5:1, on a du utiliser `!important`. C'est sale, mais c'est le seul moyen de surpasser un design system sans le forker.

---

## 6. Erreurs et impasses

**La revocation ObjectURL qui casse tout** : la revue de code a identifie les ObjectURL non revoquees comme une fuite memoire. Correction appliquee — et l'extraction a cesse de fonctionner. On revoquait l'URL de la video pendant qu'elle etait encore en cours d'utilisation pour les seeks. C'est l'equivalent de ranger les ingredients pendant que le gateau est encore au four.

**Le blob:null, trois heures de debug** : les erreurs `Not allowed to load local resource: blob:null/...` apparaissaient dans la console. On a d'abord cru que c'etait la revocation (corrige). Puis que c'etait `crossOrigin = 'anonymous'` (supprime). Puis que c'etait `createElement('video')` (remplace par previewVideo). Les erreurs persistaient. Finalement, un test DevTools a montre que les seeks fonctionnaient parfaitement — les erreurs etaient du bruit console sans impact fonctionnel. Le vrai probleme etait que la video etait en lecture pendant les seeks programmatiques. Un simple `video.pause()` a tout resolu.

La lecon : les erreurs console ne sont pas toujours la cause du bug. Il faut tester le comportement reel, pas juste lire les logs.

**Le `about:blank` bloque par la CSP** : pour eviter le `src=""` invalide W3C, on a mis `src="about:blank"`. La CSP l'a bloque. Retour au `data:image/gif;base64,...` qui etait en fait autorise par la CSP depuis le debut — le warning du validateur W3C etait un faux positif.

**25 violations de contraste d'un coup** : l'ajout du toggle theme a active le dark mode du DSFR, qui a des contrastes insuffisants. On a passe du temps a chercher un probleme CSS alors que la solution etait de ne pas proposer le dark mode du tout.

---

## 7. Pieges a eviter

**Ne jamais corriger une fuite memoire sans tester le fonctionnel** : revoquer un ObjectURL "pour la securite" peut casser le flux de donnees. Toujours tester apres.

**Les erreurs console en `file://` sont bruyantes** : Chrome genere des warnings `Unsafe attempt to load URL file://...` et `blob:null` qui ne sont pas des erreurs reelles. Ne pas partir en debug la-dessus sans d'abord verifier si le comportement attendu fonctionne.

**Le DSFR dark mode n'est pas conforme WCAG** (en 1.11.2) : ne pas ajouter de toggle theme si on ne peut pas garantir les contrastes dans les deux modes. Tester les deux themes avant de livrer.

**`dsfr.min.js` n'existe pas** sur le CDN jsdelivr. Les bons fichiers sont `dsfr.module.min.js` (bloque en `file://`) et `dsfr.nomodule.min.js` (fonctionne partout). Verifier les 404 avant d'integrer un CDN.

**Un outil d'audit ne suffit jamais** : axe-core disait "0 violation" a chaque passe. Les vrais problemes (hierarchie de titres, contrastes dark mode, `aria-label` vs texte visible, `label` sur input hidden) ont tous ete trouves par d'autres outils ou par inspection manuelle.

**Ne pas empiler 10 corrections avant de tester** : on a parfois applique accessibilite + securite + DSFR + typographie en une seule passe, puis passe des heures a debugger. Un cycle plus court (corriger, tester, valider, suivant) aurait ete plus efficace.

---

## 8. Regard expert

Un expert remarquerait que **la comparaison de pixels est naive** : 1 500 pixels echantillonnes avec une distance de Manhattan et un seuil fixe. Ca fonctionne pour des presentations avec des slides bien distinctes, mais ca raterait des changements subtils (une ligne de texte ajoutee, un graphique qui evolue legerement). Un algorithme de hachage perceptuel (pHash, dHash) serait plus robuste pour quelques lignes de code en plus.

Un expert noterait aussi que **l'accumulation de dataURL en memoire est la vraie limite d'echelle**. Pour un outil qui traite des videos de 2 heures avec 500+ slides uniques, il faudrait stocker des Blob et creer les ObjectURL a la demande. C'est le seul point qui empeche l'outil de passer a l'echelle.

Enfin, un expert verrait que **la classe `VideoFrameExtractor` fait tout** : DOM, extraction, comparaison, telechargement, accessibilite, theme. En 1 469 lignes, c'est tolerable pour un fichier standalone. Mais si l'outil grandissait (ajout de formats, previsualisation, annotation), il faudrait separer la logique metier (extraction, comparaison) de la presentation (DOM, affichage).

---

## 9. Lecons transferables

**Corriger la cause, pas le symptome** : le timeout de 339 secondes n'etait pas trop court. Le traitement etait trop lent. Ce reflexe s'applique partout — un serveur qui rame ne se corrige pas en augmentant le timeout des requetes.

**L'accessibilite est structurelle, pas cosmetique** : les titres, les landmarks, les labels — tout ca se met en place AVANT le design. Ajouter l'accessibilite apres coup, c'est comme ajouter un ascenseur dans un immeuble deja construit : faisable, mais beaucoup plus cher.

**La securite casse des choses** : chaque durcissement (CSP, revocation ObjectURL, suppression innerHTML) a introduit un bug. C'est normal — la securite restreint ce que le code peut faire, et le code existant fait souvent des choses qui ne devraient pas etre autorisees. Le reflexe est de tester systematiquement apres chaque correction de securite.

**Multiplier les outils de verification** : un validateur W3C, un audit axe-core, un Tanaguru, un DevTools — chacun voit des choses differentes. La conformite, c'est l'intersection de tous ces resultats, pas le resultat d'un seul.

**Le contexte `file://` est un piege** : les blob URLs, les modules ES, les CORS, les CSP — tout se comporte differemment en `file://` qu'en `http://`. Si l'outil doit fonctionner en double-clic, tester en `file://` des le premier jour, pas apres avoir tout construit.

---

*Document genere le 2 avril 2026 — session extracteur de diapositives video*
