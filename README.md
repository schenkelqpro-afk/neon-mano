# Néon Mano

Dessine de la musique dans l'air avec ta webcam. Tu pinces le pouce et l'index pour tracer des néons, et une barre lumineuse qui balaie l'écran en boucle les transforme en mélodie, calée sur une gamme pentatonique.

**Jouer :** https://schenkelqpro-afk.github.io/neon-mano/ (Chrome sur ordinateur, caméra autorisée)

## Ce que fait la V1

- Suivi de la main par webcam (MediaPipe Hands, jusqu'à 2 mains). Seuls le pouce et l'index dessinent ; les pincements trop brefs et le crochet de fin de trait sont filtrés.
- Haut = aigu, bas = grave. 4 gammes (pentatonique mineure, majeure, blues, japonaise) et 12 tonalités.
- 7 instruments, un par trait et chacun avec sa couleur : synthé rétro, synthé glissé (hauteur continue qui suit le trait), chiptune, nappe douce, pluck, marimba, batterie. Rythme automatique optionnel.
- 5 morceaux modèles avec traces guides en pointillés, score de suivi et bouton « Tracer pour moi ».
- Couleur des néons et de la barre de lecture au choix.
- Enregistrement vidéo (MP4 ou WebM selon le navigateur) et export audio WAV de 4 boucles.
- Bibliothèque : enregistrer ses créations (dessin, son, gamme, tempo, couleurs), les rouvrir, renommer, supprimer. Brouillon automatique avant d'effacer ou d'ouvrir autre chose. Export et import en fichier .json pour sauvegarder ou partager.
- Sans caméra : dessin à la souris.

Barre de lecture : tire-la (poignée en haut) à la souris, ou en pause en la pinçant, pour repartir d'où tu veux.

Déplacer les formes : bouton « Déplacer » (V), puis attraper une forme à la souris ou en la pinçant. Flèches ← → : un pas de rythme, ↑ ↓ : une note. Suppr efface la forme sélectionnée. Poignées aux coins (ou touches + et −, Maj + flèches pour un seul sens) pour agrandir ou rapetisser. Changer d'instrument dans ce mode change celui de la forme sélectionnée.

Instruments et couleurs : chaque trait garde l'instrument choisi au moment où il est dessiné, avec sa couleur (réglable par instrument). L'instrument Batterie joue selon la hauteur : grosse caisse, tom, caisse claire, clap, charleston, charleston ouvert.

Métronome : bouton (T), un clic sur chaque temps, accentué en début de mesure, volume réglable sur un curseur vertical tout à droite de l'écran (visible quand le métronome est actif) ou dans Réglages. Il n'apparaît ni dans les vidéos ni dans le WAV.

Édition précise : double-clic sur un trait pour faire apparaître ses points. Glisse un point, il se cale sur la note et le temps les plus proches (Alt pour le placer librement). Double-clic sur le trait pour ajouter un point, clic droit ou Suppr pour en retirer un, flèches pour le décaler d'un pas ou d'une note, Échap ou Terminer pour finir. La croix masque le bandeau d'aide sans quitter l'édition (choix retenu) ; un clic sur « Édition des points » en haut le réaffiche. La musique continue pendant l'édition.

Bande de rythme : quand le métronome est actif, les 8 temps de la boucle (1 2 3 4, 1 2 3 4) s'affichent en bas de l'écran, avec un repère vertical par temps dans la grille, et le temps en cours s'allume.

Pause du suivi : bouton main (H). Plus rien ne s'écrit pendant que tu réfléchis, la musique continue. En reprenant, une main déjà pincée doit d'abord s'ouvrir avant de dessiner.

Raccourcis : Espace lecture/pause, H pause du suivi, T métronome, V déplacer, Début revenir au départ, Z annuler, Suppr tout effacer, R enregistrer, M modèles, B bibliothèque, Ctrl/Cmd+S enregistrer la création.

Tout tourne dans le navigateur : l'image de la caméra n'est envoyée nulle part.
