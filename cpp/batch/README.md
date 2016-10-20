# Example cloud batches

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


