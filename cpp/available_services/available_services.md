#Examples
##Available Services

**This tutorial assumed that RAPP API is installed and built**

In the following example we are going to show all the services that are available in the _rapp-platform_.
So we can know what actions we can do with our robot.

First of all, include the proper headers to your example. That is to associate only the files of the library which we need. 
In this case we only need three:

1. service_controller.hpp
2. available_services.hpp
3. iostream

Now we can start to write the main program. Always that we want to use the _rapp-platform_, we need to take its parameters which are:
1. Its hostname which can be `rapp.ee.auth.gr` or `155.207.19.229`. Besides you can use `localhost`if you are running the platform in your own computer.
2. Its port
3. Authentication token

We are going to save these parameters in a `rapp::cloud::platform` variable call info.

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
```

The next step is to create a service controller (called ctrl), which is in charge of create cloud calls to the platform with the tasks that we need.
The only parameter needed is the information about the platform, which is saved in `info` variable:

```cpp
rapp::cloud::service_controller ctrl(info);
```

We want to make a call to available services class, so first, we need to know what parameters are needed.
In this case, we need a function with a vector of pair of string where we are going to save all the services.
You can find all the information in *rapp/cloud/available_services.hpp*.
In this example a lambda is done to print all the services.

```cpp
auto cb = [](std::vector<std::pair<std::string, std::string>> services) {
    std::cout << "available services: " << std::endl;
    for (const auto & pair : services) {
        std::cout << pair.first << " " << pair.second << std::endl;
    }
};```

Finally, we make the cloud call with the next line:

```cpp
ctrl.make_call<rapp::cloud::available_services>(cb);
```

Where `cb` is the function mentioned above and we have to specify the class that we want,
in this case *rapp::cloud::available_services*.
If everything is ok, a list of services have to be shown.
