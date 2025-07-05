+++
title = "Mocking the Right Way in C++"
date = "2025-07-05"

[extra]
comment = false
+++

Testing is important. I think every software developer would agree on that. But
what ways are there to improve software testing? What if a system is too
complex to be simply unit tested with pre-determined inputs and checking its
outputs? What, if the outputs only take place on a serial bus or with an HTTP
request?

> Well, we don't have any way to test this, then.

I've heard before. But this is where mocks come in to play!

Mocks are objects that are doubles of objects used within a business logic,
providing the same interface and looking identical from the outside. But
internally, their only purpose is to perform checks how they are "used" in the
context of testing. Ideally, a framework like
[googlemock](https://google.github.io/googletest/reference/mocking.html) is
used to get fine control over

- Which functions are called
- How often they are called
- In what order they are called
- What data shall be passed to them
- What they should return

This way, it can get fairly easy to test complex systems as well, using
dependency injection to "smuggle" the mock object instead of the real deal into
a business logic.

I've seen an approach leveraging dynamic polymorphism before, that I want to
review, and then come up with a better solution.

# A Difficult Class to Test

First of all, let's start with a `Serial` class that represents a serial bus,
or a similar hardware abstraction.

```cpp
class Serial {
public:
    void open_serial()  {
        // ... logic to open the serial bus
        std::cout << "Bus open\n";
    }
    void send_data(std::span<const std::uint8_t> data)  {
        // ... logic to send the data over the bus
        (void)data;
        std::cout << "Data sent over bus\n";
    }
};
```

Now, assume there is object we want to test, `UnitUnderTest`, that makes use of
the `Serial` class.

```cpp
class UnitUnderTest {
public:
    UnitUnderTest() = default;
    void work() const {
        // Actual business logic
        std::array<uint8_t, 127> data{1, 3, 3, 7};
        serial_.open_serial();
        serial_.send_data(std::span<const std::uint8_t>(data.data(), 4));
    }
private:
    Serial serial_;
};
```

This object has a `work` function that sends data over the serial bus, which is
accessed via the `serial_` member of the class.

Now the crucial question: How to write a test for the `UnitUnderTest` class?
The `work` method returns `void` and calls methods of the `Serial` class. There
is nothing that we can observe from the outside of this class to understand
what happens internally.

# Mocking with Dynamic Polymorphism

As outlined above, let's create a mock object of `Serial` for testing purposes,
that checks if the calls to it were made correctly. When using dynamic
polymorphism, a common abstract base class (or interface) is created that
specifies the interface of any `Serial` implementation:

```cpp
class ISerial {
public:
    virtual void open_serial() = 0;
    virtual void send_data(std::span<const std::uint8_t> data) = 0;
};
```

Next, the `Serial` class needs to inherit from this interface.

```cpp
class Serial : public ISerial {
public:
    void open_serial() override {
        // ... logic to open the serial bus
        std::cout << "Bus open\n";
    }
    void send_data(std::span<const std::uint8_t> data) override {
        // ... logic to send the data over the bus
        (void)data;
        std::cout << "Data sent over bus\n";
    }
};
```

On a site note, see that the `override` keyword was added to make sure that
these methods actually override a virtual method in the base class, which is a
good way to catch bugs.

Next, we have to implement a mock serial object that implements the `ISerial`
interface.

```cpp
class SerialMock : public ISerial {
public:
    void open_serial() override {
        // ... logic to check number of calls to this function, etc.
        std::cout << "SerialMock open_serial()\n";
    }
    void send_data(std::span<const std::uint8_t> data) override {
        // ... logic to check number of calls to this function, etc.
        std::cout << "SerialMock send_data()\n";
    }
};
```

Note that this is just an example, ideally this would leverage a mocking
framework as outlined in the introduction.

Lastly, we need to make `UnitUnderTest` use different derived classes of
`ISerial`, determined dynamically at runtime:

```cpp
class UnitUnderTest {
public:
    // Dependency injection via reference
    UnitUnderTest(ISerial& serial) : serial_(serial) {};
    void work() const {
        // Actual business logic
        std::array<uint8_t, 127> data{1, 3, 3, 7};
        serial_.open_serial();
        serial_.send_data(std::span<const std::uint8_t>(data.data(), 4));
    }
private:
    ISerial& serial_;
};
```

Before we review this approach, let's look at the usage of the class in the
business logic and in tests.

```cpp
// Business logic case
Serial serial; // Note that lifetime needs to be managed outside UnitUnderTest!
UnitUnderTest business(serial);
business.work();

// Test case
SerialMock serial_mock; // Note that lifetime needs to be managed outside UnitUnderTest!
UnitUnderTest test(serial_mock);
test.work();
```

The full example can be found [here](https://godbolt.org/z/8xj6nrYG1).

We can make the following observations:

- We have to extend the constructor of `UnitUnderTest` with an additional
  parameter for dynamic dependency injection
- The lifetime of the `ISerial` implementation needs to be managed _outside_
  the `UnitUnderTest` class
- The usage of `UnitUnderTest` becomes cumbersome; developers usually would
  expect to construct the class without arguments and have to think about the
  dependency injection due to mocking even outside of test scenarios
- Dynamic polymorphism has an impact on runtime

Overall, that doesn't look too great. Let's try a better approach next,
leveraging static polymorphism.

# Mocking with Static Polymorphism

Let's assume the same starting point as in the first section.

First, we create a `SerialMock` object without defining an interface first.

```cpp
class SerialMock {
public:
    void open_serial() {
        // ... logic to check number of calls to this function, etc.
        std::cout << "SerialMock open_serial()\n";
    }
    void send_data(std::span<const std::uint8_t> data) {
        // ... logic to check number of calls to this function, etc.
        std::cout << "SerialMock send_data()\n";
    }
};
```

Next, instead of leveraging dynamic polymorphism via a constructor argument, we
change the `UnitUnderTest` class to become a class template.

```cpp
template<class SerialType> // Static Polymorhism
class UnitUnderTestTemplate {
public:
    UnitUnderTestTemplate() = default;
    void work() {
        // Actual business logic
        std::array<uint8_t, 127> data{1, 3, 3, 7};
        serial_.open_serial();
        serial_.send_data(std::span<const std::uint8_t>(data.data(), 4));
    }
private:
    SerialType serial_;
};
```

A class template is a instruction how to instantiate concrete class definitions
at compile time. This allows us to let the compiler generate in fact two
classes for us: One with the mocked serial object, and one with the actual one
for usage in the business logic.

Without an interface, how can we make sure that the mock object implements the
same interface as the original object? I would argue, since we use both objects
the same way in `UnitUnderTest`, we enforce that they can be treated the same
way at compile time. If we would miss a method in the mock class that we
actually use in `UnitUnderTest`, the compilation would fail. Basically, the
business logic becomes the specification which interface must be implemented.
Also, in C++20 or later,
[concepts](https://en.cppreference.com/w/cpp/language/constraints.html) can be
used to pose restrictions on template arguments, but that's out of scope for
this article.

To improve the usage, I renamed the class template and will add a type alias
with the original name.

```cpp
using UnitUnderTest = UnitUnderTestTemplate<Serial>;
```

This completely eliminates having to think about mocking when using the class.
The usage looks like this:

```cpp
// Note that Serial lifetime is managed internally
// Note the nice ergonomics - no constructor argument necessary, just use the class while ignoring any mocking here
UnitUnderTest business;
business.work();

// Test case
UnitUnderTestTemplate<SerialMock> test;
test.work();
```

The full example is [here](https://godbolt.org/z/cPc6MK7WE).

Let's review this approach:

- The lifetime of the `Serial` object is managed internally in `UnitUnderTest`;
  exactly like it was before introducing the mock
- The constructor takes the same arguments as before as well, no additional
  parameters
- Due to the type alias, usage of the `UnitUnderTest` class is as developers
  would expect it; just instantiate it without any thoughts of mocking
- In a test scenario, use the template explicitly to create a class containing
  a mocked object
- Static polymorphism doesn't add any runtime overhead
