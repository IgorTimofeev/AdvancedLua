About
======
AdvancedLua is a library that forfills default Lua and OpenOS libraries with missing, but extremely necessary in everyday life functions: fast serialization of tables, detection of the current executable script, line wrapping, rounding of numbers, getting sorted file lists, various methods of processing binary data, etc.

| Contents |
| ----- |
| [Installation](#installation) |
| [Global functions](#global-functions) |
| [Table library additions](#table-library-additions) |
| [String library additions](#string-library-additions) |
| [Math library additions](#math-library-additions) |
| [Filesystem (OpenOS) library additions](#filesystem-openos-library0additions) |

Installation
======

Source code is available [here](https://github.com/IgorTimofeev/AdvancedLua/blob/master/lib/AdvancedLua.lua): You can download library to computer via single command:

    wget https://raw.githubusercontent.com/IgorTimofeev/OpenComputers/master/lib/advancedLua.lua /lib/advancedLua.lua -f

Global functions
======

**getCurrentScript**( ): *string* path
-----------------------------------------------------------

Get path to currently running script. For example, let's launch **/Test/Main.lua** with following data:

```lua
print("Path to current script: " .. getCurrentScript())
```

And the path to this script will be shown on screen:

```lua
Path to current script: /Test/Main.lua
```

Table library additions
======

table.**serialize**( t, [ pretty, indentationWidth, indentUsingTabs, recursionStackLimit ] ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | t | Table for serializing |
| [*boolean* | pretty] | Optional argument for simplier human-reading serializing. It has **false** value by default |
| [*int* | indentationWidth] | Optional argument that specifies indentation size in symbols when **pretty** mode is enabled |
| [*boolean* | indentUsingTabs] | Optional argument for using **tab** characters instead of whitespace when **pretty** mode is enabled |
| [*int* | recursionStackLimit] | Optional argument for limitation recursion stack depth |

The method was originally created as a fast alternative to OpenOS **/lib/serialization.lua**. It converts the contents of specified table to a string and it's is extremely convenient for saving configs for various software. Let's run following script for example:

```lua
local myTable = {
	"Hello",
	"world",
	abc = 123,
	def = "456",
	ghi = {
		jkl = true,
	}
}

print("Regular serialization: " .. table.serialize(myTable))
print(" ")
print("Pretty serialization: " .. table.serialize(myTable, true))
``` 

Result:

```lua
Regular serialization: {[1]="Hello",[2]="world",["abc"]=123,["def"]="456",["ghi"]={["jkl"]=true}}

Pretty serialization: {
	[1] = "Hello",
	[2] = "world",
	abc = 123,
	def = "456",
	ghi = {
		jkl = true,
	}
}
```

I draw your attention that the **pretty** argument performs several additional checks on the type of keys and table values, and also generates the line break after each value. Therefore, use it only if the readability of the result is in priority over performance.

table.**unserialize**( text ): *table or nil* result, *string* reason
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | text | Serialized string |

Method deserializes string representation of Lua table and returns result.
Метод пытается десериализовать строковое представление lua-таблицы и вернуть результат. If this is not possible, **nil** and the string with the reason of syntax error are returned. For example, let's perform the simplest deserialization:

```lua
local result = table.unserialize("{ abc = 123 }")
```

The result table will look like this:

```lua
{
	abc = 123
}
```

table.**toFile**( path, ... )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | The path to the file in which you want to write serialized result |
| - | ... | The set of parameters accepted by the function table.**serialize**(...) |

The method is similar to table .**serialize**(...), but instead of returning a string result, it writes it to a file. It is extremely convenient for quickly saving the config of the software without unnecessary stuff.

table.**fromFile**( path ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | The path to the file whose contents you want to deserialize |

The method is similar to table .**serialize**(...), but it reads string content directly from the existing file, returning the deserialized result. Again, for the most part it is used to conveniently download software configurations.

table.**size**( t ): *int* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | t | Table, the number of keys you want to calculate |

The method returns the number of keys of the passed table. It differs from the **#t** variant, i.e. it also counts non-numerical indexes

table.**contains**( t, object ): *boolean* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | t | Table in which the object will be searched |
| *var* | object | The object which presence in the table needs to be checked |

The method determines whether the object is present in the table and returns the result

table.**indexOf**( t, object ): *var* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | t | Table in which the object will be searched |
| *var* | object | The object whose index is to be determined |

The method returns the index (key) of the passed object. The index type can be different depending on the table structure: for example, in the table {abc = 123} the number 123 has a *string* index abc

table.**copy**( t ): *table* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | t | The table that needs to be duplicated |

The method recursively copies the contents of table **t** to a new one and returns the result. I draw your attention to the fact that tables referencing themselves are not supported (I was to lazy to make support for limiting the recursion stack depth by analogy with table .**serialize**(), sorry <3)

String library additions
======

string.**limit**( s, limit, mode, noDots ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | s | A string whose length should be limited |
| *int* | limit | Maximum string length |
| *string* | mode | The limitation mode for inserting an "..." symbol. It can take the values **left**, **center** or **right** |
| *boolean* | noDots | The mode of limitting by classical trimming without using the "..." symbol |

The method limits a string by inserting the "..." symbol in the correct location and returning the result. For example, let's run the code:

```lua
print("Left limit: " .. string.limit("HelloBeautifulWorld", 10, "left"))
print("Center limit: " .. string.limit("HelloBeautifulWorld", 10, "center"))
print("Right limit: " .. string.limit("HelloBeautifulWorld", 10, "right"))
```

As a result, the following will be displayed:

```lua
Left limit: …ifulWorld
Center limit: Hello…orld
Right limit: HelloBeau…
```

string.**wrap**( s, wrapSize ): *table* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string/string[]* | s | A string or an table of strings that need to be moved to the specified length |
| *int* | wrapSize | The wrapping length |

The method wraps a string with and returns the wrapped result as a table. If the size of a single word exceeds the specified length, the word will be "cut" into its constituent parts.

Also **\n** characters are supported to automatically move the caret to a new line. Let's run following code:

```lua
local limit = 24
local text = "Those days, the Third Age of Middle-earth, are now long past, and the shape of all lands has been changed; but the regions in which Hobbits then lived were doubtless the same as those in which they still linger: the North-West of the Old World, east of the Sea."
local lines = string.wrap(text, limit)

print(string.rep("-", limit))
for i = 1, #lines do
	print(lines[i])
end
```

As a result, the following will be displayed:

```lua
------------------------
Those days, the Third
Age of Middle-earth,
are now long past, and
the shape of all lands
has been changed; but
the regions in which
Hobbits then lived were
doubtless the same as
those in which they
still linger: the
North-West of the Old
World, east of the Sea.
```

string.**unicodeFind**( ... ): ...
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| - | ... | A set of arguments analogous to those for a string.**find**(...) function |

The method is similar to string .**find**(...), however it allows to work with unicode. Nice thing for a multilanguage-speaking people!

Math library additions
======

math.**round**( number ): *float* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *float* | number | Number to be rounded |

Method rounds the number to the nearest integer and returns the result

math.**roundToDecimalPlaces**( number, decimalPlaces ): *float* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *float* | number | Number to be rounded |
| *int* | decimalPlaces | The number of digits after the decimal point of the rounded number |

The method rounds the number to the nearest integer, limiting the result to the specified number of decimal places and returns the result

math.**shorten**( number, decimalPlaces ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number | The number to be visually shorten |
| *int* | decimalPlaces |  The number of digits after the decimal point of the rounded number |

The method converts the input number to a string with the prefixes "**K**", "**M**", "**B**" or "**T**" in dependence on the size of the number. For example, let's execute the code:

```lua
print("Shorten result: " .. math.shortenNumber(13484381, 2))
```

As a result, the following will be displayed:

```lua
Shorten result: 13.48M
```

Bit32 library additions
======

bit32.**merge**( number1, number2 ): *int* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number1 | First number for merging |
| *int* | number2 | Second number for merging |

The method "merges" two numbers and returns the result. For example, calling a method with arguments **0xAA** and **0xBB** will return a number **0xAABB**

bit32.**numberToByteArray**( number ) : *table* byteArray
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number | The number to be converted to a byte array |

The method extracts the bytes from the number and returns a table with them. For example, calling a method with the argument **0xAABBCC** will return the table * {0xAA, 0xBB, 0xCC}**

bit32.**numberToFixedSizeByteArray**( number, arraySize ) : *table* byteArray
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number | The number to be converted to a byte array |
| *int* | arraySize | Fixed size of the output array |

The method is similar to bit32.**numberToByteArray**(), but the size of the output array is specified manually. If the number of bytes in the number is less than the specified size, then the output array will be supplemented with missing zeros, otherwise the array will be filled with only a part of the bytes of the number. For example, calling a method with arguments **0xAABBCC** and **5** will return the table **{0x00, 0x00, 0xAA, 0xBB, 0xCC}**

bit32.**byteArrayToNumber**( byteArray ) : *int* number
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number | A byte array that needs to be converted to a number |

The method converts the bytes from the specified array to an integer. For example, calling a method with argument **{0xAA, 0xBB, 0xCC}** will return a number **0xAABBCC**

bit32.**byteArrayToNumber**( byteArray ) : *int* number
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | number | A byte array that needs to be converted to a number |

The method converts the bytes from the transmitted array to an integer. For example, calling a method with argument **{0xAA, 0xBB, 0xCC}** will return a number **0xAABBCC**


Filesystem (OpenOS) library additions
======

filesystem.**extension**( path ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | he path to the file, the extension of which must be obtained |

The method returns a string extension of the file along the specified path. For example, for the file **/Test/HelloWorld.lua** the string *.lua* will be returned

filesystem.**hideExtension**( path ): *string* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | The path to the file whose extension you want to hide |

The method hides the file extension on the specified path (if it exists) and returns a string result. For example, for the file **/Test/HelloWorld.lua** a string **/Test/HelloWorld** will be returned

filesystem.**isFileHidden**( path ): *boolean* result
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | The path to the file, the hidden state of which must be checked |

The method checks whether the file is hidden (i.e.whether its name starts with the dot symbol) and returns the result. For example, for the file **/Test/.Hello.lua** a **true** will be returned

filesystem.**sortedList**(path, sortingMethod, [ showHiddenFiles, filenameMatcher, filenameMatcherCaseSensitive ] ): *table* fileList
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | path | The path to the directory, the list of files that need to be obtained |
| *string* | sortingMethod | Sorting method can take values **name**, **type** and **date**|
| [*boolean* | showHiddenFiles] | Optional argument, which is responsible for including hidden files in the list. The default value is **false** |
| [*string* | filenameMatcher] | An optional argument that represents a regular expression, by which files from the list will be searched. For example, the expression "**%d+%.lua**" will only include files with the extension **.lua ** and having only digits in the name |
| [*string* | filenameMatcherCaseSensitive] | Optional argument, which allows to ignore the case of characters when using the argument **filenameMatcher** |

The method gets a list of files from the specified directory, sorting them by a certain method. The returned result is a classical numerically indexed table:

```lua
{
	"bin/",
	"lib/",
	"home/",
	"Main.lua"
}
```

filesystem.**readUnicodeChar**( fileHandle ): *string* char
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *handle* | fileHandle | Opened file descriptor in binary read mode (**rb**) |

The method reads a unicode character from the file descriptor, using binary operations. Since the "clean" Lua does not allow working with unicode characters when reading files in text mode, this method is extremely useful when writing your own file formats. You should be sure that the next byte-sequence from file handle **is guaranteed** corresponds to unicode character