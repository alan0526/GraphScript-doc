GraphScript 
===========

- [Capture Task Example](#example)
  - [set_config function](#set-config-function)
  - [get_config function](#get-config-function)
  - [link function](#link-function)
  - [release function](#release-function)
  - [execute function](#execute-function)
  - [add_to_parent() function](#addtoparent-function)
  - [debug function](#debug-function)
  - [halt function](#halt-function)

## Capture Task Example

The Capture task contains a number of methods to be implemented. These functions are called by the system.


### REGESTER()

First of all, use REGISTER() macro to register the type to system, so the system can create the type from the factory.

In this case, "Capture" is the class of task defined in this example.

```c++
// register the type to system, so the system can create the type from the factory
REGISTER(Capture);
``` 
 
   
### set_config() function 

 

```c++
// the system calls this function to set member variables based on json configuration
void Capture::set_config(const Str& config) {
	set_behavior(config);

	auto json = Json::parse(config);

	// here is a example of deserailizing json to pimpl variables, and the types are cv::Mat and bool
	DESERIALIZE(*_pimpl, json, _filter, _is_apply_filter);

	// here is a example of deserailizing json to class member variables, the types are int32_t and string vector
	DESERIALIZE_SKIP(*this, json, _debug_images, _debug_image_names);
}
```
 
The system calls this function to get the json configuration from member variables.
 
### get\_config() function 

```c++
// the system calls this function to json configuration from member variables
Str Capture::get_config() const {
	auto json = Json();

	// here is a example of serailizing from pimpl variables to json objects, and the types are cv::Mat and bool
	SERIALIZE(*_pimpl, json, _filter, _is_apply_filter);

	// here is a example of serailizing from class member variables to json objects, the types are int32_t and string vector
	SERIALIZE(*this, json, _debug_images, _debug_image_names);

	const auto ret = json.dump();
	return ret;
}
```
 
 
 
### link() function 


The link function is called by system to get pointer to other objects that this object link to.

 
```c++
 // the system calls this function to get pointer to other objects that this object link to
void Capture::link(const Linker& linker) {
	const auto links = get_links();
	const Json link_info(links);

	const auto logger = link_object<LoggerInterface>(this, link_info, linker, "logger", true, "Logger");
	if(logger) {
		set_logger(logger->get_logger());
	}

	// Here is a example of a primary way to make a link
	_parent = link_object<FlowInterface>(this, link_info, linker, "parent");

	// Here is a example alternative way to make a link. 
	// You can get the type from the variable "_camera".
	_camera = link_object<std::decay<decltype(*(_camera.get()))>::type>(this, link_info, linker, "camera");

}
```
 
 
 The system calls this function set the member variables based on json configuration 
 
 


### release() function


```c++
// the system calls this function to release the shared pointers, so it can delete the object
void Capture::release() {
	_parent.reset();
	_camera.reset();

	_image.release();
}

```

System calls this function to release shared pointers
So it can delete the object

Suggestions to improve: 


### execute() function 

The planning to call this function 

```c++
// the planning calls this function to excute the task
void Capture::execute() {
	set_status(PlanStatus::kSuccess);

	const auto camera = _camera;
	if (!camera) {
		set_status(PlanStatus::kFailure);
		THROW_EXCEPTION(kErrorNullPtr, fmt::format("camera pointer can't be empty, "));
		return;
	}

	const auto width = camera->get_width();
	const auto height = camera->get_height();

	if(_image.cols != width || _image.rows != height) {
		_image = cv::Mat(height, width, CV_8UC3);
	}

	try {
		camera->play(PlayMode::stream);

		const auto frame_index = camera->get_frame(_image.data);
		if(frame_index < 0) {
			THROW_EXCEPTION(kErrorSimulation, "get frame failed, ");
		}

		if(_pimpl->_is_apply_filter && _image.total() > 0) {
			cv::filter2D(_image, _image, CV_8UC3, _pimpl->_filter);
		}

		camera->stop();
	}
	catch (...) {
		set_status(PlanStatus::kFailure);
	}

	Json planning_data;
	planning_data["width"] = width;
	planning_data["height"] = height;
	set_planning_data(shared_from_this(), planning_data.dump());
}
```


### add\_to\_parent() function 

```c++
/* This function is called by System after linking, to create child lists in parents.  
This function calls a function of the parent to add this node to the parents child list such as the following:
void add_child() {
	_parent.add_node(this);
}
This function is only overridden by nodes when the parent needs to add this child to its child list.
*/
void Capture::add_to_parent() {
	if (_parent) {
		_parent->add_child(get_omd_key(shared_from_this()), shared_from_this());
	}
}

```


### halt() function


```c++
// try to interrupt the job in the node
void Capture::halt() {
}

```

### debug() function


```c++
// the debugger calls this function to get debugging data to display to the user
Str Capture::debug() const {
	Json json = Json::parse(Node::debug());
	Json dependences;
	Json images;

	create_node_debug_info(dependences, _camera, "camera");

	for (auto i = 0; i < _debug_images; ++i) {
		if(i < static_cast<int32_t>(_debug_image_names.size())) {
			create_debug_image(images, this, fmt::format("image_{}", _debug_image_names[i]), _image);
		}
		else {
			create_debug_image(images, this, fmt::format("image_{}", i), _image);
		}
	}

	json["dependences"] = dependences;
	json["images"] = images;

	return json.dump();
}

```




 
Use this macro to register the type to the system so the system can create the type from the factory. 



 
