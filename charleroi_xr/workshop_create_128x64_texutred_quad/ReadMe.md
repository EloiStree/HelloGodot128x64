
# Atelier : créer un quad 128×64

> Écrivons un script qui prend un tableau de booléens en entrée et le transforme en texture 2D affichée sur un quad.

Si vous aimez le chaos organisé, vous pouvez tout mettre dans un seul script.  
Sinon, pour quelque chose d’un peu plus propre, on peut le découper en quatre scripts :  
* **La ressource** : contient les données du tableau et peut être partagée entre plusieurs scripts
* **Le convertisseur** : reçoit le tableau et génère une texture2D
* **Le peintre** : prend la texture2D et l’applique au quad pour l’affichage
* **Le contrôleur** : stocke la ressource, envoie les données au convertisseur, puis transmet la texture au peintre


----------------

----------


```gdscript
class_name Quad1306BoolArrayToTexture
extends Node

signal on_texture_material_updated(index:int, material_surface:Material)
signal on_texture_updated(texture: Texture2D)

@export var color_on: Color = Color(0.0, 0.734, 0.699, 1.0)   # OLED green
@export var color_off: Color = Color(0, 0, 0, 1)  # background

const SCREEN_WIDTH: int = 128
const SCREEN_HEIGHT: int = 64
const SCREEN_SIZE: int = SCREEN_WIDTH * SCREEN_HEIGHT
const SCREEN_SIZE_INDEX_MAX: int = SCREEN_SIZE - 1

var texture_2d: Texture2D


@export var use_mipmaps: bool = false
@export var material_to_duplicate: StandardMaterial3D
@export var material_duplicated: StandardMaterial3D

var bool_array_clear: Array[bool] = []
var bool_array_full: Array[bool] = []

@export var color_style: ColorStyle

enum ColorStyle {
	OLED_BLUE,
	OLED_GREEN,
	E_INK,
	BLACK_TRUE_ON_WHITE_FALSE,
	WHITE_TRUE_ON_BLACK_FALSE,
	GAMEBOY,
	INSEPCTOR_VALUE
}


func set_color_style_as_oled_blue_screen():
	# OLED blue: #007BFF
	color_on = Color("#00ccff")  # OLED blue
	color_off = Color("#000000")  # background
	set_texture_with_boolean_array(bool_array_clear)

func set_color_style_as_oled_green_screen():
	# OLED green: #00bf29
	color_on = Color("#00bf29")   # OLED green
	color_off = Color("#000000")  # background
	set_texture_with_boolean_array(bool_array_clear)


func set_color_style_as_black_true_on_white_false():
	# black true on white false
	color_on = Color("#000000")  # black
	color_off = Color("#FFFFFF")  # white
	set_texture_with_boolean_array(bool_array_clear)

func set_color_style_as_white_true_on_black_false():
	# white true on black false
	color_on = Color("#FFFFFF")  # white
	color_off = Color("#000000")  # black
	set_texture_with_boolean_array(bool_array_clear)

func set_color_style_as_e_ink_screen():
	# E-ink style: white for "on" pixels, light gray for "off" pixels
	# white
	color_off = Color("#FFFFFF")  # white	
	# BLACK DARK GRAY #5A5A5A
	color_on = Color("#5A5A5A")  # dark gray
	set_texture_with_boolean_array(bool_array_clear)  


func set_color_as_gameboy():
	# Screen tint (unlit LCD greenish): #9BBC0F
	# Screen dark green (active pixels): #0F380F
	color_on = Color("#0F380F")  # dark green
	color_off = Color("#9BBC0F")  # light green

	set_texture_with_boolean_array(bool_array_clear)  

func _ready():
	
	bool_array_clear.resize(SCREEN_SIZE)
	bool_array_full.resize(SCREEN_SIZE)
	for i in range(SCREEN_SIZE):
		bool_array_clear[i] = false
		bool_array_full[i] = true

	material_duplicated = material_to_duplicate.duplicate() as StandardMaterial3D
	var image = Image.create(SCREEN_WIDTH, SCREEN_HEIGHT, false, Image.FORMAT_RGB8)
	texture_2d = ImageTexture.create_from_image(image)
	material_duplicated.albedo_texture = texture_2d
	
	set_texture_with_boolean_array(bool_array_clear)
	if color_style == ColorStyle.E_INK:
		set_color_style_as_e_ink_screen()
	elif color_style == ColorStyle.GAMEBOY:
		set_color_as_gameboy()
	elif color_style == ColorStyle.OLED_GREEN:
		set_color_style_as_oled_green_screen()
	elif color_style == ColorStyle.OLED_BLUE:
		set_color_style_as_oled_blue_screen()
	elif color_style == ColorStyle.BLACK_TRUE_ON_WHITE_FALSE:
		set_color_style_as_black_true_on_white_false()
	elif color_style == ColorStyle.WHITE_TRUE_ON_BLACK_FALSE:
		set_color_style_as_white_true_on_black_false()


func set_boolean_array_to_full():
	for i in range(SCREEN_SIZE):
		bool_array_full[i] = true

func set_boolean_array_to_clear():
	for i in range(SCREEN_SIZE):
		bool_array_clear[i] = false


func get_on_off_color(is_on: bool) -> Color:
	return color_on if is_on else color_off



func set_texture_with_boolean_array(display_as_boolean_array: Array[bool]):
	var image = Image.create(SCREEN_WIDTH, SCREEN_HEIGHT, false, Image.FORMAT_RGB8)
	for i in range(SCREEN_SIZE):
		var pos := index_to_xy(i)
		var is_on: bool = display_as_boolean_array[i]
		var color = get_on_off_color(is_on)
		image.set_pixel(pos.x, pos.y, color)

	texture_2d = ImageTexture.create_from_image(image)
	on_texture_updated.emit(texture_2d)
	on_texture_material_updated.emit(0, material_duplicated)

func set_texture_with_bit_array(bit_pack_as_bytes:PackedByteArray):
	# expects width * height bits, packed as bytes (8 bits per byte)
	var total_bits = bit_pack_as_bytes.size() * 8
	var max_size = min(total_bits, SCREEN_SIZE)
	var result_array: Array[bool] = []
	
	for i in range(max_size):
		var byte_index = i / 8
		var bit_index = i % 8
		var is_on: bool = (bit_pack_as_bytes[byte_index] & (1 << bit_index)) != 0
		result_array.append(is_on)
	
	set_texture_with_boolean_array(result_array)
		
func index_to_xy(index: int) -> Vector2i:
	var x: int = index % SCREEN_WIDTH
	var y: int = index / SCREEN_WIDTH
	return Vector2i(x, y)

func xy_to_index(x: int, y: int) -> int:
	return y * SCREEN_WIDTH + x
```


``` gdscript
class_name Quad1306SetMeshInstanceTexture
extends Node

@export var mesh_instance: MeshInstance3D
@export var use_duplicate_at_ready: bool = true

@export_group("Debug")
@export var material_to_duplicated: StandardMaterial3D

func _ready() -> void:
	if not mesh_instance:
		return
	var material := mesh_instance.get_surface_override_material(0)
	
	if material:
		material_to_duplicated = material.duplicate() as StandardMaterial3D
		if use_duplicate_at_ready:
			mesh_instance.set_surface_override_material(0, material_to_duplicated)


func set_texture(texture: Texture2D) -> void:
	if not mesh_instance:
		return
	if not material_to_duplicated:
		return
	material_to_duplicated.albedo_texture = texture

```



