#Face detection
---------------

* *This tutorial assumes that RAPP API and OpenCV are installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to do a project using OpenCV, RAPP and CMake.

Using CMake requires you to configure your CMakeLists.txt in the **root** of your RAPP project.
Essentially, the structure is the following:

```
project/
        CMakeLists.txt
        source/
                face_detection.cpp
        build/
```

##Source code

We created a project called `face_detection` with a folder `source` where we have our
example call `face_detection.cpp`.
You can see the complete example [here](source/face_detection.cpp).

In this example we take image data from a camera and we use the image
to call the RAPP platform and look for faces.
For acquisition of the image from our usb camera we use OpenCV:

```cpp
cv::VideoCapture camera(0); 
if(!camera.isOpened()) { 
    std::cout << "Failed to connect to the camera" << std::endl;
    return -1;
}
```

We created the parameter `camera` which is our `dev0`. You will have to check if your camera is
the correct device and change the number if necessary. In the case that the program
doesn't recognise any devices, it will exit.

In previous `face_detection` examples, we only print the result of the cloud to stdout. In this example we are goint to  draw rectangles around the faces in the image that our camera is recording:

```cpp
cv::namedWindow("Face detection", cv::WINDOW_AUTOSIZE);
```

The size of the window depends on the camera resolution.
We can initialize our OpenCV matrix where we are going to save the image data from the camera:

```cpp
cv::Mat frame;
```

We initialize the platform information and the service controller:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

We define an *inline* lambda which will capture by reference the variables used, and at the same time will iterate the discovered faces and draw a frame around them:

```cpp
auto callback = [&](std::vector<rapp::object::face> faces) { 
    std::cout << "Found: " << faces.size() << " faces" << std::endl; 
    for(auto each_face : faces) {
        cv::rectangle(frame,
                      cv::Point(each_face.get_left_x(), each_face.get_left_y()),
                      cv::Point(each_face.get_right_x(), each_face.get_right_x()),
                      cv::Scalar(255,0,0),
                      1, 8, 0);
    }
};
```

__NOTE:__ keep in mind that we can't use a simple loop to make the calls. 
There is a **pause** between the calls so that our client doesn't hog up the CPU. 
Also, the platform may **block** us if we spam non-stop calls.

Because of that we wait for a second (see `rapp-api/cpp/examples/loop.cpp`) before doing a call. 
In the `loop` example we use a `std::chrono` object which counts the time between loops.
In this example, we are going to make a call every 500 ms.

```cpp
auto before = std::chrono::system_clock::now();
for (;;) {
    auto now = std::chrono::system_clock::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(now - before).count(); 
    ...
}
```

OpenCV has the function [`cv::imencode`](http://docs.opencv.org/2.4/modules/highgui/doc/reading_and_writing_images_and_video.html).
which saves an image matrix to an image file.
Thus, whilst we acquire an *image matrix* from OpenCV, we end up transmitting a **PNG** to the platform.

```cpp
if (elapsed > 500) {
    camera >> frame;
    std::vector<int> param = {{ CV_IMWRITE_PNG_COMPRESSION, 3 }};
    cv::vector<uchar> buf;
    cv::imencode(".png", frame, buf, param);
    std::vector<rapp::types::byte> bytes(buf.begin(), buf.end());
    auto pic = rapp::object::picture(bytes);
    before = now;
    ctrl.make_call<rapp::cloud::face_detection>(pic, true, callback);
    cv::imshow("Face detection", frame);
}
```

After calling the cloud function, we use [`cv::imshow`](http://docs.opencv.org/2.4/modules/highgui/doc/user_interface.html) to draw the frame on the GUI.
The interface is not going to refresh the image until we use `cv::waitKey` function.

##CMakeLists.txt

We assume that you have built your RAPP API in the **static** and **shared** libraries mode.
This file is going to be the same that we have in `helloworld/CMakeLists.txt`.
We only have to add the OpenCV library and change the names of the project and executable.

*NOTE:* If you want to use only the **static** libraries, you can see `helloworld_static` project.
The only lines that we have to add are:

```
find_package(OpenCV REQUIRED)
target_link_libraries(face_detection ${RAPP_LIBRARIES}
                                     ${OpenCV_LIBS})

```

##Building your code

The next step is to create our `build` folder and build the prorgam:

1. Go to your project path (in our case `face_detection/`)
2. Build your project
```bash
mkdir build
cd build 
cmake ..
make
```

3. If everything is ok, you will have an executable `face_detection` in the folder build.
4. Run your executable
    ```
    ./face_detection
    ```

