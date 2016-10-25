#Text to Speech
-----------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

We are going to do a project using RAPP and CMake.

You have to configure your project like this tree:

```
project/
        CMakeLists.txt
        source/
                text_to_speech.cpp
        build/
```

##Source code

We are going to create an audio file from text.

We created a project called `text_to_speech` with a folder `source` where we have our
example call `text_to_speech.cpp`.
You can see the complete example [here](source/text_to_speech.cpp).

We are going to initialize the platform information and the service controller, which is in charge
of make the cloud calls to the RAPP platform:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

We need a callback to save the audio in a file.
You can change the name and type of the audio.

```cpp
auto callback = [&](rapp::object::audio audio) { 
        if (audio.save("audio.wav")) {
            std::cout << "File saved" << std::endl;
        }
        else {
            std::cout << "Error: file not save\r\n";
        }
    };
```

Now, we can make the call. 
The only element we need is a string with the text that we want to record in the audio.
In this case, we are going to start with `hello world`.
If you want to choose a different language you have to change `en` for the language that you prefer.

```cpp
ctrl.make_call<rapp::cloud::text_to_speech>("hello world", "en", callback);
```

##CMakeLists.txt

In this case it assumes that you have built your RAPP API in the **static** and **shared** libraries mode.

This file is going to be the same that we have in [`getting_started/cloud_call/CMakeLists.txt`](../../getting_started/cloud_call/CMakeLists.txt) file.
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

3. If everything is ok, you will have created your executable `text_to_speech` in the folder build.
4. Run your executable
    ```
    ./text_to_speech
    ```
At the end of the process you'll have an `audio.wav`in your `build` folder.
Now you can explore and make your own projects!
