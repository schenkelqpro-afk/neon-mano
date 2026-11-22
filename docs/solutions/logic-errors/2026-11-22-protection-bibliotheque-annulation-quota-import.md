---
title: "Bibliothèque localStorage : annulation qui écrase une création, quota plein, import piégé"
category: logic-errors
date: 2026-11-22
problem_type: logic_error
component: frontend_stimulus  # le schéma n'a pas de valeur « JS navigateur sans framework » ; c'est la plus proche
root_cause: logic_error
resolution_type: code_fix
severity: high
tags: [undo, historique, localStorage, quota, import-json, prototype, css-url, pointer-events, double-clic, hermite, pagehide, playwright]
module: index.html (Néon Mano)
commit: 80eeed3
related:
  - /mnt/project-files/neon-mano/2026-11-22-revue-code-neon-mano.md
---

# Bibliothèque localStorage : annulation qui écrase une création, quota plein, import piégé

## Problème

La bibliothèque de Néon Mano est enregistrée dans `localStorage`. Une revue de code (ce:review, 7 relecteurs) y a trouvé trois manières de perdre ou de casser une création :

1. **Écraser une création avec l'annulation.** L'historique d'annulation ne mémorisait que les traits, pas la création ouverte. Ouvrir la création B, appuyer une fois de trop sur Z, puis faire « Mettre à jour » ou Cmd+S remplaçait B par le dessin précédent.
2. **Perdre du travail quand le stockage est plein.** Plusieurs échecs se combinaient sans que l'utilisateur le voie.
3. **Casser l'app avec un fichier importé.** Un fichier JSON abîmé ou piégé pouvait figer l'écran pour de bon, même après rechargement, ou faire charger une adresse externe.

## Symptômes

- **Annulation :** après « ouvrir B, puis Z, puis Cmd+S », la création B contient les traits d'une autre création. La liste affiche « B : modifications non enregistrées », ce qui ne permet pas de deviner ce qui s'est passé.
- **Stockage plein (environ 5 Mo, partagé par tous les projets GitHub Pages du même compte) :**
  - la sauvegarde automatique échoue en silence (`catch` vide) ;
  - l'avertissement est remplacé tout de suite par le message suivant (« Tout est effacé ») ;
  - le dessin est quand même effacé ;
  - la liste montre des créations qui n'existent qu'en mémoire.
- **Import :** un trait avec `pts: []`, ou un instrument nommé `"toString"`, fait lever une exception dans `frame()`. Comme `requestAnimationFrame(frame)` est la dernière ligne de `frame()`, l'animation ne repart jamais. L'état fautif étant sauvegardé, l'écran reste figé à chaque rechargement.

## Ce qui n'a pas marché, ou était trompeur

- **Vérifier avec `SOUNDS[x]`.** Ce test laisse passer les clés héritées d'`Object.prototype` (`toString`, `constructor`, `__proto__`). Il « validait » donc des valeurs qui cassent tout.
- **Valider seulement les coordonnées.** `[].every(...)` vaut `true` sur un tableau vide, donc un trait sans aucun point passait la validation.
- **Faire confiance aux couleurs importées.** Une propriété CSS personnalisée accepte n'importe quel texte. `--head: url(https://…)` déclenche donc une requête dès qu'un `background: var(--head)` est affiché.
- **Détecter le double-clic avec `e.detail` sur `pointerdown`.** Chrome renvoie toujours `0` pour ce champ, donc le double-clic n'était jamais détecté.
- **Dans les tests Playwright, écrire dans `localStorage` puis faire `reload()`.** Le nouveau gestionnaire `pagehide` (qui force la sauvegarde) réécrit l'état courant par-dessus ce que le test vient d'injecter. Le test semble alors montrer une régression qui n'existe pas.

## Solution

### 1. L'historique mémorise aussi la création ouverte

```js
function pushHistory() {
  history.push({ strokes: state.strokes, libId: state.libId });
  if (history.length > 120) history.shift();
}
function popHistory() {
  const h = history.pop();
  state.strokes = h.strokes;
  // revenu avant l'ouverture : ce dessin n'est plus cette création
  if (h.libId !== state.libId) state.libId = null;
  return h;
}
```

Tous les endroits qui écrivaient `history.push(state.strokes)` passent maintenant par `pushHistory()` : `commit`, `beginMove`, `beginResize`, `editableCopy`. Le retrait du « tap » au double-clic passe par `popHistory()`.

Avant toute annulation ou suppression, `endGestures()` termine le déplacement ou le glisser de point en cours. Sinon le geste continuerait d'écrire dans un objet déjà restauré, et donc dans les entrées d'historique qui le partagent.

### 2. Stockage plein : rien n'est effacé tant que rien n'est mis à l'abri

- **Retirer les plus vieux brouillons.** `saveLib(keepId)` retire les brouillons automatiques les plus anciens (`draft: true`), jamais une création enregistrée par l'utilisateur, puis réessaie.
- **Bloquer l'effacement si le brouillon échoue.** `protectCurrent()` renvoie `false` quand le brouillon n'a pas pu être écrit. `clearAll`, `openTemplate` et `openCreation` s'arrêtent alors sans rien effacer.
- **Ne pas garder en mémoire ce qui n'est pas écrit.** `saveCreation` et `importFile` annulent leur changement en mémoire si l'écriture échoue. La liste reflète ainsi exactement ce qui est stocké.
- **Garder l'avertissement visible.** `toast(msg, ms)` accepte une durée. Un message de plus de 2,4 s ne peut pas être remplacé par un message ordinaire pendant qu'il s'affiche.
- **Corriger `openTemplate`.** Il appelait `protectCurrent()` après avoir déjà changé la gamme et le tempo, si bien que le brouillon était enregistré avec les réglages du modèle. L'appel est maintenant fait en premier.

### 3. Import ramené à une forme connue, boucle de dessin protégée

```js
const own = (o, k) => typeof k === 'string' && Object.prototype.hasOwnProperty.call(o, k);
const isHex = (c) => typeof c === 'string' && /^#[0-9a-f]{6}$/i.test(c);
const okPts = (a) => Array.isArray(a) && a.length > 0 &&
  a.every(p => Array.isArray(p) && p.length === 2 && p.every(Number.isFinite));
```

- **Partout où une valeur importée sert à chercher dans une table :** `own(SOUNDS|SCALES|DRUMS, k)` remplace `SOUNDS[k]`.
- **`cleanItem(it)`** reconstruit chaque création importée champ par champ, au lieu de recopier l'objet tel quel (`{ ...it }`) :
  - couleurs au format `#rrggbb` uniquement, et seulement pour les instruments connus ;
  - tonalité bornée de 0 à 11 ;
  - tempo borné de 50 à 180 ;
  - traits vides retirés ;
  - un trait sans instrument prend le son de la création, comme le faisaient les anciennes versions.
- **`loadLib()` et `load()`** appliquent le même nettoyage aux données déjà stockées. Une ancienne valeur fautive ne peut donc plus revenir au rechargement.
- **`frame()`** est maintenant `try { drawFrame() } catch {} requestAnimationFrame(frame)`. Une erreur de dessin ne fige plus l'écran.

## Pourquoi ça marche

- **Annulation :** la cause était qu'une annulation peut revenir à un moment où une autre création était ouverte, sans que l'app le sache. Chaque entrée d'historique sait maintenant quelle création était ouverte. En revenant à un moment où c'en était une autre, le dessin est détaché (`libId = null`), et l'enregistrement crée une nouvelle création au lieu d'écraser B.
- **Stockage plein :** l'ordre est maintenant « écrire, et seulement ensuite effacer ou afficher ».
- **Import :** on construit une forme sûre au lieu de seulement vérifier que la forme reçue est correcte. Tout champ inattendu disparaît, et chaque champ attendu a une valeur garantie.

## Prévention

- **Toute action qui change de document fait passer par `pushHistory()`.** C'est le cas d'ouvrir, de vider et d'ouvrir un modèle. Tout nouvel état qui définit le document courant (par exemple un futur « projet ») doit être ajouté à l'entrée d'historique.
- **Ne jamais chercher une valeur venue de l'extérieur dans un objet littéral avec `obj[k]`.** Utiliser `own()` ou `Object.create(null)`.
- **Ne jamais passer à `style.setProperty` une couleur venue d'un fichier ou du stockage** sans la vérifier avec `isHex()`.
- **Toute écriture dans `localStorage` peut échouer.** Changer la liste en mémoire seulement après une écriture réussie, ou revenir en arrière en cas d'échec.
- **Tests de non-régression** (Playwright, voir `test15.js` dans le scratchpad de la session) :
  - « ouvrir A, Z, Cmd+S » : la création A est inchangée ;
  - remplacer `Storage.prototype.setItem` par une version qui lève `QuotaExceededError` : `clearAll` n'efface rien et la liste ne change pas ;
  - importer `{ inst: 'toString', head: 'url(...)', pts: [] }` : l'élément est refusé ou nettoyé, et il n'y a aucune erreur de page ;
  - un trait sur toute la largeur joue deux notes de 16 pas (voir ci-dessous).

## Autres apprentissages de la session

- **Notes tenues plus de 16 pas.** `buildScore` limitait la durée d'une note à 16 pas, mais considérait chaque pas suivant comme « tenu ». Une ligne sur toute la largeur restait donc muette sur la 2e mesure. Correctif : compter la longueur de la tenue (`run`) et relancer la note quand `run % 16 === 0`.
- **Double-clic avec les pointer events.** Il faut un compteur maison (moins de 450 ms et moins de 10 px entre deux `pointerdown`). Le premier clic d'un double-clic crée un « tap » qu'il faut retirer ensuite. Il faut aussi traiter comme un tap tout trait souris de moins de 8 px : un clic qui tremble laisse 2 points.
- **Édition de courbe par points.** Utiliser une courbe Hermite monotone (tangente nulle aux extrêmes, limite de Fritsch–Carlson, chaque point de la courbe borné entre ses deux points voisins). La courbe ne dépasse jamais la hauteur d'un point, donc un point aimanté sur une note joue exactement cette note. Garder `ctrl` comme référence et recalculer `pts`. Toute transformation (déplacer, agrandir) doit s'appliquer aux deux.
- **Scripts de test et sauvegarde à la fermeture.** Pour injecter un état de test, bloquer d'abord le gestionnaire de fermeture : `addEventListener('pagehide', e => e.stopImmediatePropagation(), true)`, puis écrire dans `localStorage`, puis faire `reload()`.
- **Clic de sélection en mode Déplacer.** Il ne doit laisser aucune trace. `beginMove` empile une entrée d'historique par précaution, et `endMove` l'enlève si rien n'a bougé (`moving.moved === false`).

## Encore ouvert (voir le rapport de revue)

- performance des glisser (`rebuildLayer` est relancé à chaque image) ;
- deux onglets ouverts qui s'écrasent mutuellement la bibliothèque ;
- arrêts silencieux du suivi de la main (CDN, caméra, GPU) ;
- tests versionnés dans le dépôt ;
- contrôle d'intégrité (SRI) du code chargé depuis le CDN.
