#RAPP Platform
---------------

The main concept of RAPP Platform aligns with the cloud robotics approach. RAPP Platform is a collection of ROS nodes and back-end processes that aim to deliver ready-to-use generic services to robots. The ROS nodes of the RAPP platform are back-end processes that provide generic functionalities, such as Image processing, Audio processing, Speech-to-text and Text-to-speech, Ontology & Database operations, as well as ML procedures.

We are going to call the platform from the API with a `service controller` to use its services.
Before doing that we have to configure the parameters which are needed by the platform:

##Platform
Structure `rapp::cloud::platform` in `rapp-api/cpp/rapp/cloud/asio/platform.hpp`.
It saves the parameters of the RAPP platform.
The parameters are the following ones:

* **IP/domain**: You can choose between write the IP `155.207.19.229` or the domain `rapp.ee.auth.gr`. Both are valid.
* **Port**: It's `9001`.
* **Authentication token**: It's needed to authorise access to the platform services. (e.g., `rapp_token`).  
* **Protocol**: It's optional. You can choose between `http` or `https` protocols. The first one is by default.

In every new program that you need to call the platform you'll have to create a new `platform` structure and write the parameters above.

Instead of doing this you can write a single `yaml` file and load the parameters when you need them.

##Config
Class `rapp::config` in `rapp-api/cpp/rapp/config/config.hpp`.
It allows to load the parameters of the platform from a file.

**Methods of the class**
* `std::string rapp::config::load_auth_token(std::string filename)`: Return `token` parameter, if there is, from a file.
* `rapp::cloud::platform rapp::config::load_yaml_config(std::string filename)`: Return a `platform` variable fill with `ip/domain`, `port` and `protocol` from a `yaml` file.

You can see an example in `source/config.cpp`.

###Build example

To build the example, follow the next steps:

1. Go to your project path (in our case `config/`)
2. Build your project
```
mkdir build
cd build 
cmake ..
make
```

3. If everything is ok, you will have created your executable `config` in the folder `build`.
4. Run your executable
    ```
    ./config
    ```
Now you can explore and make your own projects!
