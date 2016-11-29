#Qr Recognition
---------------

* *This tutorial assumes that RAPP API and OpenCV are installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to do a project using OpenCV, RAPP and CMake.

Using CMake requires you to configure your CMakeLists.txt in the **root** of your RAPP project.
The structure is the following:

```
project/
        CMakeLists.txt
        source/
                qr_recognition.cpp
        build/
```

##Source code

We create a project called `qr_recognition` with a folder `source` where we have our
example `qr_recognition.cpp`.
You can see the complete example [here](source/qr_recognition.cpp).

In this example we are taking images from a camera and then
we call the RAPP platform and look for qr_codes.
We use a usb camera and OpenCV for image acquisition:

```cpp
cv::VideoCapture camera(0); 
if(!camera.isOpened()) { 
    std::cout << "Failed to connect to the camera" << std::endl;
    return -1;
}
```

We created the parameter `camera` which is our `dev0`. You will have to check if your camera is
in the correct device and change the number if it's necessary. In the case that the program
don't recognise any devices, the program will close.

In other examples, we only see the result with a stdout. Now we would like to see the position
of the qr_codes in the image that our camera is recording. So, we are going to create an OpenCV window:

```cpp
cv::namedWindow("QR recognition", cv::WINDOW_AUTOSIZE);
```

The size of the window depends of your camera resolution.
At the same time, we can initialize our OpenCV matrix where we are going to save the image data
from the camera:

```cpp
cv::Mat frame;
```

At this point, we can continue our program like RAPP API examples.
We are going to initialize the platform information and the service controller, which is in charge
of make the cloud calls to the RAPP platform:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

One the parts that is needed to make the call is a callback:

```cpp
auto callback = [&](std::vector<rapp::object::qr_code> qr_codes) { 
    std::cout << "Found: " << qr_codes.size() << " qr_codes" << std::endl; 
    for(auto each_qr_code : qr_codes) {
        cv::rectangle(frame,
                      cv::Point(each_qr_code.get_left_x(), each_qr_code.get_left_y()),
                      cv::Point(each_qr_code.get_right_x(), each_qr_code.get_right_x()),
                      cv::Scalar(255,0,0),
                      1, 8, 0);
    }
};
```

This callback shows how many qr_codes have been found and, if there are any, and
it draws a blue rectangle around the qr code.
In this example, we repeat the call every 500 ms.

```cpp
auto before = std::chrono::system_clock::now();
for (;;) {
    auto now = std::chrono::system_clock::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(now - before).count(); 
    ...
}
```

##CMakeLists.txt

We have updated `helloworld/CMakeLists.txt` to use opencv:

```
find_package(OpenCV REQUIRED)
target_link_libraries(qr_recognition ${RAPP_LIBRARIES}
                                     ${OpenCV_LIBS})
```

##Building your code

The next step is about create our `build` folder and run our code.

Now we are going to work in the terminal.

1. Go to your project path (in our case `qr_recognition/`)
2. Build your project
```
mkdir build
cd build 
cmake ..
make
```

3. If everything is ok, you will have created your executable `qr_recognition` in the folder build.
4. Run your executable
    ```
    ./qr_recognition
    ```
    
