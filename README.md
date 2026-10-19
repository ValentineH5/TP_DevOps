# TP_DevOps

# TP partie 01 - Docker

1-1 Pour quelle raison est-il préférable de faire tourner le conteneur avec un drapeau -e pour donner les variables d'environnement plutôt que de les mettre directement dans le Dockerfile?

Pour des questions de sécurité, il vaut mieux utiliser un drapeau -e pour les variables d'environnement plutôt que de les mettre directement dans le Dockerfile. Cela évite qu'elles soient directement visibles dans le Dockerfile. Cela permet également de pouvoir modifier les variables d'environnement sans devopir reconstruire toute l'image Docker.

