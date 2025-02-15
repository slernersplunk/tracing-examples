## Part 4: Add some manual instrumentation

The SplunkRum class in the Android RUM library contains a bunch of options for doing manual instrumentation and for
customizing the telemetry that is sent to Splunk RUM. In this part of the workshop, you'll try a couple of them, and you
can explore the rest on your own.

| Objective  | To add some manually instrumented spans and events to the app. |
| ---        | ---            |
| Duration   | 10 minutes +   | 
| Difficulty | Advanced       |

#### 1. Add a "workflow" to the app, and add some custom attributes.

In Splunk's RUM implementation, a "workflow" is a custom span for which metrics will be automatically collected. In the
Android instrumentation, these workflows are modeled as OpenTelemetry Span instances.

Let's create a workflow for the user's login. This will be triggered by the button labled "LOGIN" on the first screen of
the app.

1. The code for this is in the `com.splunk.android.workshopapp.FirstFragment` class. Open up that class in Android
   Studio.
2. Find this block of code in the `onViewCreated` method:
   ```
        binding.loginButton.setOnClickListener(v -> {
            //not really a login, but it does make an http call
            makeCall("https://ssidhu.o11ystore.com/");
        });
   ```
3. Change this block of code to look like this:
   ```
        binding.loginButton.setOnClickListener(v -> {
            Span loginSpan = SplunkRum.getInstance().startWorkflow("Login");
            try (Scope s = loginSpan.makeCurrent()) {
               //not really a login, but it does make an http call
               makeCall("https://ssidhu.o11ystore.com/");
            } finally {
                loginSpan.end();
            }
        });
   ```
   When prompted by Android Studio, add imports for the OpenTelemetry `Span` and `Scope` classes:
   ```
      io.opentelemetry.api.trace.Span;
      io.opentelemetry.context.Scope;
   ```
4. Add a custom "global attribute" after the `makeCall` line (but before the `finally` block):
   ```
   SplunkRum.getInstance().setGlobalAttribute(stringKey("user_id"), "123456");
   ```
   This attribute will now be added to every span that is generated by the instrumentation.

   Be sure to add the static import for the OpenTelemetry `AttributeKey.stringKey()` method when prompted. A "stringKey"
   is a key for a value that must be a java `String`. There are corresponding key types for longs, doubles and booleans,
   in addition to arrays of the base types.

   The code should now look like this:

  ```
        Span loginSpan = SplunkRum.getInstance().startWorkflow("Login");
        try (Scope s = loginSpan.makeCurrent()) {
          //not really a login, but it does make an http call
          makeCall("https://ssidhu.o11ystore.com/");
          
          //add a global attribute for the user's id (pretend we got this id back from the http call)
          //after this, the attribute will be added to every span generated by the instrumentation (not this span, though)
          SplunkRum.getInstance().setGlobalAttribute(stringKey("user_id"), "123456")
        } finally {
          loginSpan.end();
        }
  ```

5. Build and restart the app. Try out the "LOGIN" button. Can you find workflow Event metrics in the UI?

#### 2. Bonus content: Try out other manual instrumentation options

Here are some other ideas to try, if you have the time:

1. Set some global attributes when building the `Config` that you created in Part 2.
2. Create your own workflows.
3. Write some customization of the `Config.Builder` that will change or remove span attributes.
4. Create a custom Event or an Exception.
5. Use the
   [OpenTelemetry APIs](https://opentelemetry.io/docs/java/manual_instrumentation/#create-a-basic-span)
   directly to create your own custom spans.
6. Did you notice that the "Login" workflow span ended before the http call was complete? Can you update the code to
   make it so the workflow isn't complete until the http call is complete?

