Salut a tous.

Avant de passer a du Godot Engine et de la XR,  
Je vous propose de revoir et repratiquer les bases de la programmation:
reserved keyword, variable, boucle, methode, tableau 1D 2D,...

Si vous avez deja le niveau, je vous invite faire votre propre SSD1306.
[Faire un quad Texturable](#)

Mais pour les autres, utilisons ce que je vous ai preparez 😉:
https://github.com/EloiStree/2026_04_27_gdp_oled_128x64

La solution a vos exercice sont le package en question:
Je vous demande donc de ne pas regarder 😜
(_Sauf si vous etes vraiment blocker_)

Pour les exercices, vous ne pouvez utiliser que;


SSD1306NodeFacade.gd
```gdscript
# Allows to set the next draw value to `true` or `false`
func set_value_at_index_1d(index_0_8191:int, is_on:bool):
func get_value_at_index_1d(index_0_8191:int)->bool:

# Request to create the Texture2D to be displayed
func draw():

# Allows to configure the full array in one call
func set_value_with_1d_array(array_1d:Array[bool]):
func get_value_as_1d_array_reference(array_1d:Array[bool]):
func get_value_as_1d_array_copy(array_1d:Array[bool]):


# Use those four only if you created it youself before from 1D exercice.
func set_value_at_x_y_lrtd(x_left_right:int,y_top_down:int, is_on:bool):
func get_value_at_x_y_lrtd(x_left_right:int,y_top_down:int)->bool:
func set_value_at_x_y_lrdt(x_left_right:int,y_top_down:int, is_on:bool):
func get_value_at_x_y_lrdt(x_left_right:int,y_top_down:int)->bool:
```


Regardons a nos feuilles A4 et essayons de barrer les concepts:   
https://miro.com/app/board/uXjVGUVLhc8=/   
