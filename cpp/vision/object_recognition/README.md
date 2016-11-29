#Object recognition
-----------------

* *This tutorial assumes that RAPP API and OpenCV are installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to create a project using OpenCV, RAPP API and CMake.
Using CMake requires you to configure your CMakeLists.txt in the **root** of your RAPP project.
The structure is the following:

```
project/
        CMakeLists.txt
        source/
                object_recognition.cpp
        build/
```

##Source code
The source is inside the folder `source` where we have the example `object_recognition.cpp`.
You can see the complete example [here](source/object_recognition.cpp).

In this example we are taking images from a camera and we are using them
to call the RAPP platform and detect objects.
Object detection returns the highest clasification class of the object found in the image.

For taking a image from our usb camera we have to use OpenCV:

```cpp
cv::VideoCapture camera(0); 
if(!camera.isOpened()) { 
    std::cout << "Failed to connect to the camera" << std::endl;
    return -1;
}
```

We create the variable `camera` which is at `dev0`. You will have to check if your camera is
the same device and change it if needed.

If you want to change the resolution of the image from your camera, change the parameters of the camera. You have to remember that the bigger the image is, the more time is needed to process it.

```cpp
camera.set(CV_CAP_PROP_FRAME_WIDTH,640);
camera.set(CV_CAP_PROP_FRAME_HEIGHT,480);
```

The callback that receives the cloud reply

```cpp
auto callback = [&](std::string object) { 
        if (object.empty()) {
            std::cout << "No object found" << std::endl;
        }
        else {
            std::cout << "Found " << object << std::endl;
            cv::putText(frame, objects, cv::Point(50,50), cv::FONT_HERSHEY_PLAIN, 2, cv::Scalar(0, 0, 255), 2);
        }
    };
```

This callback shows what object has been found and, if there is any, 
it draws the description of the object in the window.

```cpp
if (elapsed > 500) {
    camera >> frame;
    std::vector<int> param = {{ CV_IMWRITE_PNG_COMPRESSION, 3 }};
    cv::vector<uchar> buf;
    cv::imencode(".png", frame, buf, param);
    std::vector<rapp::types::byte> bytes(buf.begin(), buf.end());
    auto pic = rapp::object::picture(bytes);
    before = now;
    ctrl.make_call<rapp::cloud::object_recognition>(pic, callback);
    cv::imshow("Object recognition", frame);
}
```

##Building the example

- Go to your project path (in our case `object_recognition/`)
- Build the example:
```
mkdir build
cd build 
cmake ..
make
```
- Run the executable
```
./object_recognition
```
