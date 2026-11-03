# Réflexions TD2 — Smart Mobility Le Mans

## 1. Sous-questions faciles vs difficiles (avant de construire)
je pense que le plus facile sera de voir l'état du réseau en ce moment c'est a dire quelles lignes sont en retard. Le plus difficile sera de savoir où sont les vrais besoins, parce qu'on n'a pas les données de fréquentation, et de trouver des tendances récurrentes, parce qu'il faut plusieurs semaines d'historique.

## 2. Pourquoi le flux statique n'est pas rafraîchi toutes les 15 minutes (prédiction)
Les horaires théoriques ne changent presque jamais, la SETRAM les met à jour seulement de temps en temps. Les télécharger toutes les 15 minutes donnerait toujours le même fichier, ce serait du gaspillage.

## 3. Jointure sur des colonnes d'ID typées différemment (prédiction)
Si un fichier lit `stop_id` comme un nombre (1001) et l'autre comme du texte ("1001"), Python les considère comme différents. La jointure ne trouve pas de correspondance, ou plante, et on perd des arrêts sans forcément s'en rendre compte. C'est pour ça que le script lit tout en texte avec `dtype=str`.


## 4. Volume de lignes : par ligne de bus vs par arrêt
Par arrêt : 5000 lignes × 96 relevés par jour × 84 jours ~ 40 millions de lignes, beaucoup trop pour un fichier dans Git. Par ligne de bus : 30 × 96 × 84 ~ 240 000 lignes, ça reste raisonnable. Mais on ne peut plus savoir à quels arrêts précis les retards se produisent.

## 5. Alternative au code couleur du graphique des retards
je mettrais le vert autour de 0 (à l'heure), et du rouge des deux côtés : quand le bus est très en retard, mais aussi quand il est très en avance. Avec la règle actuelle, une ligne qui passe 3 minutes en avance a l'air parfaite, alors que les gens ratent leur bus. Avec ma règle, on verrait que cette ligne a aussi un problème de régularité.

## 6. Raisonnement réel sur le décalage de fréquence statique / temps réel
Statique toutes les 15 min : inutile, car les horaires changent quelques fois par an, on referait le même calcul pour rien et on remplirait Git de commits identiques.
Temps réel une fois par mois : on aurait une seule photo du réseau par mois, prise à une heure au hasard. Les retards changent toutes les minutes, donc on raterait les heures de pointe et les perturbations, et l'historique ne voudrait rien dire.
Donc la fréquence doit suivre la vitesse à laquelle la donnée change.

## 7. Vérification d'une ligne en retard par rapport à la réalité
le 21/10 vers 11h40, la ligne 5 était la plus en retard avec environ 2 minutes de retard moyen. Ce n'est pas l'heure de pointe, donc c'est un peu surprenant, mais 2 minutes reste un retard normal en ville. Ça me paraît crédible.

Le cas le plus bizarre est la ligne 34, avec environ 16 minutes d'avance en moyenne. Un bus ne roule pas 16 minutes en avance en réalité, il attendrait à un arrêt. Je pense plutôt que c'est un artefact de données : par exemple un trajet mal recalé par rapport à l'horaire théorique, ou très peu de trajets observés, donc une seule valeur aberrante suffit à fausser la moyenne. Cette seule ligne tire aussi la moyenne du réseau à -41,6 s, ce qui donne une fausse impression que tout le réseau est en avance
15 minutes plus tard (relevé de 09:53 UTC), la ligne 34 n'était plus qu'à environ 37 secondes d'avance, ce qui confirme que les 16 minutes étaient une anomalie ponctuelle et pas un vrai problème de la ligne. La ligne 5 était toujours la plus en retard dans les deux relevés, donc ce retard paraît plus fiable. Ça montre qu'il ne faut pas tirer de conclusion à partir d'un seul relevé.
## 8. Cas où la population trompe comme indicateur de demande
Une commune peut avoir peu d'habitants mais une forte demande de transport si elle accueille un lieu qui attire du monde tous les jours; une zone commerciale, une zone industrielle, un hôpital ou un campus. Par exemple, le campus universitaire au nord-ouest du Mans fait venir chaque jour des milliers d'étudiants et de salariés qui habitent ailleurs, alors que ces trajets n'apparaissent pas dans la population de la commune. Les grands événements comme les 24 Heures du Mans créent aussi une demande énorme mais ponctuelle.
À l'inverse, une commune assez peuplée où presque tout le monde a une voiture peut avoir une demande réelle plus faible que ce que sa population laisse penser.

## 9. Mon indicateur offre / population, et une commune où il trompe
Mon indicateur : nombre de passages de bus/tram prévus par jour aux arrêts d'une commune, pour 1 000 habitants (sur un mardi type).
Dans mon résultat, La Chapelle-Saint-Aubin arrive première avec 1 010 passages pour 1 000 habitants et 53 arrêts pour seulement 2 319 habitants, alors que Le Mans n'arrive que 10e (153,8), derrière plusieurs petites communes. C'est clairement faux : Le Mans, avec le tram et le centre-ville, est la commune la mieux desservie.

L'erreur vient de la façon dont les arrêts sont attribués aux communes : le script prend le centre de commune le plus proche, pas la vraie limite communale. Le Mans est une grande ville représentée par un seul point au centre-ville, donc les arrêts de ses quartiers périphériques (par exemple autour de l'université, desservie par le tram) sont plus proches du centre de La Chapelle-Saint-Aubin et lui sont attribués. Les communes voisines (La Chapelle-Saint-Aubin, Coulaines, Rouillon) sont donc surestimées et Le Mans est sous-estimé.

Pour corriger ça, il faudrait utiliser les vrais contours des communes et vérifier dans quel polygone se trouve chaque arrêt.


## 10. Agréger les retards par arrêt sans exploser l'historique
Au lieu de garder chaque relevé brut, je regrouperais les données par arrêt et par jour, ou par tranche horaire (matin, midi, soir). Pour chaque arrêt, je garderais seulement le nombre de relevés, le retard moyen et le nombre de fois où le retard a dépassé 5 minutes. On pourrait aussi ne garder que les arrêts qui ont eu un retard important. Avec un fichier par mois, on passerait de centaines de milliers de lignes par jour à quelques milliers, ce qui reste raisonnable dans Git tout en permettant de repérer les arrêts souvent en retard.

## 11. Comparaison avec mes prédictions de départ
j'avais raison sur le fait que l'état du réseau en direct était facile à obtenir : en quelques minutes j'avais les retards par ligne. Mais même là, il a fallu du jugement, par exemple pour repérer que les 16 minutes d'avance de la ligne 34 étaient une anomalie.

La partie offre / demande a demandé beaucoup plus de choix que prévu : choisir l'indicateur, utiliser la population à la place de la vraie fréquentation, et surtout la façon d'attribuer les arrêts aux communes, qui fausse complètement le classement.

Enfin, les tendances récurrentes (quelles lignes sont souvent en retard, à quelles heures) ne sont toujours pas exploitables : je n'ai que quelques relevés d'une seule matinée. Il faudra plusieurs semaines d'historique avant de pouvoir répondre à la question de départ sur « où et quand » intervenir.
