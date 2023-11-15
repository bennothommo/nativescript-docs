---
title: Error handling
---

Errors in NativeScript are handled differently to how they operate in a web-based app. By default, when an unhandled exception is thrown in NativeScript, the app will crash and will trigger the native application error handling, showing the error with the corresponding stack trace in a debugging modal.

While this may be the desired behaviour in **development** mode, an app crash in **production** may seriously hurt an app's credibility and drive away users or customers. In many cases, it may be desirable to have different error handling behaviour depending on whether you are in development mode or in production.

NativeScript allows custom error handling behaviour to be defined through the [Trace module](/core/tracing#error-handling).

## Defining a custom error handler

A custom error handler instance can be defined and registered using the [`Trace.setErrorHandler()`](/core/tracing#seterrorhandler) method. The instance must specify a `handleError()` method that accepts a single argument - the `Error` instance being thrown. You should define your custom error handling early in the [app.ts](/project-structure/main-js-ts) file.

The error handler will receive any uncaught errors that are thrown by the app.

```ts
const errorHandler: TraceErrorHandler = {
  handleError(error: Error) {
    // Handle the error
  },
}

Trace.setErrorHandler(errorHandler)
```

Only one error handler will be registered at a time - any further calls to [`Trace.setErrorHandler()`](/core/tracing#seterrorhandler) will override the previous error handler.

You can also use the [`__DEV__`](/configuration/webpack#global-magic-variables) magic variable to tailor your error handling based on whether you are in development or production.

```ts
const errorHandler: TraceErrorHandler = {
  handleError(error: Error) {
    if (__DEV__) {
      // Handle the error in development - for example, log to console, show app debugger, etc.
    } else {
      // Handle the error in production - for example, report to a bug tracker, direct the user to an error screen, etc.
    }
  },
}

Trace.setErrorHandler(errorHandler)
```

## Examples

### Development Mode

**Allow app crash**: Throw exceptions as soon as an error occurs and crash the app. This is similar to the default behaviour.

```ts
const errorHandler: TraceErrorHandler = {
  handlerError(err) {
    throw err
  },
}
```

**Prevent app crash**: Alternatively, you could write the error message to the console and continue the execution of the app.

```ts
const errorHandler: TraceErrorHandler = {
  handlerError(err) {
    Trace.write(err, 'unhandled-error', type.error)
  },
}
```

### Production Mode

**Prevent app crash**: For example, send an error report to an analytics server but continue app execution.

```ts
const errorHandler: TraceErrorHandler = {
  handlerError(err) {
    reportToAnalytics(err)
  },
}
```

## Disabling native error handling for uncaught errors

NativeScript also allows the prevention of an app crash by disabling the rethrowing of uncaught errors to the native application error handler. This can be done by setting the `discardUncaughtJsExceptions` property to `true` inside the [nativescript.config.ts](/project-structure/nativescript-config) file.

:::tabs

== iOS

```ts
ios: {
...
    "discardUncaughtJsExceptions": true,
},

== Android

android: {
...
    "discardUncaughtJsExceptions": true,

},
```

:::

To handle discarded exceptions, two options are available:

- Listening to the `Application.discardedErrorEvent` and using the received `DiscardedErrorEventData` instance

```ts
import { Application, DiscardedErrorEventData } from '@nativescript/core'

Application.on(
  Application.discardedErrorEvent,
  function (args: DiscardedErrorEventData) {
    const error = args.error

    console.log('Received discarded exception: ')
    console.log(error.message)
    console.log(error.name)
    console.log(error.stack)
    console.log(error.nativeError)
    // for example, report the exception to an analytics solution here
  }
)
```

- Assigning a one-argument function to `global.__onDiscardedError` which will receive the exception as a `NativeScriptError` instance.
