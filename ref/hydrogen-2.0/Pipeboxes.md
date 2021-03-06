<!-- REFERENCES -->

[REF_PHPA]: ./Pipeboxes.Abstractions.md
[REF_PHPL]: ./Pipelines.md

<!-- DOCUMENT -->

# Primavera.Hydrogen.Pipeboxes

**Class library that contains types that implement the Pipeline design pattern with multiple execution flows.**

## About

This class library is the default implementation of the [Primavera.Hydrogen.Pipeboxes.Abstractions][REF_PHPA]. The main purpose of this implementation is to support the Pipeline design pattern with multiple execution flows. Alternatively, you can use [Primavera.Hydrogen.Pipelines][REF_PHPL] which is a single implementation of the recursive flow.

## Getting Started

The `Pipebox Design Pattern` defines the processing of a complex or time consuming operation into a set of small tasks (handlers) that combine together to form an asynchronous unit of work (or pipeline), with the aim of improving performance, scalability and component reusability in any application or service. This pattern is implemented here by the `Pipebox` set of classes and supplementary types.

Consider the following sample of code:

```csharp
using System.IO;
using Microsoft.Extensions.DependencyInjection;
using Primavera.Hydrogen.Pipeboxes;
using Primavera.Hydrogen.Pipeboxes.Config;
using Primavera.Hydrogen.Pipeboxes.Context;
using Primavera.Hydrogen.Pipeboxes.Handlers;
using Primavera.Hydrogen.Pipeboxes.State;

namespace Primavera.Hydrogen.Samples.Pbx
{
    /// <summary>
    /// Pipebox samples.
    /// </summary>
    internal static class Samples
    {
        /// <summary>
        /// Code samples for the <see cref="Pipebox{T}"/>.
        /// </summary>
        public static void Sample1()
        {
            // Arrange the pipebox configuration

            string json = File.ReadAllText(@".\PipeboxConfig01.json");
            PipeboxConfig config = PipeboxConfig.Create(json);

            // Arrange the service provider registering the
            // default handler type

            ServiceCollection services = new ServiceCollection();
            services.AddTransient(typeof(DefaultHandler<>));
            ServiceProvider provider = services.BuildServiceProvider();

            // Arrange the pipebox context with your data

            string data = "Hello";
            var context = new PipeboxContext<string>(data);

            // Arrange the pipebox to use the specified provider,
            // configuration and pipeline "p1"

            using Pipebox<string> pipebox = new Pipebox<string>();

            pipebox
                .UseProvider(provider)
                .UseConfig(config)
                .UsePipeline("p1");

            // Run the pipeline "p1"

            pipebox.ExecuteAsync(context).GetAwaiter().GetResult();

            // Get the state of the pipebox execution

            PipeboxState state = pipebox.CurrentState;

            // Get the result of context data

            string result = context.Data;
        }
    }
}
```

### Pipebox

The `Pipebox<T>` provides the implementation of `IPipebox<TContext, PipeboxConfig>`, which is the engine that runs a specified pipeline with a `PipelineContext<T>`.

See the [IPipebox<TContext, TConfig>][REF_PHPA] for more information about the interface members.

### Pipebox Configuration

The `PipeboxConfig` provides the implementation of `<TConfig>`, which is the pipebox configuration that defines one or more pipelines and their corresponding handlers.

**Creating the Configuration**

Consider the following configuration file:

```json
{
  "version": "1.0",
  "expressions": [
    {
      "pattern": "%p2p3%",
      "value": "p2=v2; p3=v3"
    },
    {
      "pattern": "%defaultHandler%",
      "value": "Primavera.Hydrogen.Pipeboxes.Handlers.DefaultHandler\u00601, Primavera.Hydrogen.Pipeboxes"
    }    
  ],
  "pipelines": [
    {
      "id": "p1", 
      "tag": "pipeline 1",
      "configStr": "[{\"p1\":\"v1\"},{\"p2\":\"v2\"}]",      
      "handlers": [
        {
          "id": "h1",
          "tag": "handler 1",
          "configStr": "p1=v1; %p2p3%"
        },
        {
          "id": "h2",
          "tag": "handler 2",
          "type": "%defaultHandler%",
          "configStr": "p1={ p11=v11; p12=v12 }; %p2p3%"
        }
      ]
    }
  ]
}
```

The configuration can be created at runtime or deserialized from file, just like any other application setting. The following example shows how to load the configuration using a JSON file:

```csharp
string json = File.ReadAllText("PipeboxConfig.json");
PipeboxConfig config = PipeboxConfig.Create(json);
```

**Missing and Default Settings**

In the previous example, the `Create` method deserializes the configuration and implicitly executes another three operations:
*  applies regular expression replacements to the `configStr` and `type` values
*  applies default settings to the configuration, if required values are missing
*  asserts the configuration is compliant with the predefined validation rules, or throws

The final result, after processing the configuration, is as follows:

```json
{
  "version": "1.0",
  "expressions": [
    {
      "pattern": "%p2p3%",
      "value": "p2=v2; p3=v3"
    },
    {
      "pattern": "%defaultHandler%",
      "value": "Primavera.Hydrogen.Pipeboxes.Handlers.DefaultHandler\u00601, Primavera.Hydrogen.Pipeboxes"
    }
  ],
  "pipelines": [
    {
      "id": "p1",
      "active": "True",
      "tag": "pipeline 1",
      "configStr": "[{\"p1\":\"v1\"},{\"p2\":\"v2\"}]",
      "handlers": [
        {
          "id": "h1",
          "order": "1",
          "tag": "handler 1",
          "type": "Primavera.Hydrogen.Pipeboxes.Handlers.DefaultHandler\u00601, Primavera.Hydrogen.Pipeboxes",
          "configStr": "p1=v1; p2=v2; p3=v3",
          "active": "True"
        },
        {
          "id": "h2",
          "order": "2",
          "tag": "handler 2",
          "type": "Primavera.Hydrogen.Pipeboxes.Handlers.DefaultHandler\u00601, Primavera.Hydrogen.Pipeboxes",
          "configStr": "p1={ p11=v11; p12=v12 }; p2=v2; p3=v3",
          "active": "True"
        }
      ]
    }
  ]
}
```

**Configuration String**

The `configStr` property is used to define the customized configuration of pipelines and handlers, which only their implementations know how to use, in two possible formats:

Using the `ConfigString` class, which provides a string parser that builds a dictionary of key/value pairs:

```csharp
// For the configuration string
string configStr = "p1=v1; p2={ p21=v21; p22=v22 }; p3=v3";

// Create the configuration string objet
var config = new ConfigString(configStr);

// Get the value of p1
string p1 = config.GetValue<string>("p1");

// Get the value of p2
ConfigStr p2 = config.GetValue<ConfigString>("p2");
string p21 = config.GetValue<string>("p21");
string p22 = config.GetValue<string>("p22");

// Get the value of p3
string p1 = config.GetValue<string>("p3");
```

Using a stringified JSON:

```csharp
// For the stringified JSON
string configStr = "[{\"p1\":\"v1\"},{\"p2\":\"v2\"}]"

// Create the JSON document
var jsonDoc = JsonDocument.Parse(configStr);
var jsonText = jsonDoc.RootElement.GetRawText();
```

The parsing of the configuration string must be performed within the `UseConfig` method of your handler. See an example in the [Custom Handler](#Custom-Handlers) section.

**Default Type**

When the handler configuration does not define the `type` (of the middleware class) it is assumed to be the [Default Handler](#DefaultHandler), which type name is ``Primavera.Hydrogen.Pipeboxes.Handlers.DefaultHandler`1`` corresponding to the generic type name of `DefaultHandler<>`. 

> A back tick `` ` `` character is escaped as `\u0060` in JSON.

### Pipebox Context

The `PipeboxContext<T>` provides the implementation of `IPipeboxContext<T>`, which is the data context that is processed along the pipeline execution.

See the [IPipeboxContext<T>][REF_PHPA] for more information about the interface members.

### Pipebox State

The `PipeboxState` provides the implementation of the state of execution for the `Pipebox<T>` and it has the following values that is managed internally by a state machine:
* **New** - When the pipebox is created.
* **Ready** - When the pipebox is configured.
* **Running** - When the pipebox is executing.
* **Failed** - When the pipebox execution failed with an internal error.
* **Canceled** - When the pipebox execution failed due to the cancellation token requested.
* **Completed** - When the pipebox has executed with success.

### Pipebox Exceptions

When the pipebox is running with the `ExecuteAsync` method, any internal error is shielded within a `PipeboxException` that contains the inner exception that raised the error and the corresponding error code (if the exception was properly handled). 

The error codes are:

* **PipelineError** - Failed to execute the pipeline due to an internal error.
* **HandlerError** - Failed to execute the handler due to an internal error.
* **HandlerNotFound** - Failed to get the specified handler type from the service provider.
* **PipelineNotFound** - Failed to get the specified pipeline identifier from the configuration.
* **PipelineIdDuplicates** - Failed to validate the configuration due to duplicate pipeline identifiers.
* **InvalidServiceProvider** - Invalid or null service provider.
* **InvalidConfiguration** - Invalid or null pipebox configuration.
* **InvalidPipelineConfiguration** - Invalid or null pipeline configuration (or pipeline identifier not found).
* **InvalidStateCondition** - Failed to execute the pipeline due to an unexpected state transition.

### Handler Configuration

The `HandlerConfig` provides the implementation of `<TConfig>`, which is the handler configuration.

## Built-in Handlers

This implementation also provides a set of built-in handlers useful for the most common scenarios. You can use them as is, or inherit from the base class to add the behavior of your solution.

#### HandlerBase

Is the base class that implements `IPipelineHandler<PipeboxContext<T>, HandlerConfig>` and provides the common behavior for all handlers. You should inherit from this class to build a custom handler.

See the [IPipelineHandler<TContext, TConfig>][REF_PHPA] for more information about the interface members.

#### DefaultHandler

Is the default implementation of `IPipelineHandler<PipeboxContext<T>, HandlerConfig>` and provides useful features for debugging and prototyping. 

When the `Pipebox<T>` is executing, any handler which type was not specified will be assumed to be the `DefaultHandler<T>`.

**Parameters**

Parameter | Value | Description
:--- | :--- | :---
action | error | Throws an exception. Useful to test the pipeline behavior when an exception occurs.

**Configuration**

In the following example, pipeline `p1` will throw an exception when executing the handler `h2`.

```json
{
  "version": "1.0",
  "pipelines": [
    {
      "id": "p1",
      "handlers": [
        {
          "id": "h1",
          "type": "<custom-handler>"
        },        
        {
          "id": "h2",
          "configStr": "action=error;"
        },
        {
          "id": "h3",
          "type": "<custom-handler>"
        }        
      ]
    }
  ]
}
```

#### HttpHandler

The `HttpHandler<T>` is the base class that implements the basics of HTTP requests. Use it as is, or inherit from this class, if you need to make HTTP requests.

**Configuration**

The parameters that can be configured, are the ones that follow:

|Parameter | Value | Description |
| :--------|:------| :-----------|
|credentialsrequired| true, false |If, to perform the Http request, is necessary to have credentials this option should be defined has 'true'.|
|applicationscopes | ******|The application scopes that will be used to get the client credentials token.|
|authorityserveruri  | ****** |The address to get the client credentials token.|
|clientid| ******|The client id to get the client credentials token.|
|clientsecret  | ****** |The client secret to get the client credentials token.|
|endpoint| The request endpoint.|
|httpmethod  | post, put, get, delete |The HTTP request method.|
|retryonfailure  |true, false| This option should be defined has 'true' if you want to activate the retry on failure.|
|retryattempts| n|The number of retry attempts.|
|minbackoff  | 0|Minimum exponential backoff value. |
|maxbackoff| 30|Maximum exponential backoff value.|
|deltabackoff  |2| Delta exponential backoff value.|

The handler should be configured, in the json configuration file, as follow:

```json
{
    "id": "h1",
    "order": "1",
    "tag": "My frist handler",
    "type": "Primavera.Hydrogen.Samples.Pbx.MyCustomHttpHandler,Primavera.Hydrogen.Samples.Pbx",
    "configStr": "httpmethod=post; endpoint=https://httpbin.org/post;",
    "active": "True"
}
```

The following code is an example of a custom http handler which performs a post with the specified context.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using Primavera.Hydrogen.Configuration;
using Primavera.Hydrogen.Pipeboxes;
using Primavera.Hydrogen.Pipeboxes.Config;
using Primavera.Hydrogen.Pipeboxes.Context;
using Primavera.Hydrogen.Pipeboxes.Handlers;
using Primavera.Hydrogen.Pipeboxes.Handlers.Http;

namespace Primavera.Hydrogen.Samples.Pbx
{

  /// <summary>
  /// The CustomHttpHandler is responsible for performing post and get operations.
  /// </summary>
  /// <seealso cref="HttpHandler{T}"/>
  public class MyCustomHttpHandler : HttpHandler<string>
  {
      /// <inheritdoc />
      public override string SetEndpoint()
      {
          return this.Configuration.Endpoint;
      }

      /// <inheritdoc />
      public override Task<ByteArrayContent> SetPostObject()
      {
          string contentType = "application/json";

          byte[] buffer = System.Text.Encoding.UTF8.GetBytes(this.Data);

          ByteArrayContent byteContent = new ByteArrayContent(buffer);

          byteContent.Headers.ContentType = new MediaTypeHeaderValue(contentType);

          return Task.FromResult(byteContent);
      }

      /// <inheritdoc />
      public override Task SaveResponse(HttpResponseMessage responseMessage)
      {
          // add code to perform operations with the response message.
      }

      /// <inheritdoc />
      public override Task ExecuteAsync(PipeboxContext<string> context, CancellationToken cancellationToken = default)
      {
          return base.ExecuteAsync(context, cancellationToken);
      }
  }
}
```

Notice the following:
|Method | Description |
| :--------| :-----------|
| `SetEndpoint()`| Is where the HTTP request endpoint is defined (since that endpoint is specified in the handler configuration, it is available from the `Configuration` property).|
| `SetPostObject()`| Is where the object that should be posted is created.|
| `SaveResponse()` | Is where the response received can be processed.|

You will need to add this new handler to the service collection.

Add the following to `AddAdditionalServices()`:

```csharp
services.AddTransient<MyCustomHttpHandler>();
```


## Custom Handlers

You can  build your own handlers to run a pipeline with the business logic of your solution. The following is an example of a custom handler which applies an upper/lower case on the context string depending on the action specified in the configuration string:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using Primavera.Hydrogen.Configuration;
using Primavera.Hydrogen.Pipeboxes;
using Primavera.Hydrogen.Pipeboxes.Config;
using Primavera.Hydrogen.Pipeboxes.Context;
using Primavera.Hydrogen.Pipeboxes.Handlers;

namespace Primavera.Hydrogen.Samples.Pbx
{
    /// <summary>
    /// My handler implementation, which applies an upper/lower case of the context string
    /// depending on the action specified in the configuration string.
    /// </summary>
    /// <seealso cref="Primavera.Hydrogen.Pipeboxes.Handlers.HandlerBase{T}" />
    public class MyHandler : HandlerBase<string>
    {
        #region Protected Properties

        /// <summary>
        /// Gets or sets the configuration string.
        /// </summary>
        /// <value>
        /// The configuration string.
        /// </value>
        protected ConfigString ConfigStr { get; set; }

        #endregion

        #region IPipelineHandler Members

        /// <inheritdoc />
        public override void Execute(PipeboxContext<string> context)
        {
            this.ExecuteAsync(context).GetAwaiter().GetResult();
        }

        /// <inheritdoc />
        public override Task ExecuteAsync(PipeboxContext<string> context, CancellationToken cancellationToken = default)
        {
            // Validate parameters

            SmartGuard.NotNull(() => context, context);

            // Cancellation token

            if (cancellationToken.IsCancellationRequested)
            {
                return Task.FromCanceled(cancellationToken); // Exit here
            }

            // Execute the action that was requested in the configuration string 

            var action = this.ConfigStr.GetValueOrDefault<string>("action");

            if (!string.IsNullOrEmpty(action))
            {
                // Upper action

                if (action.Equals("upper", StringComparison.OrdinalIgnoreCase))
                {
                    context.Data = context.Data.ToUpperInvariant();
                }

                // Lower action

                if (action.Equals("lower", StringComparison.OrdinalIgnoreCase))
                {
                    context.Data = context.Data.ToLowerInvariant();
                }
            }

            // Completed task

            return Task.CompletedTask;
        }

        #endregion

        #region Protected Methods | Overrides

        /// <inheritdoc />
        public override IPipelineHandler<PipeboxContext<string>, HandlerConfig> UseConfig(HandlerConfig config)
        {
            // Set the handler configuration

            base.UseConfig(config);

            // Set the handler configuration string (with case-insensitive keys)

            this.ConfigStr = new ConfigString(this.Config.ConfigStr, StringComparer.OrdinalIgnoreCase);

            // Fluent return

            return this;
        }

        #endregion
    }
}
```

Now, you need to build a configuration that uses your custom handler:

```json
{
  "version": "1.0",
  "pipelines": [
    {
      "id": "p1",
      "handlers": [
        {
          "id": "h1",
          "type": "Primavera.Hydrogen.Samples.Pbx.MyHandler, Primavera.Hydrogen.Samples",
          "configStr": "action=upper;"
        }
      ]
    }
  ]
}
```

In order to get the type name of your handler, just write the following in the immediate window of your IDE (debug):

```csharp
?typeof(MyHandler).ShortAssemblyQualifiedName()
"Primavera.Hydrogen.Samples.Pbx.MyHandler, Primavera.Hydrogen.Samples"
```

And finally, to run the pipeline:

```csharp
/// <summary>
/// Code samples for the <see cref="Pipebox{T}"/>.
/// </summary>
public static void Sample2()
{
    // Arrange the pipebox configuration

    string json = File.ReadAllText(@".\PipeboxConfig02.json");
    PipeboxConfig config = PipeboxConfig.Create(json);

    // Arrange the service provider registering the handler

    ServiceCollection services = new ServiceCollection();
    services.AddTransient(typeof(MyHandler));
    ServiceProvider provider = services.BuildServiceProvider();

    // Arrange the pipebox context with your data

    string data = "Hello";
    var context = new PipeboxContext<string>(data);

    // Arrange the pipebox to use the specified provider,
    // configuration and pipeline "p1"

    using Pipebox<string> pipebox = new Pipebox<string>();

    pipebox
        .UseProvider(provider)
        .UseConfig(config)
        .UsePipeline("p1");

    // Run the pipeline "p1"

    pipebox.ExecuteAsync(context).GetAwaiter().GetResult();

    // Get the state of the pipebox execution

    PipeboxState state = pipebox.CurrentState;

    // Get the result of context data

    string result = context.Data; 
    
    // Output: "HELLO"
}
```