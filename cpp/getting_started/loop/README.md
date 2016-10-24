# Cloud Loop
------------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We'll show you how to do a simple loop with the API.
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
