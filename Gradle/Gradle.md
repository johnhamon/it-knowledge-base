## [API and implementation separation](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_separation)

The key difference between the standard Java plugin and the Java Library plugin is that the latter introduces the concept of an _API_ exposed to consumers. A library is a Java component meant to be consumed by other components. It’s a very common use case in multi-project builds, but also as soon as you have external dependencies.

The plugin exposes two [configurations](https://docs.gradle.org/current/userguide/declaring_dependencies.html#sec:what-are-dependency-configurations) that can be used to declare dependencies: `api` and `implementation`. The `api` configuration should be used to declare dependencies which are exported by the library API, whereas the `implementation` configuration should be used to declare dependencies which are internal to the component.

Example 2. [Declaring API and implementation dependencies](https://docs.gradle.org/current/userguide/java_library_plugin.html#ex-declaring-api-and-implementation-dependencies)
```Groovy
dependencies {
    api 'org.apache.httpcomponents:httpclient:4.5.7'
    implementation 'org.apache.commons:commons-lang3:3.5'
}
```
Dependencies appearing in the `api` configurations will be transitively exposed to consumers of the library, and as such will appear on the compile classpath of consumers. Dependencies found in the `implementation` configuration will, on the other hand, not be exposed to consumers, and therefore not leak into the consumers' compile classpath. This comes with several benefits:

- dependencies do not leak into the compile classpath of consumers anymore, so you will never accidentally depend on a transitive dependency
- faster compilation thanks to reduced classpath size
- less recompilations when implementation dependencies change: consumers would not need to be recompiled
- cleaner publishing: when used in conjunction with the new `maven-publish` plugin, Java libraries produce POM files that distinguish exactly between what is required to compile against the library and what is required to use the library at runtime (in other words, don’t mix what is needed to compile the library itself and what is needed to compile against the library).


If your build consumes a published module with POM metadata, the Java and Java Library plugins both honor api and implementation separation through the scopes used in the POM. Meaning that the compile classpath only includes Maven `compile` scoped dependencies, while the runtime classpath adds the Maven `runtime` scoped dependencies as well.

This often does not have an effect on modules published with Maven, where the POM that defines the project is directly published as metadata. There, the compile scope includes both dependencies that were required to compile the project (i.e. implementation dependencies) and dependencies required to compile against the published library (i.e. API dependencies). For most published libraries, this means that all dependencies belong to the compile scope. If you encounter such an issue with an existing library, you can consider a [component metadata rule](https://docs.gradle.org/current/userguide/component_metadata_rules.html#sec:component_metadata_rules) to fix the incorrect metadata in your build. However, as mentioned above, if the library is published with Gradle, the produced POM file only puts `api` dependencies into the compile scope and the remaining `implementation` dependencies into the runtime scope.
## [Recognizing API and implementation dependencies](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_recognizing_dependencies)

This section will help you identify API and Implementation dependencies in your code using simple rules of thumb. The first of these is:
- Prefer the `implementation` configuration over `api` when possible
This keeps the dependencies off of the consumer’s compilation classpath. In addition, the consumers will immediately fail to compile if any implementation types accidentally leak into the public API.

So when should you use the `api` configuration? An API dependency is one that contains at least one type that is exposed in the library binary interface, often referred to as its ABI (Application Binary Interface). This includes, but is not limited to:

- types used in super classes or interfaces
- types used in public method parameters, including generic parameter types (where _public_ is something that is visible to compilers. I.e. , _public_, _protected_ and _package private_ members in the Java world)
- types used in public fields
- public annotation types

By contrast, any type that is used in the following list is irrelevant to the ABI, and therefore should be declared as an `implementation` dependency:

- types exclusively used in method bodies
- types exclusively used in private members
- types exclusively found in internal classes (future versions of Gradle will let you declare which packages belong to the public API)