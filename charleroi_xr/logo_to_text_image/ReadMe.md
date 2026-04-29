

# Workshop: Une image a du binaire

> Essayons de transformer un image en un tableau binaire de 128x64 =1024 bytes et 8196 bits.


[<img width="750" height="500" alt="image" src="https://github.com/user-attachments/assets/d4dc1bda-449a-437a-bb2d-9b34bd3c90e4" />
](https://www.locationscout.net/belgium/27063-charleroi/60943)  
https://www.locationscout.net/belgium/27063-charleroi/60943   

Could be a logo:
<img width="998" height="640" alt="image" src="https://github.com/user-attachments/assets/cbae4217-6a72-492d-af5f-03c429c7cf56" />

Official Logo:
<img width="675" height="780" alt="5862a2e11aa68e0bf26fa1e2" src="https://github.com/user-attachments/assets/8effe111-2994-4340-bb4f-15b75eaf7678" />


--------------------------

--------------------------


``` gdscript

class_name SSD1306ParseImageToBoolean
extends Node
signal on_image_parsed(width:int, binary_image : Array[bool])
func image_to_parse_and_emit(image:Texture2D)->Array[bool]:
	var array:=convert_image_to_boolean_1d_array(image)
	var width = image.get_width()
	on_image_parsed.emit(width, array)
	return array

func image_to_parse(image:Texture2D)->Array[bool]:
	return convert_image_to_boolean_1d_array(image)

static func convert_image_to_boolean_1d_array(image:Texture2D)->Array[bool]:
	var result:Array[bool] =[]	
	var img:Image = image.get_image()
	#img.lock()
	for y in range(img.get_height()):
		for x in range(img.get_width()):
			var color:Color = img.get_pixel(x, y)
			var is_transparent:bool = color.a <=0.2
			var is_white:bool = color.r > 0.9 and color.g > 0.9 and color.b > 0.9
			var is_on:bool = true
			if is_transparent or is_white:
				is_on = false	
			result.append(is_on)
	#img.unlock()
	return result

```
