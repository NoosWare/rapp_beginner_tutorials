#Callback
-------

Since C++11, a `callback` is a [callable](http://en.cppreference.com/w/cpp/concept/Callable) accepted by a class or function, used to customize the current logic depending on that callback.

One reason to use callbacks is to write generic code which is independant from the logic in the called function and can be reused with different callbacks.

When making cloud calls, and due to their asyncrhonous nature, we pass callback parameters.
This approach is common throught the entire API; almost all cloud calls expect you to pass a callback functor.

Your callback can also be a class method, which is bound (using `std::bind` or `boost::bind`)
however we suggest you use lambdas or `std::function` when starting.

There are different methods to implement it:
* Function pointer 
* `std::function` objects
* Templates

We are going to explain the second one, because it's the one we are using it.

**For more information about all type of callbacks, you can see** [here](http://stackoverflow.com/questions/2298242/callback-functions-in-c).

##`std::function` objects
Class template std::function is a general-purpose polymorphic function wrapper. 
The complete definition you can find it [here](http://en.cppreference.com/w/cpp/utility/functional/function).

An example of the syntax of a `std::function`:

```cpp
std::function<void(int)>
```

Where `int` is the input arguments of this function and `void` is the return arguments.

There are three ways to use it:

### Using a free function
In this case you only need to have a function and initialize the `std::function` with this function.
For example:

```cpp
void print_int(int i)
{
    std::cout << i << std::endl;
}

int main() {

    std::function<void(int)> f_print = print_int;
    f_print(5);
    return 0;
}
```

Both functions have to have the same input and return arguments types.

###`std::bind`
The function template bind generates a forwarding call wrapper for a callable object. 
The complete definition you can find it [here](http://en.cppreference.com/w/cpp/utility/functional/bind).

It's another way to call the function. With an example we'll see better:

```cpp
void print_int(int i)
{
    std::cout << i << std::endl;
}

int main() {

    std::function<void(int)> f_print = std::bind(print_int, 5);
    f_print();
    return 0;
}
```

In this case we are invoking `print_int` passing the arguments with std::bind. 
When we need to call `f_print` we won't use input arguments because we wrote them before.

In this example it doesn't seem very useful. However, when we have a more complex programming structure is really useful.
For example:

```cpp
class Foo {
public:
    Foo(int num) : num_(num) {}
    void print_add(int i) const { std::cout << num_ + i << std::endl; }
private:
    int num_;
};

int main() {

    Foo num(5);
    std::function<void(int)> f_add = std::bind(&Foo::print_add, num, std::placeholders::_1);
    f_add(2);

    return 0;
}
```

In this case, we are associating our `std::function` with `Foo::print_add`.  
In this case we have to add `std::placeholders` to indicate the position of the argument. 
We only have one argument, so the placeholder indicate the `_1` position.

For more information about `std::placeholders` you can see in [cppreference](http://en.cppreference.com/w/cpp/utility/functional/placeholders).

To see an example in RAPP API click [here](../class_reactor/README.md).

###Lambda functions

A lambda function is an anonymous, temporary and usually small function.
To read the complete definition you can go to [cppreference](http://en.cppreference.com/w/cpp/language/lambda).
This way is the one which we are going to use the most in RAPP API.

Following the examples above:

```cpp
class Foo {
public:
    Foo(int num) : num_(num) {}
    void print_add(int i) const { std::cout << num_ + i << std::endl; }
private:
    int num_;
};

int main() {

    Foo num(5);
    std::function<void(int)> f_add = [](int a) { Foo::print_add(a) };
    auto f_add_2 = [](int a) { Foo::print_add(a) };
    f_add(2);
    f_add_2(2);

    return 0;
}
```

In the first case, we are using the same method than examples above. 
We are saving our lambda function in a `std::function`.

However, since `C++11` we can use [auto specifier](http://en.cppreference.com/w/cpp/language/auto) to declare variables.
It's simpler in that way.

So, in all the examples of RAPP API you'll find a lambda callback with the input arguments depending of the cloud call which you want to do.

This is how you can obtain the data from the platform easily and use it like you need for your projects.
