# Néon Mano

Dessine de la musique dans l'air avec ta webcam. Tu pinces le pouce et l'index pour tracer des néons, et une barre lumineuse qui balaie l'écran en boucle les transforme en mélodie, calée sur une gamme pentatonique.

**Jouer :** https://schenkelqpro-afk.github.io/neon-mano/ (Chrome sur ordinateur, caméra autorisée)

## Ce que fait la V1

- Suivi de la main par webcam (MediaPipe Hands, jusqu'à 2 mains). Seuls le pouce et l'index dessinent ; les pincements trop brefs et le crochet de fin de trait sont filtrés.
- Haut = aigu, bas = grave. 4 gammes (pentatonique mineure, majeure, blues, japonaise) et 12 tonalités.
- 5 sons synthétisés : synthé rétro, chiptune, nappe douce, pluck, marimba. Batterie optionnelle.
- 5 morceaux modèles avec traces guides en pointillés, score de suivi et bouton « Tracer pour moi ».
- Couleur des néons et de la barre de lecture au choix.
- Enregistrement vidéo (MP4 ou WebM selon le navigateur) et export audio WAV de 4 boucles.
- Sans caméra : dessin à la souris.

Raccourcis : Espace lecture/pause, Z annuler, Suppr tout effacer, R enregistrer, M modèles.

Tout tourne dans le navigateur : l'image de la caméra n'est envoyée nulle part.
