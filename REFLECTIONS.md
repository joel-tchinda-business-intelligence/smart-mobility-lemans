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

## 8. Cas où la population trompe comme indicateur de demande

## 9. Mon indicateur offre / population, et une commune où il trompe

## 10. Agréger les retards par arrêt sans exploser l'historique

## 11. Comparaison avec mes prédictions de départ
