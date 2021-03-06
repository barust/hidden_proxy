# hidden_proxy
A simple API that shows how,to use Hidden class to create proxies.

This is a replacement of `java.lang.reflect.Proxy` API of Java using a more modern design leading to must faster proxies implementation. See the javadoc of `HiddenProxy` for more information.
 
This code is build with Maven and will require Java 15 to run, currently it requires the nestmates branch of the OpenJDK projet Vahalla because the Hidden class API [JEP 371](https://openjdk.java.net/jeps/371) has not yet being integrated.

Here are pre-compiled versions of the JDK 15 with the Hidden Class API
- linux : https://github.com/forax/java-next/releases/download/untagged-f3c3bf2d93bb686c17a1/jdk-15-nestmates-linux.tar.gz
- macOs : https://github.com/forax/java-next/releases/download/untagged-f3c3bf2d93bb686c17a1/jdk-15-nestmates-osx.tar.gz

## Example

Let say we have an interface `HelloProxy` with an abstract method `hello` and a static method `implementation` somewhere else
```java
  interface HelloProxy {
    String hello(String text);
  }
  static class Impl {
    static String implementation(int repeated, String text) {
      return text.repeat(repeated);
    }
  }
```

The method `HiddenProxy.defineProxy(lookup, interface, field type, linker)` let you dynamically create a class that implements the interface `HelloProxy` with a field (here an int). The return value of `defineProxy` is a constructor you can invoke with the value of the field.
Then the first time an abstract method of the interface is called, here when calling `proxy.hello("proxy")`, the linker is called to ask how the abstract method should be implemented. Here we lookup for `implementation` and discard (using `dropArguments`) the first argument (the proxy) before calling `implementation`. The `implementation` is called with the field stored inside the proxy as first argument followed by the arguments of the abstract method.
```java
  var linker = (InvocationLinker)(lookup, methodInfo) -> {
      return switch(methodInfo.getName()) {
        case "hello" -> MethodHandles.dropArguments(
            lookup.findStatic(Impl.class, "implementation", methodType(String.class, int.class, String.class)),
            0, HelloProxy.class);
        default -> fail("unknown method " + methodInfo);
      };
    };
  var constructor = HiddenProxy.defineProxy(MethodHandles.lookup(), HelloProxy.class, int.class, linker);
  var proxy = (HelloProxy)constructor.invoke(2);
  assertEquals("proxyproxy", proxy.hello("proxy"));
```

If you want more examples, you can take a look to the test class `HiddenProxyTest`.

