#Human detection
----------------

* *This tutorial assumes that RAPP API and OpenCV are installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to create a project using OpenCV, RAPP API and CMake.

Using CMake requires you to configure your CMakeLists.txt in the **root** of your RAPP project.
The structure is the following:

```
project/
        CMakeLists.txt
        source/
                human_detection.cpp
        build/
```

##Source code

We created a project called `human_detection` with a folder `source` where we have our
example call `human_detection.cpp`.
You can see the complete example [here](source/human_detection.cpp).

In this example we take images from a camera and we use this image to call the RAPP platform and search for humans.

Taking a image from our usb camera uses OpenCV:

```cpp
cv::VideoCapture camera(0); 
if(!camera.isOpened()) { 
    std::cout << "Failed to connect to the camera" << std::endl;
    return -1;
}
```

We created the parameter `camera` which is our `dev0`. You will have to check if your camera is
the same device and adjust if necessary. In the case that the program doesn't find any devices, it will exit.

If you want to change the resolution of the image from your camera, change 
the parameters of the camera. 

__NOTE:__ The bigger the image is, the more time is needed to process it.

```cpp
camera.set(CV_CAP_PROP_FRAME_WIDTH,640);
camera.set(CV_CAP_PROP_FRAME_HEIGHT,480);
```

We create an OpenCV window, and then we'll draw rectangles around the humans detected:

```cpp
cv::namedWindow("Human detection", cv::WINDOW_AUTOSIZE);
```

The size of the window depends of your camera resolution.
At the same time, we initialize our OpenCV matrix where we are going to save the image data
from the camera:

```cpp
cv::Mat frame;
```

Same as before, we setup the platform ifno and cloud controller:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

We set the lambda callback, which will iterate over the humans and drawn around them on screen:

```cpp
auto callback = [&](std::vector<rapp::object::human> humans) { 
    std::cout << "Found: " << humans.size() << " humans" << std::endl; 
    for(auto each_human : humans) {
        cv::rectangle(frame,
                      cv::Point(each_human.get_left_x(), each_human.get_left_y()),
                      cv::Point(each_human.get_right_x(), each_human.get_right_x()),
                      cv::Scalar(255,0,0),
                      1, 8, 0);
    }
};
```

This callback shows how many humans have been found and, if there are any
it draws a blue rectangle:

```cpp
if (elapsed > 500) {
    camera >> frame;
    std::vector<int> param = {{ CV_IMWRITE_PNG_COMPRESSION, 3 }};
    cv::vector<uchar> buf;
    cv::imencode(".png", frame, buf, param);
    std::vector<rapp::types::byte> bytes(buf.begin(), buf.end());
    auto pic = rapp::object::picture(bytes);
    before = now;
    ctrl.make_call<rapp::cloud::human_detection>(pic, callback);
    cv::imshow("Human detection", frame);
}
```

##CMakeLists.txt

Add the following lines which use the OpenCV package in cmake:

```
find_package(OpenCV REQUIRED)
target_link_libraries(human_detection ${RAPP_LIBRARIES}
                                      ${OpenCV_LIBS})

```

And modify the names of the executable and the project:

```
project(human_detection)
add_executable(human_detection source/human_detection)
```

##Building

1. Go to your project path (in our case `human_detection/`)
2. Build your project:
```
mkdir build
cd build 
cmake ..
make
```

3. You should now have `human_detection` in the folder build.
4. Run your executable:
    ```
    ./human_detection
    ```
