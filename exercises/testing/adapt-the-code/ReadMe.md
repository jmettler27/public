# Adapt the code!

In these exercises, you will change some provided code to allow you to test it. You will then perform some basic debugging. 
For this, we recommend that you use an IDE such as IntelliJ or Android Studio.

The code we provide you instantiates its own dependencies. This is problematic for multiple reasons:
- Your tests may fail because of factors outside your control; for instance, testing a class that gets data from the Web may 
  fail if the Internet connection is down for any reason.
- It may be impractical to simulate some failure cases; for instance, if you want to test what happens when a Web server 
  is down, your tests would have to run when it really is down, which is hard to predict.
- Depending on where your tests are executed, they may not have access to external resources, such as a particular Internet 
  service that is available only in some countries.
- Your test may change the state of the external service, in a way that is not recoverable (e.g., delete a user). You would 
  not want your tests to wreak havoc in your production database!

You will change the code to use _dependency injection_ instead.
The idea is simple: instead of instantiating the objects and services that a class depends on (i.e., its "dependencies"), 
a class should receive them as parameters to its constructor (i.e., get them "injected"). Those dependencies should be 
_interfaces_, not specific implementations.

This will also be particularly useful to you when working with frameworks such as Android, which require your code to 
have a certain shape.


## 1. Weather service

We provide a [`WeatherService`](src/main/java/WeatherService.java) class that calls an 
[`HttpClient`](src/main/java/HttpClient.java) class to retrieve weather information over the Internet.

Refactor the `WeatherService` so that it can be tested and write tests for it.


### Side node: without changing the constructor

Sometimes you need to test a class, but you cannot change its constructor parameters, for instance because existing code 
depends on the current constructor signature.
In this case, you can add _another_ constructor to the class for dependency injection. For instance, if for some reason 
you could not modify the `WeatherService` constructor above, you could modify the class as follows:

```java
class WeatherService {
  private final HttpClient client;

  public WeatherService() {
    this.client = new RealHttpClient();
  }

  public WeatherService(HttpClient client) {
    this.client = client;
  }

  public Weather getWeatherToday() { ... /* omitted for brevity */ ... }
}
```

In this way, the class can be used without providing constructor parameters, but you can also inject the dependencies 
during testing.


## 2. External dependencies

Another issue is that the dependencies may not be defined in your code, and thus may not have a corresponding interface. 
For instance, consider the simplified case of Google sign-in:

```java
class GoogleUser {
  // name, email, age, etc.
}

class GoogleService {
  public static GoogleUser signIn() { /* ... */ }
}
```

Not only does `GoogleService` not implement an interface, but its `signIn` method is static! To avoid this issue, create 
the interface yourself, then use the [Adapter pattern](https://en.wikipedia.org/wiki/Adapter_pattern) to create the "real" 
implementation. The Adapter pattern is a simple concept: create adapter code so that a type can be used as if it implemented 
an interface. This is just like real-life adapters, where a Swiss laptop charger can be used in Japan with an adapter 
that "implements" the Swiss electrical "interface" on top of the Japanese one. For instance, in the Google sign-in example, 
the code could look like this:

```java
class MyUser {
  // ... properties of GoogleUser that you want to expose to the app ...
  // This class exists so that the rest of the app doesn't depend on GoogleUser;
  // after all, Google is just one way to sign in, you could in the future add Facebook or Twitter.
}

interface SignInService {
  MyUser signIn();
}

class GoogleSignInAdapter implements SignInService {
  public MyUser signIn() {
    GoogleUser user = GoogleService.signIn();
    // ... convert 'user' to an object of class MyUser, and return it ...
  }
}
```

Now any class that used to directly call `GoogleService` can instead have a `SignInService` as a dependency, so that you 
can write proper tests, and you can use `GoogleSignInAdapter` for the actual implementation in your app.

### Treasure Finder

We provide [`TreasureFinder.java`](src/main/java/TreasureFinder.java). It uses a 
[`Geolocator`](src/main/java/ch/epfl/sweng/locator/Geolocator.java) which comes from an external library.
The `TreasureFinder` determines how close the user is to a treasure.
However, it currently cannot be tested properly because `Geolocator` does not implement any interfaces and uses the real 
location of the user. You also see that `Geolocator` returns an imprecise location encoded in a proprietary type 
[`PositionRange`](src/main/java/ch/epfl/sweng/locator/PositionRange.java) which differs from the 
[`Position`](src/main/java/Position.java) class that you are using.
Furthermore, `TreasureFinder` must have a parameterless constructor, because of external requirements, and you cannot 
modify `Geolocator`.
Refactor `TreasureFinder` so that it can be tested with a fake location service, and write tests for it. Note that the 
code located in the `ch/epfl/sweng/locator` package comes from an external library and therefore cannot be modified.

## 3. Debugging

You learn from a strange email that the precise location of the treasure is `46.516659, 6.5641015`. You decide to test 
your code with these values but you see that the output of the `TreasureFinder` is not exactly what you expected.

Use the debugger of your IDE to step through the execution and find out what bug is causing this strange behavior. In the 
context of this exercise, do not use `System.out.println()`. Even if it can be an efficient tool for finding bugs, you 
should learn to use a debugger.

What is the source of the bug ? Are you able to fix it immediately ? What would be the next logical step into fixing the bug ?

## 4. Mocking

Now that you've written a few tests using dependency injection, you're probably thinking that writing full implementations 
of the dependencies for every test is rather annoying. And in a real application, those interfaces would likely have many 
methods, all of which need to be implemented in every fake dependency class! To solve this, use a _mocking library_.

Mocking libraries help you write fake dependencies, also called _mocks_, by making it easy to create implementations of 
any dependency interface that return the values you want, instead of having to write an implementation of the interface 
every time.

We recommend using the _Mockito_ library; take a look at the [Mockito website](https://site.mockito.org/#how) for a quick 
tutorial. Note that the library is already imported in the project, so you can start to use it right away!

Rewrite the tests from step 2 to use Mockito instead of ad-hoc interface implementations.
