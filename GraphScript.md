**GraphScript Example**

- [Examples](#examples)
  - [Methods discussed](#methods-discussed)
  - [link function](#link function)
  - [JSON as first-class data type](#json-as-first-class-data-type)
  - [Serialization / Deserialization](#serialization--deserialization)
  
### Methods & Macros discussed  
  
   set\_config(); get\_config(); link(); and debug();
   
   REGISTER(), LINK\_LOGGER(), and LINK\_COMMON(),
   
   
### link() function 


The original function uses macros.

 
 ```c++
 void Capture::link(const Linker& linker) {
	const auto links = get_links();
	Json link_info(links);

	LINK_LOGGER("logger");
	LINK_COMMON(parent, "", FlowInterface, true); 
	LINK_COMMON(camera, "CaptureDevice", CaptureDeviceInterface, false);
}
 ```
 
Suggestions to improve: 

*  Name of macro and comments for each macro should be easy to understand
*  The variable name used in macro without underscore is difficult in programming. The original variable name with underscore should be used if possible. 


 ```c++
 void Capture::link(const Linker& linker) {
	const auto links = get_links();
	Json link_info(links);

	LINK_LOGGER("logger");
	LINK_COMMON(parent, "", FlowInterface, true); 
	LINK_COMMON(camera, "CaptureDevice", CaptureDeviceInterface, false);
}
 ```
 
 
 
### JSON as first-class data type

Here are some examples to give you an idea how to use the class.

Assume you want to create the JSON object

```json
{
  "pi": 3.141,
  "happy": true,
  "name": "Niels",
  "nothing": null,
  "answer": {
    "everything": 42
  },
  "list": [1, 0, 2],
  "object": {
    "currency": "USD",
    "value": 42.99
  }
}
```

With this library, you could write:

```cpp
// create an empty structure (null)
json j;

// add a number that is stored as double (note the implicit conversion of j to an object)
j["pi"] = 3.141;

// add a Boolean that is stored as bool
j["happy"] = true;

// add a string that is stored as std::string
j["name"] = "Niels";

// add another null object by passing nullptr
j["nothing"] = nullptr;

// add an object inside the object
j["answer"]["everything"] = 42;

// add an array that is stored as std::vector (using an initializer list)
j["list"] = { 1, 0, 2 };

// add another object (using an initializer list of pairs)
j["object"] = { {"currency", "USD"}, {"value", 42.99} };

// instead, you could also write (which looks very similar to the JSON above)
json j2 = {
  {"pi", 3.141},
  {"happy", true},
  {"name", "Niels"},
  {"nothing", nullptr},
  {"answer", {
    {"everything", 42}
  }},
  {"list", {1, 0, 2}},
  {"object", {
    {"currency", "USD"},
    {"value", 42.99}
  }}
};
```

Note that in all these cases, you never need to "tell" the compiler which JSON value type you want to use.