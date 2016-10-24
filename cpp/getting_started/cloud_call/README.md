#Cloud Call
---------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

RAPP Cloud is based on the *PaaS* (Platform as a Service) principle, 
and the core notion is that you make requests to the platform and you will receive replies.
It is **important** to note that the calls are asynchronous, and therefore the reply to your query
is non-deterministic (it is not possible to guarantee time of execution). 
Different queries will require different processing time,
and as such you can run other code on the robot whilst a query is running on the cloud.

## Service Controller

RAPP is divided in two main parts:

* a platform which is either on a cloud server, or on a computer you've setup.
* an API which is a library and  headers that you use to communicate with the platform.

The `service_controller` class is the controller used to run cloud calls. 
When you instantiate an object, you have to specify the **address** and **port** of the platform.

To see this better, we are going to explain `available_services` example.

##Available Services

In the following example we are going to query all the services that are available on the _rapp-platform_.
We do that so we can obtain a list of possible cloud calls.

Include the proper headers to your example. 
In this case we only need three:

1. `rapp/cloud/service_controller.hpp`
2. `rapp/cloud/available_services.hpp`

First we setup the struct `rapp::cloud::platform` which contains:
1. The paltform hostname (`rapp.ee.auth.gr` or `155.207.19.229`). 
2. The platform port
3. Your authentication token

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
```

Then create a service controller (`ctrl`), which is in charge of creating and controlling cloud calls to the platform.
The only argument needed is the information about the platform, which is saved in `info` variable:

```cpp
rapp::cloud::service_controller ctrl(info);
```

Create a callback functor which will receive the reply from the cloud platform:

```cpp
auto cb = [](std::vector<std::pair<std::string, std::string>> services) {
    std::cout << "available services: " << std::endl;
    for (const auto & pair : services) {
        std::cout << pair.first << " " << pair.second << std::endl;
    }
};
```

Finally, make the cloud call:

```cpp
ctrl.make_call<rapp::cloud::available_services>(cb);
```

*Note:* This call is going to block the service controller queue until it completes.
You can run multiple calls using multiple service controllers, or a batch of calls (shown below)

The reply will be:

```
available services: 
path_planning_upload_map http://rapp.ee.auth.gr:9001/hop/path_planning_upload_map
path_planning_plan_path_2d http://rapp.ee.auth.gr:9001/hop/path_planning_plan_path_2d
set_noise_profile http://rapp.ee.auth.gr:9001/hop/set_noise_profile
speech_detection_google http://rapp.ee.auth.gr:9001/hop/speech_detection_google
weather_report_forecast http://rapp.ee.auth.gr:9001/hop/weather_report_forecast
speech_detection_sphinx4 http://rapp.ee.auth.gr:9001/hop/speech_detection_sphinx4
text_to_speech http://rapp.ee.auth.gr:9001/hop/text_to_speech
ontology_subclasses_of http://rapp.ee.auth.gr:9001/hop/ontology_subclasses_of
hazard_detection_light_check http://rapp.ee.auth.gr:9001/hop/hazard_detection_light_check
human_detection http://rapp.ee.auth.gr:9001/hop/human_detection
register_user_from_store http://rapp.ee.auth.gr:9001/hop/register_user_from_store
object_recognition_caffe http://rapp.ee.auth.gr:9001/hop/object_recognition_caffe
weather_report_current http://rapp.ee.auth.gr:9001/hop/weather_report_current
ontology_superclasses_of http://rapp.ee.auth.gr:9001/hop/ontology_superclasses_of
qr_detection http://rapp.ee.auth.gr:9001/hop/qr_detection
cognitive_get_history http://rapp.ee.auth.gr:9001/hop/cognitive_get_history
news_explore http://rapp.ee.auth.gr:9001/hop/news_explore
register_user_from_platform http://rapp.ee.auth.gr:9001/hop/register_user_from_platform
geolocation http://rapp.ee.auth.gr:9001/hop/geolocation
login_user http://rapp.ee.auth.gr:9001/hop/login_user
hazard_detection_door_check http://rapp.ee.auth.gr:9001/hop/hazard_detection_door_check
cognitive_get_scores http://rapp.ee.auth.gr:9001/hop/cognitive_get_scores
ontology_is_subsuperclass_of http://rapp.ee.auth.gr:9001/hop/ontology_is_subsuperclass_of
face_detection http://rapp.ee.auth.gr:9001/hop/face_detection
available_services http://rapp.ee.auth.gr:9001/hop/available_services
cognitive_record_performance http://rapp.ee.auth.gr:9001/hop/cognitive_record_performance
cognitive_test_chooser http://rapp.ee.auth.gr:9001/hop/cognitive_test_chooser
email_send http://rapp.ee.auth.gr:9001/hop/email_send
email_fetch http://rapp.ee.auth.gr:9001/hop/email_fetch
```

Thus, the platform has given you a list with the active running cloud services,
which you can now use.
