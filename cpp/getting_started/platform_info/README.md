#RAPP Platform
---------------

The main concept of RAPP Platform is based on the cloud robotics concept. 
Currently, the RAPP Platform is a collection of ROS nodes and back-end processes that aim to deliver ready-to-use generic services to robots. The ROS nodes provide generic functionalities, such as Image processing, Audio processing, Speech-to-text and Text-to-speech, Ontology & Database operations, and other procedures.

Querying available services through the API with a `service controller` is normally the first step.
However, before doing that we have to configure the parameters which are needed to connect to the platform:

##Platform
Structure `rapp::cloud::platform` in `rapp-api/cpp/rapp/cloud/asio/platform.hpp`.
It saves the parameters of the RAPP platform.
The parameters are the following:

* **IP/domain**: use an IP (e.g., `155.207.19.229`) or a domain (e.g., `rapp.ee.auth.gr`)
* **Port**: default port is `9001`
* **Authentication token**: needed to authorise access to the platform service
* **Protocol**:  optional: you can choose between `http` or `https` protocols. Will default to HTTP.

In every application you create, you must define the platform endpoint, by creating a `platform` object and using the parameters above.

Alternatively,  you can use a `yaml` configuration file and load the parameters from it.

##Config
Class `rapp::config` in `rapp-api/cpp/rapp/config/config.hpp`.
It loads the parameters of the platform from a file.

**Methods of the class**
* `std::string rapp::config::load_auth_token(std::string filename)`: Return `token` parameter from file.
* `rapp::cloud::platform rapp::config::load_yaml_config(std::string filename)`: Return a `platform` variable fill with `ip/domain`, `port` and `protocol` from a `yaml` file.

You can see an example in `source/config.cpp`.

###Build example

To build the example, follow the next steps:

1. Go to your project path (in our case `config/`)
2. Build your project:
```
mkdir build
cd build 
cmake ..
make
```

3. If build was succesful you will now have an executable `config` in the folder `build`.
4. Run your executable
    ```
    ./config
    ```
