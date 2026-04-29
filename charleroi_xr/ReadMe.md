
Salut à tous,
   
Avant de passer à Godot Engine et à la XR,   
je vous propose de revoir et de repratiquer les bases de la programmation:   
mots-clés réservés, variables, boucles, méthodes, tableaux 1D et 2D, etc.    
  
Si vous avez déjà le niveau, je vous invite à faire votre propre SSD1306 :   
[Créer un quad texturable](https://github.com/EloiStree/HelloGodot128x64/tree/main/charleroi_xr/workshop/create_128x64_textured_quad)   

Mais pour les autres, utilisons ce que je vous ai préparé 😉 :   
[https://github.com/EloiStree/2026_04_27_gdp_oled_128x64](https://github.com/EloiStree/2026_04_27_gdp_oled_128x64)    

Les solutions aux exercices se trouvent dans le package en question.    
Je vous demande donc de ne pas les regarder 😜    
(sauf si vous êtes vraiment bloqués)   

Pour les exercices, n'utilisez que:

`SSD1306NodeFacade.gd`
```gdscript
# Allows to set the next draw value to `true` or `false`
func set_value_at_index_1d(index_0_8191:int, is_on:bool):
func get_value_at_index_1d(index_0_8191:int)->bool:

# Request to create the Texture2D to be displayed
func draw():

# Allows to configure the full array in one call
func set_value_with_1d_array(array_1d:Array[bool]):
func get_value_as_1d_array_reference()->Array[bool]:
func get_value_as_1d_array_copy()->Array[bool]:


# Use those four only if you created it youself before from 1D exercice.
func set_value_at_x_y_lrtd(x_left_right:int,y_top_down:int, is_on:bool):
func get_value_at_x_y_lrtd(x_left_right:int,y_top_down:int)->bool:
func set_value_at_x_y_lrdt(x_left_right:int,y_top_down:int, is_on:bool):
func get_value_at_x_y_lrdt(x_left_right:int,y_top_down:int)->bool:
```

---------

# Installation / Setup

Créons un projet vide avec l’outil (l’addon) dans votre projet :

* [ ] Par fichier ZIP
* [ ] Par git clone
* [ ] Par git submodule
* [ ] Via une Godot Lib
* [ ] Via le Godot Asset Store (Beta)


---------

# Que peut-on etudier ?

Regardons nos feuilles A4 et essayons de cocher ou barrer les concepts :  
[https://miro.com/app/board/uXjVGUVLhc8=/](https://miro.com/app/board/uXjVGUVLhc8=/)  




