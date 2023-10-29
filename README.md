# gdscript-srlz
A simple gdscript data serilazer/deserializer that works with custom objects.

## Note
- Classes must be declared with the "class_name" keyword and placed in their own script file.
```
##### player.gd
class_name Player # this will work

var health: int
var attack: int

class Player2: # this won't work
  var health: int
  var attack: int
```

- Objects must be able to be created without passing in arguments.
```
func _init(): # either by using constructor with no parameters
  health = 10

#####

func _init(_health = 10): # or by using default arguments
  health = _health
```

- Dictionary keys should be primitive, typically in int or string. If for some reason you need to use object as key, ~~stop cooking~~ it can be done by modifying the source code.

## Usage
```
# serialization
var data = SRLZ.serialize(obj)

##### after reading saved data from file or received from networking

# deserialization
var obj2 = SRLZ.deserialize(data)
```

## Supported types

### Primitive types
Primitive types supported by Godot's Binary Serialization API will also work.
```
data = SRLZ.serialize(2)
data = SRLZ.serialize(2.0)
data = SRLZ.serialize("2")
data = SRLZ.serialize(Vector2(1,2))
data = SRLZ.serialize(Color.WHITE)
data = SRLZ.serialize(AABB())
data = SRLZ.serialize(var_to_bytes("2"))
# ...
```
Deserializing returns the serialized data in the correct type. Use type hint if needed.
```
var obj = SRLZ.deserialize(data)
```

### Objects
Custom objects, nested objects, inheritance, object containers are supported.

Example classes:
```
##### entity.gd
class_name Entity
var uuid: String

##### weapon.gd
class_name Weapon
var attack: int

func _init(_attack = 10):
  attack = _attack

##### player.gd
class_name Player extends Entity
var health: int
var weapon: Weapon = Weapon.new(10)

func _init(_health = 100, _weapon = null):
  health = _health
  weapon = _weapon

##### room.gd
class_name Room
var player_dict: Dictionary

func _init(dict = {}):
  player_dict = dict
```
Usage:
```
# serialization
var player = Player.new(100, Weapon.new(10))
player.uuid = "12345"
var room = Room.new({"12345": player})
var weapon_list = [Weapon.new(10), Weapon.new(15)]
var player_data = SRLZ.serialize(player)
var room_data = SRLZ.serialize(room)
var weapon_list_data = SRLZ.serialize(weapon_list)

##### after reading saved data from file or received from networking

# deserialization, type hint highly recommended
var _player: Player = SRLZ.deserialize(player_data)
var _room: Room = SRLZ.deserialize(room_data)
var _weapon_list: Array = SRLZ.deserialize(weapon_list_data)

print("%s %s %s" % [_player.uuid, _player.health, _player.weapon.attack]) # 12345 100 10
print("%s %s %s" % [_room.player_dict["12345"].uuid, _room.player_dict["12345"].health, _room.player_dict["12345"].weapon.attack]) # 12345 100 10
print("%s %s" % [_weapon_list[0].attack, _weapon_list[1].attack]) # 10 15
```

