
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

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/3337077c-315c-414a-89c4-76e130f180d1" />


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

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/19effa46-114c-4793-a726-b5d199c93d90" />

De gauche à droite, nous avons la ligne 1 de 0 à 127 pour 128 pixels.

On veut donc allumer le 127e pixel.

Sans regarder, ça fait quoi à votre avis si on allume le 129e pixel ?

```gdscript
func my_code():
  screen[127] = true
  screen[129] = true
```

# Une ligne

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/120dcb33-92d7-44d7-b265-1aab0c2c1676" />


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


<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/139965d0-3dd5-4060-be2f-a5feaddcda60" />

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

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/af6b07bf-109b-46f9-8e44-f4afef180d7e" />

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


## Essayez de remplir l’écran de couleur

À faire

---

## Créons une méthode turn off et turn on

À faire

---

## Utilisons await pour le faire clignoter 😁

À faire


---------------------------


# Temporaire, tous les exercices en images sans commentaire

## Essayer de colorer les bords sur 1 pixel puis sur 2

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/07255051-7461-41a5-bfe5-91353d8634cd" />

---

# Un centre avec une valeur paire ?

Quatre points <img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/e7498f1f-9816-4ffa-a6e1-966e7a0f1b0b" />

---

# Dessinez une ligne à partir d’un point

Vers la droite

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/ee4db2c1-40d8-4c53-9c67-e8453737148d" />

Vers le dessus

<img width="128" height="64" alt="image" src="https://github.com/user-attachments/assets/04b2eb80-7218-455c-9e26-640a5e36934b" />

Quatre méthodes pour chaque direction

<img width="50" height="44" alt="image" src="https://github.com/user-attachments/assets/93bdd271-7b6d-47ce-8d01-5fa69b45c71a" />

---

# Dessinons un rectangle plein

<img width="55" height="24" alt="image" src="https://github.com/user-attachments/assets/c103dc69-abcb-4a8f-bb94-c031488bd852" />

* En dessinant case par case
* En utilisant notre méthode pour dessiner une ligne

Bonus :

* Essayer de dessiner un pixel sur deux
* Essayer de dessiner un pixel sur deux avec un décalage sur les lignes impaires

---

# Dessinons les bords d’un rectangle

<img width="60" height="32" alt="image" src="https://github.com/user-attachments/assets/d5a2ae03-0990-4f9a-a472-966878c261d5" />

Utiliser ce code pour remplir la moitié de gauche

<img width="70" height="64" alt="image" src="https://github.com/user-attachments/assets/92d5ed31-e106-4d23-935c-00b0d4ac1a7c" />

Bonus : créer un faux code-barres 😅
Challenge : créer un QR code valide à partir d’une image
Expert : créer un QR code valide à partir d’un texte donné

---

# Une ligne

<img width="66" height="49" alt="image" src="https://github.com/user-attachments/assets/dee26f58-cc02-4783-8878-d3f078068af0" />

Si vous avez peur de la trigonométrie,
commencez par une diagonale 😜

Puis revenez après un bon café…
et utilisez la trigonométrie 🍻🔥

Montrez-moi que votre prof de maths peut être fier de vous

---

# Un triangle… rectangle 😉

<img width="41" height="23" alt="image" src="https://github.com/user-attachments/assets/50e54369-6154-457b-a645-3536994c23fa" />

En gros, votre code pour des lignes verticales et horizontales,
suivi de votre nouveau code pour faire des lignes non droites

---

# Alors… un octogone

<img width="60" height="46" alt="image" src="https://github.com/user-attachments/assets/63da8f54-d9a9-410e-b951-23f277ac0e7c" />

Mais si vous voulez, vous pouvez aussi faire un hexagone…

---

# Hey hey… Un cercle 😁

Bah oui, on n’allait pas le laisser de côté.
Si vous avez fait l’octogone avec des sin et cos, vous y êtes presque.

<img width="72" height="59" alt="image" src="https://github.com/user-attachments/assets/6048b894-40be-4e63-af3e-cf49bc003973" />

---

# Le drapeau japonais ?

<img width="62" height="61" alt="image" src="https://github.com/user-attachments/assets/e478f055-d250-4615-84ab-027436f72f95" />

---

# Un game engine

<img width="94" height="47" alt="image" src="https://github.com/user-attachments/assets/0071dd5e-2b21-4921-b940-10cd4f70f511" />

Tous les game engines du monde sont juste de grosses boîtes à outils qui dessinent des triangles,
et utilisent des sphères pour calculer des collisions

Alors essayez de dessiner votre premier triangle (en noir et blanc ici)

---

# Une lettre

<img width="9" height="12" alt="image" src="https://github.com/user-attachments/assets/bd6d0257-2cbf-4e44-af7d-29ddf8715d09" />

Essayez à la main d’écrire un B sur une taille de 8 × 6

Si vous y arrivez à la main, essayons avec du texte

```
var b :String ="""
000000
011100
010010
011100
010010
010010
011100
000000
"""
```

---

# Byte et pages

<img width="2" height="9" alt="image" src="https://github.com/user-attachments/assets/0f32a3bc-295b-410e-9392-7e3454d10253" />

En réalité, si vous regardez sous le capot d’un SSD1306, ce sont 8 tableaux de 8×128

Pour le délire…

Créons une méthode qui prend un byte et le dessine verticalement sur le XY donné

---

# Comptons en binaire 😉

<img width="128" height="44" alt="image" src="https://github.com/user-attachments/assets/74f8c8a0-a8b4-4023-9f94-f85917a8744b" />

Ah bah, si vous savez dessiner un byte :

Dessinez les bytes de 0 à 127 sur la page 1
Dessinez les bytes de 128 à 255 sur la page 2







