
# Essayez de dessiner

## Redimensionnement

Pour interagir avec notre écran, il nous faut un tableau de 128×64, ce qui représente 8192 valeurs booléennes (vrai ou faux).

Pour un écran en couleur, on utilise généralement 3 octets (bytes) allant de 0 à 255, appelés RGB.
On parle de RGBA (4 octets) si on doit prendre en compte la transparence.

Dans notre cas, nous allons utiliser des booléens `Array[bool]`.
Cela représente donc 8192 octets, soit 65 536 bits.

Pourquoi utilise-t-on 256 valeurs possibles pour stocker seulement deux états (0 ou 1) ?
Un bit seul n’existe pas vraiment dans l’architecture classique des ordinateurs…
C’est une longue histoire.

On pourrait utiliser un `PackedByteArray` de Godot et faire des opérations binaires pour stocker les données bit par bit.
Mais on verra ça plus tard. Pour le moment, on privilégie la simplicité plutôt que l’optimisation.

```gdscript
## J'existe dans la scène de Godot
extends Node

## Je crée un tableau nommé screen (écran)
## Il contient des valeurs booléennes (vrai ou faux)
var screen : Array[bool] = []

## Quand le script démarre
func _ready():
  ## On réserve de la mémoire pour 8192 éléments
  ## Cela correspond à une zone de RAM de 8192 octets
  ## C'est 8 fois trop, mais ça fera l'affaire
  screen.resize(128 * 64)
```

---

# Envoyer des tableaux à des inconnus

Ce tableau, on aimerait le transmettre à notre écran OLED SSD1306 128×64.
Cela peut être un simulateur 2D/3D ou un écran réel.

On délègue donc la responsabilité :

> "Prenez ce tableau et débrouillez-vous."

```gdscript
extends Node

## Signal pour demander la mise à jour de l'écran
signal on_request_to_update_screen_128x64(bit_array: Array[bool])

var screen : Array[bool] = []

func _ready():
  ## Création du tableau
  screen.resize(128 * 64)

  ## Envoi du tableau à ceux qui écoutent
  on_request_to_update_screen_128x64.emit(screen)
```

---

# Du signal à notre SSD1306

Dans l’éditeur, il faut relier ce signal à une méthode capable de dessiner sur l’écran.

(Ajouter les images ici)

---

# Modding

Si vous faites l’exercice en WebGL ou en exécutable hors de l’éditeur Godot :

Voir Release :
*Ajouter le lien ici*

Vous devez utiliser un nom de signal spécifique :

```
on_screen_update_request
```

---

# Un pixel

Essayons de transmettre un tableau avec uniquement le premier pixel allumé.

Un tableau 2D est en réalité stocké comme un tableau 1D (une liste).

* Le premier élément est à l’index `0` (en haut à gauche)
* Le dernier est à `8191`
* En informatique, on commence généralement à compter à partir de 0

Essayons d’allumer le pixel 0 :

```gdscript
## Ceci est un commentaire
## Texte non exécuté

"""
Commentaire multi-lignes
"""

#region Minimum viable

extends Node

signal on_request_to_update_screen_128x64(bit_array: Array[bool])
var screen : Array[bool] = []

## Envoie le tableau à l'écran
func _push_screen():
  on_request_to_update_screen_128x64.emit(screen)

## Redimensionne le tableau
func _sized_array():
  screen.resize(128 * 64)

## Initialisation
func _ready():
  _sized_array()
  my_code()
  _push_screen()

#endregion

#region NOTRE CODE

func my_code():
  screen[0] = true
  ## Pour le fun
  screen[8191] = true

#endregion
```


------------------

# Milestone: Vous savez allumer un pixel

Si vous avez réussi à afficher deux pixels sur un écran SSD1306 dans Godot,
alors tout est bon et vous pouvez passer à la suite.

Après avoir essayé sous différents angles, demandez de l’aide si vous êtes bloqué.
En classe, si je vous donne cours sur Discord, ou dans les commentaires YouTube si vous êtes en ligne.

---------------------

# Baladons-nous sur les X

<img width="3" height="3" alt="image" src="https://github.com/user-attachments/assets/dca5291a-5cc1-4265-bc05-f3a3c4066bf4" />

De gauche à droite, nous avons la ligne 1 de 0 à 127 pour 128 pixels.

On veut donc allumer le 127e pixel.

Sans regarder, ça fait quoi à votre avis si on allume le 129e pixel ?

```gdscript
func my_code():
  screen[127] = true
  screen[129] = true
```

# Une ligne

<img width="128" height="4" alt="image" src="https://github.com/user-attachments/assets/45dd21c4-5d2b-40e4-988f-1b3a569929cc" />

Essayons d’allumer la ligne du dessus.

On pourrait écrire 128 fois screen[0] = true   
Mais comme on est des informaticiens et donc que l’on a la flemme,   
on va utiliser une boucle qui fait x fois ce qu’on lui demande.  

```gdscript
func my_code():
  for i in range(0,128):
    screen[i] = true
```

range(0,128) va me créer une liste comme ceci :
var ma_list: Array[int] = [0,1,2,3,4,5,6...,127,128]

`for i in` veut dire : pour chaque élément dans ma_list,
je vais stocker le nombre dans un tiroir que je nomme `i` et je fais l’action qui suit.

En gros.

Faut pas chercher quand on débute, juste utiliser et comprendre plus tard.

Comment vous feriez si vous deviez faire une ligne de 20 cases sur la gauche et de 20 cases sur la droite ?

# Baladons-nous sur les Y du dessus au-dessous

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/f9277533-108a-4565-a948-85c690dbca27" />

Du coup, comment on fait pour allumer le coin à gauche et à droite sur la dernière ligne ?

À gauche c’est 0 et à droite 127 sur la ligne 1.
128 c’est à gauche sur la ligne 2.
De fait... la droite sur la ligne deux serait... ?
127 + 128 = 255

Sur la ligne 3 ?
3: 127 + 128 + 128 = 383
4: 127 + 128 + 128 + 128 = 511
5: 127 + 128 + 128 + 128 + 128 = 639

En algèbre 😜, ça donnerait :
- `x + y * 128 = c`
- `colonne + ligne × 128 cases` = `index du tableau à une dimension`

Nous, on veut la ligne 64 (-1) sur la colonne 0 et 127

0 = 63 * 128 = 8064
127 + 63 * 128 = 8191

```gdscript
func my_code():
  screen[0] = true
  screen[127] = true
  screen[8191] = true
  screen[8064] = true
  ## ou
  screen[63*128] = true
  # pour le pixel suivant
  screen[63*128 + 1] = true
```

## Vous savez la suite 😉

<img width="2" height="64" alt="image" src="https://github.com/user-attachments/assets/71823194-f51e-4c22-8702-ddedad8baa4e" />

Essayons d’allumer la ligne de droite de notre écran.

```gdscript
func my_code():
  screen[0] = true
  screen[127+1] = true
  screen[127+127+1] = true
  screen[127+127+127+1] = true
  ## copier 64 fois xD
  screen[128 * 5 + 1] = true
  ## Bon, si vous écrivez ça, c’est que les for vous font peur ;)

  # Essayons cela
  for i in range(64):
    screen[i*128+1]
```


---------------------------


# Temporaire, tout les exercices en images sans commentaire

## Essayer de colorer les bors sur 1 pixels puis sur 2










