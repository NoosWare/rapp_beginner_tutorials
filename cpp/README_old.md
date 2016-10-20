# Getting Started

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

## Service Controller

In the following tutorial we are going to show you how to get up and running and how to make cloud calls.
RAPP is divided in two main parts:

* a platform which is either on a cloud server, or on a computer you've setup.
* an API which is a library and  headers that you use to communicate with the platform.

The `service_controller` class is the controller used to run cloud calls. 
When you instantiate an object, you have to specify the **address** and **port** of the platform.
You can find an example [here](#example service controller and cloud call)

## Cloud call

RAPP Cloud is based on the *PaaS* (Platform as a Service) principle, 
and the core notion is that you make requests to the platform and you will receive replies.
It is **important** to note that the calls are asynchronous, and therefore the reply to your query
is non-deterministic (it is not possible to guarantee time of execution). 
Different queries will require different processing time,
and as such you can run other code on the robot whilst a query is running on the cloud.

## Example service controller and cloud calls

In the following example we query all the services that are available on the _rapp-platform_. 

*Note:* for all examples we assume you've included the proper headers. 
In this case we need:

* `rapp/cloud/service_controller.hpp`
* `rapp/cloud/available_services.hpp`
* `iostream`

Construct the `service_controller` object using a `rapp::cloud::platform` struct, which contains:

* platform address, hostname or IP (e.g., `rapp.ee.auth.gr` or `155.207.19.229`). 
* if running your own platform, use `localhost`
* the platform port (default is 9001)
* your authentication token (e.g., `rapp_token`)

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
```

The next step is to create a service controller (we call it `ctrl` in these examples),
which is in charge of creating and controlling cloud calls to the platform.
The only parameter needed is the information about the platform, which is saved in the `info` variable:

```cpp
rapp::cloud::service_controller ctrl(info);
```

## Callback functors

When making cloud calls, and due to their asyncrhonous nature, we pass callback parameters.
This approach is common throught the entire API; almost all cloud calls expect you to pass a callback functor.
For more information on callbacks, see: 
* http://en.cppreference.com/w/cpp/language/lambda
* http://en.cppreference.com/w/cpp/utility/functional/function

Your callback can also be a class method, which is bound (using `std::bind` or `boost::bind`)
however we suggest you use lambdas or `std::function` when starting.

## Examples of Cloud calls

Each subfolder in this repository has an example, those are the same examples taken from
the API example folder (`rapp-api/cpp/examples`) however they require that you have
built and installed the rapp library and headers.

You can build each one with cmake, by doing:

```
mkdir build
cd build
cmake ..
make
```

For each example you'll end up with an executable in your `build` directory.
If you're having trouble building the examples, feel free to chat with us on gitter.

## A simple cloud call:

Lets make a call to available services class.
The construct parameter is a callback functor, e.g., a *lambda* or `std::function`.
It will receive an `std::vector<std::pair<std::string, std::string>>` which is
a vector of pairs f strings, the first string in the pair naming the service, and the second one defining the URL
of the service.

For more details, see: `rapp/cloud/available_services.hpp`

In this example the lambda function will print the available services on screen.

```cpp
auto cb = [](std::vector<std::pair<std::string, std::string>> services) {
    std::cout << "available services: " << std::endl;
    for (const auto & pair : services) {
        std::cout << pair.first << " " << pair.second << std::endl;
    }
};
```

Finally, run the actual call:

```cpp
ctrl.make_call<rapp::cloud::available_services>(cb);
```

*Note:* This call is going to block the service controller queue until it completes.
You can run multiple calls using multiple service controllers, or a batch of calls (shown below)

## Example cloud batches

In the following example we are going to execute three actions at the same time:

* face detection
* human detection
* door angle detection

Include the following headers:

* service_controller.hpp
* picture.hpp
* vision_detection.hpp
* iostream

Then create a service controller as before:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

In this example we are going to use vision detection, so we need to load a picture.
In this case we are going to detect faces, humans and doors in the same picture (for simplicty's shake).
You can use the picture that you want using the correct path or use this example.
If loading a picture from disk, then constuct a `rapp::object::picture` object (replace *my_picture.jpg* with a valid one!).

```cpp
auto pic = rapp::object::picture("my_picture.jpg");
```

Each cloud call takes a different callback functor, so
we'll create one for each call.
(You can find more information in `rapp/cloud/vision_detection.hpp`).

* face detection:

```cpp
auto face_cb = [&](std::vector<rapp::object::face> faces) { 
    std::cout << "Found " << faces.size() << " faces!" << std::endl;
};
```

* human detection:

```cpp
auto human_cb = [&](std::vector<rapp::object::human> humans) {
    std::cout << "Found " << humans.size() << " humans!" << std::endl;
};
```

* door angle:

```cpp
auto hazard_cb = [&](double door_angle) {
    std::cout << "Door angle: " << door_angle  << std::endl;
};
```

When we construct the batch, we construct the cloud call in a one-off statement
as we execute `service_controller::make_calls`.
For a complete parameter list of each class, please see the respective class documentation.

```cpp
ctrl.make_calls(rapp::cloud::face_detection(pic, false, face_cb),
                rapp::cloud::human_detection(pic, human_cb),
                rapp::cloud::door_angle_detection(pic, hazard_cb));
```

When the platform responds, we'll receive:

```
Door angle: 2
Found 2 humans!
Found 1 faces!
```

The order of the calls may change: there is a probability that face detection may finish first
and then human detection.

## Cloud Loop

In this part of the tutorial we'll show you how to do a simple loop with the API.
Because we don't want to spam the CPU with a constant `for(;;)` loop, 
we'll use boost library's `async_wait` which will run at a specific interval.

Include the headers:

* `rapp/cloud/service_controller.hpp`
* `rapp/cloud/vision_detection.hpp`
* `rapp/objects/picture.hpp`
* `iostream`
* `functional`

Also (assuming you have `boost` installed) include:

* `boost/asio.hpp`

We initialize the service controller, the timer and a io_service needed by the timer:

```cpp
boost::asio::io_service io_service;
boost::asio::deadline_timer timer(io_service, boost::posix_time::seconds(1));
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

Create a functor which will make the cloud calls.
This call will be repeatedly invoked on a one second interval.
That one second **does not include** and **does not take into account** the processing time
of the platform or the information roundtrip (the time it takes to connect, process and obtain the reply).
It simply states that the callback will run every one second, **however** the response time may take less or more than one second.

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

Run the timer and service:

```cpp
timer.async_wait(func);
io_service.run();
```

At this point we will obtain a list of faces found in the in a continous loop.
Hit `CTRL+c` when you want to stop the application.

## Class reactor

Running cloud calls using lambdas may not always be easy; under certain conditions
a reactor class which handles callbacks may be a better solution.
In this scenario we define and implement a class which does just that.
The `reactor` class methods will be used to handle cloud replies:

```cpp
class reactor
{
public:

    void handle_face(std::vector<rapp::object::face> faces)
    {        
        std::cout << "Found " << faces.size() << " faces!" << std::endl;
    }

    void reactor::handle_ontology_sub(std::vector<std::string> classes)
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

    ...
};
```

The reactor class also wraps around a `service_controller` and thus
controls the calls.

```cpp
class reactor
{
    ...

private:
    rapp::cloud::service_controller ctrl_;
};
```

We construct it using:

```cpp
reactor::reactor(rapp::cloud::platform info)
: ctrl_(info)
{}
```

Making the calls is done using the `run` method.
We also use it to bind to the handler methods.

```cpp
void reactor::run(rapp::object::picture pic, std::string object, std::string city)
{
    ctrl_.make_calls(face_detection(pic, false, std::bind(&reactor::handle_face, this, std::placeholders::_1)),
                     weather_report_current(city, "", 0, std::bind(&reactor::handle_weather, this, std::placeholders::_1)),
                     ontology_subclasses_of(object, true, std::bind(&reactor::handle_ontology_sub, this, std::placeholders::_1)));
}
```

Thus this method will run three different cloud calls as a batch job, whilst
re-using its own methods to receive the replies.
Because we're using class methods we have to either use `std::function` class members,
or use a `bind` (either `std::bind` or `boost::bind`) to the class methods.

Constructing a unique pointer to our reactor class is done like so:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
auto my_reactor = std::make_unique<reactor>(info);
```

And finally we execute the `run` method:

```cpp
my_reactor->run(pict, "Toy", "Madrid");
```

The reply you'll receive is:

```
Sub classes:
http://knowrob.org/kb/knowrob.owl#Toy
Temperature: 79.36
Humidity: 0.26
Wind speed: 2.5
Found 1 faces!
```

# Sub-Folders

Have a look at each subfolder in this repository.

| Examples | Link |
|----------|------|
| Helloworld | [Helloworld](helloworld/) |
| Helloworld static | [Helloworld static](helloworld_static)|
| | |
| Available services | [Available services](available_services/) |
| Batch | [Batch](batch/) |
| Class reactor | [Class reactor](class_reactor/) |
| Cognitive games | [Cognitive games](cognitive_games/) |
| Config | [Config](config/) |
| Email handling | [Email handling](email_handling/) |
| Face detection | [Face detection](face_detect/) |
| Geolocation | [Geolocation](geolocation/) |
| Hazard detection | [Hazard detection](hazard_detect/) |
| Human detection | [Human detection](human_detect/) |
| Knowrob ontology | [Knowrob ontology](knowrob_ontology/) |
| Light detection | [Light detection](light_detection/) |
| Loop | [Loop](loop/) |
| News | [News](news/) |
| Object recognition| [Object recognition](object_recognition/) |
| Path planning | [Path planning](path_planning/) |
| Picture handling | [Picture handling](picture_handling/) |
| Qr detection | [Qr detection](qr_detect/) |
| Set noise profile | [Set noise profile](set_noise_profile/) |
| Speech recognition with Google | [Speech recognition with Google](speech_recognition_google/) |
| Speech recognition with Sphinx4 | [Speech recognition with Sphinx4](speech_recognition_sphinx4/) |
| Speech to text | [Speech to text](speech_to_text/) |
| Weather | [Weather](weather/) |

We go through each cloud class, as well as the design patterns explained above.
If you need any help, you can chat with us at [![Join the chat at https://gitter.im/rapp-project/rapp-api](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/rapp-project/rapp-api?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) 
