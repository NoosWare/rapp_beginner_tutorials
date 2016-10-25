#Speech recognition using Sphinx4
----------------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

In this tutorial we are going to recognise the speech from an audio using [CMU Sphinx 4](http://cmusphinx.sourceforge.net/wiki/sphinx4:webhome), RAPP and CMake.

You have to configure your project in like this tree:

```
project/
        CMakeLists.txt
        source/
                speech_recognition_sphinx4.cpp
        build/
```

##Source code

We created a project called `speech_recognition_sphinx4` with a folder `source` where we have our
example call `speech_recognition_sphinx4.cpp`.
You can see the complete example [here](source/speech_recognition_sphinx4.cpp).

We are going to initialize the platform information and the service controller, which is in charge
of make the cloud calls to the RAPP platform:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
rapp::cloud::service_controller ctrl(info);
```

The callback for this example need vectors of string to save the words that have been recognised and the possible alternatives.

```cpp
    auto callback = [&](std::vector<std::string> words) { 
            if (words.size() != 0) {
                std::cout << "Words: " << std::endl;
                for (auto each_word : words) {
                    std::cout << each_word << std::endl;
                }
            }
            else {
                std::cout << "No words found" << std::endl;
            }
    };
```

If we see the documentation of the class `rapp::cloud::speech_recognition_sphinx4`,
we'll see that an audio and its type are necessary.

```cpp
rapp::object::audio audio("../data/yes-no.wav");
rapp::types::audio_source audio_src = rapp::types::nao_wav_1_ch;
```
*NOTE*: `audio_source` is a enum. 
You can find the different types of audio in `rapp-api/cpp/rapp/objects/globals.hpp`

Sphinx4 can use a [JSGF](http://cmusphinx.sourceforge.net/wiki/sphinx4:jsgfsupport) input to achieve more accuracy in the recognition.
*It's an optional input.*
```cpp
std::string jsgf =  "#JSGF V1.0;\r\n\r\n";
            jsgf += "grammar simpleExample; \r\n\r\n";
            jsgf += "public <greet> = Yes | No;\r\n";
```

Finally, we can make the call to the platform.

```cpp
ctrl.make_call<rapp::cloud::speech_recognition_sphinx4>(audio.bytearray(), 
                                                        audio_src, 
                                                        "en",
                                                        std::vector<std::string>({{jsgf}}),
                                                        std::vector<std::string>({"yes", "no"}),
                                                        std::vector<std::string>({""}),
                                                        callback);
```

Besides the audio and the callback you need to indicate the language, in this case English `"en"`.
These are the same inputs as speech recognition using Google. 
CMU Sphinx allows us to add some parameters to improve the accuracy, like a JSGF, keywords or sentences to find in the audio.
All of them are optional inputs.

##CMakeLists.txt

In this case it assumes that you have built your RAPP API in the **static** and **shared** libraries mode.

We haven't added any new library to do this example, so the CMakeLists.txt is the same that [`getting_started`](../../getting_started/) examples.

You only have to change the name of your source file and the executable.

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

3. If everything is ok, you will have created your executable `speech_recognition_sphinx4` in the folder build.
4. Run your executable
    ```
    ./speech_recognition_sphinx4
    ```

Now you can explore and make your own projects!
