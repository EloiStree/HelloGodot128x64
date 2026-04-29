# Atelier : créer un quad 128×64

> Écrivons un script qui prend un tableau de booléens en entrée et le transforme en texture 2D affichée sur un quad.

Si vous aimez le chaos organisé, vous pouvez tout mettre dans un seul script.  
Sinon, pour quelque chose d’un peu plus propre, on peut le découper en quatre scripts :  
* **La ressource** : contient les données du tableau et peut être partagée entre plusieurs scripts
* **Le convertisseur** : reçoit le tableau et génère une texture2D
* **Le peintre** : prend la texture2D et l’applique au quad pour l’affichage
* **Le contrôleur** : stocke la ressource, envoie les données au convertisseur, puis transmet la texture au peintre
