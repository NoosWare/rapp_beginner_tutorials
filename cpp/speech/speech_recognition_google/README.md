#Speech recognition using Google
----------------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

In this tutorial we are going to recognise the speech from an audio using Google, RAPP and CMake.

You have to configure your project in like this tree:

```
project/
        CMakeLists.txt
        source/
                speech_recognition_google.cpp
        build/
```

##Source code

We created a project called `speech_recognition_google` with a folder `source` where we have our
example call `speech_recognition_google.cpp`.
You can see the complete example [here](source/speech_recognition_google.cpp).

We are going to initialize the platform information and the service controller, which is in charge
of make the cloud calls to the RAPP platform:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```


