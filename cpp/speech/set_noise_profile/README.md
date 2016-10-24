#Set noise 
-----------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to do a project using RAPP and CMake.

You have to configure your project like this tree:

```
project/
        CMakeLists.txt
        source/
                set_noise_profile.cpp
        build/
```

##Source code

We are going to do a program to configure our noise profile.
So in the future, when we want to do speech recognition, we will be able to denoise the noise.

We created a project called `set_noise_profile` with a folder `source` where we have our
example call `set_noise_profile.cpp`.
You can see the complete example [here](source/set_noise_profile.cpp).

We are going to initialize the platform information and the service controller, which is in charge
of make the cloud calls to the RAPP platform:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

The only element we need is an audio file.
In this case, we have an example with a lot of noise. 
To be more efective you should change this audio for your own.

```cpp
rapp::types::audio_source audio_src = rapp::types::nao_ogg;
```

To make the call we need the audio and define what kind of audio is.
For defining you have to use `rapp::types::audio_source` enum. 
You can find more information in `rapp-api/cpp/rapp/cloud/globals.hpp`.

```cpp
rapp::types::audio_source audio_src = rapp::types::nao_ogg;
```

Now, we can make the call. 
We don't need a callback because we are going to send only the audio to the platform.
The platform will set the noise of this audio to analize the followings.

```cpp
ctrl.make_call<rapp::cloud::set_noise_profile>(audio.bytearray(), audio_src);
```

##CMakeLists.txt

In this case it assumes that you have built your RAPP API in the **static** and **shared** libraries mode.

This file is going to be the same that we have in `getting_started/cloud_call/CMakeLists.txt` file.
We only have to change the names of the project and executable.

##Build

The next step is about create our `build` folder and run our code.

Now we are going to work in the terminal.

1. Go to your project path
2. Build your project
```
mkdir build
cd build 
cmake ..
make
```

3. If everything is ok, you will have created your executable `set_noise_profile` in the folder build.
4. Run your executable
    ```
    ./set_noise_profile
    ```

Now you can explore and make your own projects!
