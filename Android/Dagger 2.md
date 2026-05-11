### Best practices
- The `ApplicationComponent` should always be in the `app` module.
- Create Dagger components in modules if you need to perform field injection in that module or you need to scope objects for a specific flow of your application.
- For Gradle modules that are meant to be utilities or helpers and don't need to build a graph (that's why you'd need a Dagger component), create and expose public Dagger modules with @Provides and @Binds methods of those classes that don't support constructor injection.
- To use Dagger in an Android app with feature modules, use component dependencies to be able to access dependencies provided by the `ApplicationComponent` defined in the `app` module.