#Getting Started

**_This tutorial assumes that RAPP API is installed and built_**

##Service Controller

In the following tutorial we are going to show the basic knowledge that you need to start using RAPP Cloud.
RAPP is divided in two main parts:

1.RAPP platform which is in charge of do the actions that you need.
2.RAPP API which is in charge of communicate with the first one and make your life easier.
With that in mind, we are going to work directly with the API for programming robots.

**Service controller** is the part of the API which is in charge of telling the platform what we want to do. 
This is done with cloud calls. They can be done one by one, or many at the same time (we explain that later).
It's going to be the main part of our code to achive information or make actions with our robot.

You can find an example [here](#example service controller and cloud call);

##Cloud call

RAPP Cloud is based on *cloud communications*, so we have to make cloud call to communicate between the two parts of RAPP.
The main action of service controller is to make them. 
That is to communicate with the platform asking data or actions to do with our robot.
It's important to difference between a single cloud call and many of them. 
The command to use is not the same and the behaviour as well. 
In this section, we explain the uses of a single one.

We can see this better with an example:

###Example service controller and cloud calls

In the following example we are going to show all the services that are available in the _rapp-platform_. 
So we can know what actions we can do with our robot.

First of all, include the proper headers to your example. That is to associate only the files of the library which we need. 
In this case we only need three:

1. service_controller.hpp
2. available_services.hpp
3. iostream

Now we can start to write the main program. Always that we want to use the _rapp-platform_, we need to take its parameters which are:

1. Its hostname which can be `rapp.ee.auth.gr` or `155.207.19.229`. Besides you can use `localhost`if you are running the platform in your own computer.
2. Its port
3. Authentication token

We are going to save these parameters in a `rapp::cloud::platform` variable call `info`.

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
```

The next step is to create a service controller (called `ctrl`), which is in charge of create cloud calls to the platform with the tasks that we need.
The only parameter needed is the information about the platform, which is saved in `info` variable:

```cpp
rapp::cloud::service_controller ctrl(info);
```

We want to make a call to available services class, so first, we need to know what parameters are needed.
In this case, we need a function with a vector of pair of string where we are going to save all the services. 
*You can find all the information in rapp/cloud/available_services.hpp*.
In this example a lambda function is done to print all the services.

```cpp
auto cb = [](std::vector<std::pair<std::string, std::string>> services) {
         std::cout << "available services: " << std::endl;
         for (const auto & pair : services) {
            std::cout << pair.first << " " << pair.second << std::endl;
         }
};```

Finally, we make the cloud call with the next line:

```cpp
ctrl.make_call<rapp::cloud::available_services>(cb);
```

Where `cb` is the function mentioned above and we have to specify the class that we want, in this case *rapp::cloud::available_services*.
In this case you can see that you only need a single call because you obtain all the information requerided in one step. 
Available services are always the same. 

*NOTE:* This call is going to block the platform until the action is done.

If everything is ok, a list of services have to be shown.

*You can find the example file in rapp_api/cpp/examples/available_services*

##Cloud batches

Like we said in [Service Controller](#service controller) many calls can be done at the same time. 
This is what we call *cloud batches*. 
On the contrary of a call cloud, we are not goint to block anything while is running. 
In this case, they are going to be executed at the same time, working in parallel.

We can see better in the next example:

###Example cloud batches

In the following example we are going to execute three actions at the same time:

1. Face detection
2. Human detection
3. Door angle detection

Like we did in the previous example, the first step is to include the proper headers.
In this case we need the nexts ones:

* service_controller.hpp
* picture.hpp
* vision_detection.hpp
* iostream

The second step is to create the service controller with the information of the platform.

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

In this example we are going to use vision detection, so we need to load at least a picture.
In this case we are going to detect everything in the same picture, but different
pictures can be used to each case.
You can use the picture that you want using the correct path or use this example.

```cpp
auto pic = rapp::object::picture("data/object_classes_picture_7.jpg");
```

Before making the calls, we have to know what parameters need every class.
For example, face_detection needs as parameters the next list:

* A picture 
* A boolean to determine is we want a fast search. In this case we are going to say no (`false`).
* A function where save a vector of faces.

*You can find more information in rapp/cloud/vision_detection.hpp*

So, we are going to create a lamba function which is going to see how many faces has found.

```cpp
auto face_cb = [&](std::vector<rapp::object::face> faces) { 
    std::cout << "Found " << faces.size() << " faces!" << std::endl;
};
```

In the same way we have to do the same with the other classes.
`door_angle_detection` and `human_detection` only need the picture and the function, so we only have to do the function for each one.

```cpp
auto human_cb = [&](std::vector<rapp::object::human> humans) {
    std::cout << "Found " << humans.size() << " humans!" << std::endl;
};

auto hazard_cb = [&](double door_angle) {
    std::cout << "Door angle: " << door_angle  << std::endl;
};
```

Finally, we can make the call batch using the services controller `ctrl`.

```cpp
ctrl.make_calls(rapp::cloud::face_detection(pic, false, face_cb),
                rapp::cloud::human_detection(pic, human_cb),
                rapp::cloud::door_angle_detection(pic, hazard_cb));
```

If everything is ok, we have to receive something like this:

```
Door angle: 2
Found 2 humans!
Found 1 faces!
```

We can appreciate that the order of the calls is not relevant.
They are running on parallel, so the first process to conclude will be the first shown and so on.

*You can find the complete example in rapp-api/cpp/examples/batch.cpp*

##Cloud Loop

In this part of the tutorial how to do a simple loop with the API.

The way to do one is using the function of boost library `async_wait`, because, like the name said, is an asynchronous period of time.
In this way, you can avoid to block the robot.

We are going to see that with an example:

###Example cloud loop

In the next example we are going to do an asynchronous loop, making calls to the platform every second.

First of all, like the previous examples, is to include the headers:

1. service_controller.hpp
2. vision_detection.hpp
3. picture.hpp
4. iostream

These ones are the same as before. However, now we are going to use two new ones which allow us to us timers and functions:

5. boost/asio.hpp
6. functional

Now, in our main program, we are going to initialize the service controller, the timer and a io_service needed to the timer:

```cpp
boost::asio::io_service io_service;
boost::asio::deadline_timer timer(io_service, boost::posix_time::seconds(1));
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

The most important part is to make a function with all the actions that we want.
In this example we are going to analize a picture and find if there are faces every second.
For doing that we have the next function:

```cpp
std::function<void(const boost::system::error_code&)> func = [&](const auto & err) {
	auto pic = rapp::object::picture("data/object_classes_picture_1.png");
	auto callback = [&](std::vector<rapp::object::face> faces) { 
		std::cout << "Found " << faces.size() << " faces!" << std::endl;
	};
	ctrl.make_call<rapp::cloud::face_detection>(pic, false, callback);
	timer.expires_at(timer.expires_at() + boost::posix_time::seconds(1));
	timer.async_wait(func);
};
```

In this function we have the code that we knew for the examples above (initialize a picture, do a callback for our face_detection class and make the call) and then, we have the timer. 
All it does is to reinicialize its value to a second later and them wait for it calling the own function again.

After that, we have to begin the process with the same function that we have above, and running the service for the timer:

```cpp
timer.async_wait(func);
io_service.run();
```

If everything is ok we have to obtain the faces of the picture every second.

*You can find the complete example in rapp-api/cpp/examples/loop.cpp*

##Class reactor

In this part we are going to see that we are not limited to simple functions o loops for doing that we want.
In this case, we'll use a class to implement `face_detection`, `ontolog_subclasses_of` and `weather_report_current`.

Fisrt of all, use the proper headers. 
Apart from that we do a class that need the information of the platform, so the constructor we'll be this:

```cpp
reactor(rapp::cloud::platform info)
: ctrl_(info)
{}
```

Being `ctrl_` the name of the rapp::cloud::service_controller member.

After that, the common thing that all the classes have is that they need callback.
This is why we are going to do a method instead the std::function that we did in other examples.

```cpp
void handle_face(std::vector<rapp::object::face> faces)
{        
    std::cout << "Found " << faces.size() << " faces!" << std::endl;
}

void handle_ontology_sub(std::vector<std::string> classes)
{        
    std::cout << "Sub classes: " << std::endl;
    for (const auto & str : classes) {
        std::cout << str << std::endl;
    }    
}

void handle_weather(std::vector<std::string> weather) 
{
    std::cout << "Temperature: " << weather.at(1) << std::endl;
    std::cout << "Humidity: " << weather.at(3) << std::endl;
    std::cout << "Wind speed: " << weather.at(6) << std::endl;
}
```

The next step is to make the calls. 
For doing that we have created `run` method, which needs as parameters the inputs of the classes said. 
For more information see:

* rapp/cloud/vision_detection.hpp
* rapp/cloud/weather_report.hpp
* rapp/cloud/ontology.hpp

```cpp
void run(rapp::object::picture pic, std::string object, std::string city)
{
    ctrl_.make_calls(face_detection(pic, false, std::bind(&reactor::handle_face, this, std::placeholders::_1)),
                     weather_report_current(city, "", 0, std::bind(&reactor::handle_weather, this, std::placeholders::_1)),
                     ontology_subclasses_of(object, true, std::bind(&reactor::handle_ontology_sub, this, std::placeholders::_1)));
}
```

For using the methods above like in the examples before we have to use `std::bind()`.
Then we have finished our new class, now we are going to implement it.

Inside the main program we create `rapp::cloud::platform` with the information of the platform.
After that, we can use a `reactor` object.
The exception here is that we have to create it as a pointer because the member `ctrl_` of the class `reactor` requires it.

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
auto my_reactor = std::make_unique<reactor>(info);
```

Now we have to initialize the parameters that we need.
In this case we need only to load an image, the other parameters are `std::string` which we can write directly.
And finally, we use the `run` method.

```cpp
my_reactor->run(pict, "Toy", "Madrid");
```

If everything goes well, the answer'll be something similar to this:

```
Sub classes:
http://knowrob.org/kb/knowrob.owl#Toy
Temperature: 79.36
Humidity: 0.26
Wind speed: 2.5
Found 1 faces!
```

*You can find the complete example in rapp-api/cpp/examples/class_reactor.cpp*

