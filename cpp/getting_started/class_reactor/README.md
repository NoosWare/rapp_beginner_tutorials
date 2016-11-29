# Class reactor
--------------

* *This tutorial assumes that RAPP API is installed and built*
* *This tutorial uses the 0.7.0 version* of the C++ API

Running cloud calls using lambdas may not always be the best solution. 
Under certain conditions a class which handles callbacks may be a better.
In this scenario we define and implement a class which does just that.
The `reactor` class methods will be used to handle cloud replies:

```cpp
class reactor
{
public:

    void handle_face(std::vector<rapp::object::face> faces)
    {        
        std::cout << "Found " << faces.size() << " faces!" << std::endl;
    }

    void reactor::handle_ontology_sub(std::vector<std::string> classes)
    {        
        std::cout << "Sub classes: " << std::endl;
        for (const auto & str : classes) {
            std::cout << str << std::endl;
        }    
    }

    void handle_weather(std::vector<std::string> weather) 
    {
        std::cout << "Temperature: " << weather.at(1) << std::endl;
        std::cout << "Humidity: " << weather.at(3) << std::endl;
        std::cout << "Wind speed: " << weather.at(6) << std::endl;
    }

    ...
};
```

The reactor class also wraps around a `service_controller` and thus controls the calls.

```cpp
class reactor
{
    ...

private:
    rapp::cloud::service_controller ctrl_;
};
```

We construct it using:

```cpp
reactor::reactor(rapp::cloud::platform info)
: ctrl_(info)
{}
```

Making the calls is done using the `run` method. We also use it to bind to the handler methods.

```cpp
void reactor::run(rapp::object::picture pic, std::string object, std::string city)
{
    ctrl_.make_calls(face_detection(pic, false, std::bind(&reactor::handle_face, this, std::placeholders::_1)),
                     weather_report_current(city, "", 0, std::bind(&reactor::handle_weather, this, std::placeholders::_1)),
                     ontology_subclasses_of(object, true, std::bind(&reactor::handle_ontology_sub, this, std::placeholders::_1)));
}
```

Thus this method will run three different cloud calls as a batch job, whilst
re-using its own methods to receive the replies.
Because we're using class methods we have to either use `std::function` class members,
or use a `bind` (either `std::bind` or `boost::bind`) to the class methods.
A lambda callback could also be used, but it would have to capture `this`.

Constructing a unique pointer to our reactor class is done like so:

```cpp
rapp::cloud::platform info = {"rapp.ee.auth.gr", "9001", "rapp_token"}; 
auto my_reactor = std::make_unique<reactor>(info);
```

And finally we execute the `run` method:

```cpp
my_reactor->run(pict, "Toy", "Madrid");
```

The reply you'll receive is:

```
Sub classes:
http://knowrob.org/kb/knowrob.owl#Toy
Temperature: 79.36
Humidity: 0.26
Wind speed: 2.5
Found 1 faces!
```

Therefore, for more sophisticated use of a group of cloud calls, especially if post-processing or pre-processing
is required, or if taking an object-oriented approach, a reactor class may be better suited.
