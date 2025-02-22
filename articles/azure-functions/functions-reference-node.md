---
title: JavaScript developer reference for Azure Functions 
description: Understand how to develop functions by using JavaScript.

ms.assetid: 45dedd78-3ff9-411f-bb4b-16d29a11384c
ms.topic: conceptual
ms.date: 02/24/2022
ms.devlang: javascript
ms.custom: devx-track-js, vscode-azure-extension-update-not-needed
zone_pivot_groups: functions-nodejs-model
---

# Azure Functions JavaScript developer guide

This guide is an introduction to developing Azure Functions using JavaScript or TypeScript. The article assumes that you have already read the [Azure Functions developer guide](functions-reference.md).

> [!IMPORTANT]
> The content of this article changes based on your choice of the Node.js programming model in the selector at the top of this page. The version you choose should match the version of the [`@azure/functions`](https://www.npmjs.com/package/@azure/functions) npm package you are using in your app. If you do not have that package listed in your `package.json`, the default is v3. Learn more about the differences between v3 and v4 in the [upgrade guide](./functions-node-upgrade-v4.md).

As a JavaScript developer, you might also be interested in one of the following articles:

| Getting started | Concepts| Guided learning |
|---|---|---|
| <ul><li>[Node.js function using Visual Studio Code](./create-first-function-vs-code-node.md)</li><li>[Node.js function with terminal/command prompt](./create-first-function-cli-node.md)</li><li>[Node.js function using the Azure portal](functions-create-function-app-portal.md)</li></ul> | <ul><li>[Developer guide](functions-reference.md)</li><li>[Hosting options](functions-scale.md)</li><li>[TypeScript functions](#typescript)</li><li>[Performance&nbsp; considerations](functions-best-practices.md)</li></ul> | <ul><li>[Create serverless applications](/training/paths/create-serverless-applications/)</li><li>[Refactor Node.js and Express APIs to Serverless APIs](/training/modules/shift-nodejs-express-apis-serverless/)</li></ul> |

[!INCLUDE [Programming Model Considerations](../../includes/functions-nodejs-model-considerations.md)]

## Supported versions

The following table shows each version of the Node.js programming model along with its supported versions of the Azure Functions runtime and Node.js.

| [Programming Model Version](https://www.npmjs.com/package/@azure/functions?activeTab=versions) | Support Level | [Functions Runtime Version](./functions-versions.md) | [Node.js Version](https://github.com/nodejs/release#release-schedule) | Description |
| ---- | ---- | --- | --- | --- |
| 4.x | Preview | 4.16+ | 18.x | Supports a flexible file structure and code-centric approach to triggers and bindings. |
| 3.x | GA | 4.x | 18.x, 16.x, 14.x | Requires a specific file structure with your triggers and bindings declared in a "function.json" file |
| 2.x | GA (EOL) | 3.x | 14.x, 12.x, 10.x | Reached end of life (EOL) on December 13, 2022. See [Functions Versions](./functions-versions.md) for more info. |
| 1.x | GA (EOL) | 2.x | 10.x, 8.x | Reached end of life (EOL) on December 13, 2022. See [Functions Versions](./functions-versions.md) for more info. |

## Folder structure

::: zone pivot="nodejs-model-v3"

The required folder structure for a JavaScript project looks like the following example:

```
<project_root>/
 | - .vscode/
 | - node_modules/
 | - myFirstFunction/
 | | - index.js
 | | - function.json
 | - mySecondFunction/
 | | - index.js
 | | - function.json
 | - .funcignore
 | - host.json
 | - local.settings.json
 | - package.json
```

The main project folder, *<project_root>*, can contain the following files:

- **.vscode/**: (Optional) Contains the stored Visual Studio Code configuration. To learn more, see [Visual Studio Code settings](https://code.visualstudio.com/docs/getstarted/settings).
- **myFirstFunction/function.json**: Contains configuration for the function's trigger, inputs, and outputs. The name of the directory determines the name of your function.
- **myFirstFunction/index.js**: Stores your function code. To change this default file path, see [using scriptFile](#using-scriptfile).
- **.funcignore**: (Optional) Declares files that shouldn't get published to Azure. Usually, this file contains *.vscode/* to ignore your editor setting, *test/* to ignore test cases, and *local.settings.json* to prevent local app settings being published.
- **host.json**: Contains configuration options that affect all functions in a function app instance. This file does get published to Azure. Not all options are supported when running locally. To learn more, see [host.json](functions-host-json.md).
- **local.settings.json**: Used to store app settings and connection strings when it's running locally. This file doesn't get published to Azure. To learn more, see [local.settings.file](functions-develop-local.md#local-settings-file).
- **package.json**: Contains configuration options like a list of package dependencies, the main entrypoint, and scripts.

::: zone-end

::: zone pivot="nodejs-model-v4"

The recommended folder structure for a JavaScript project looks like the following example:

```
<project_root>/
 | - .vscode/
 | - node_modules/
 | - src/
 | | - functions/
 | | | - myFirstFunction.js
 | | | - mySecondFunction.js
 | - test/
 | | - functions/
 | | | - myFirstFunction.test.js
 | | | - mySecondFunction.test.js
 | - .funcignore
 | - host.json
 | - local.settings.json
 | - package.json
```

The main project folder, *<project_root>*, can contain the following files:

- **.vscode/**: (Optional) Contains the stored Visual Studio Code configuration. To learn more, see [Visual Studio Code settings](https://code.visualstudio.com/docs/getstarted/settings).
- **src/functions/**: The default location for all functions and their related triggers and bindings.
- **test/**: (Optional) Contains the test cases of your function app.
- **.funcignore**: (Optional) Declares files that shouldn't get published to Azure. Usually, this file contains *.vscode/* to ignore your editor setting, *test/* to ignore test cases, and *local.settings.json* to prevent local app settings being published.
- **host.json**: Contains configuration options that affect all functions in a function app instance. This file does get published to Azure. Not all options are supported when running locally. To learn more, see [host.json](functions-host-json.md).
- **local.settings.json**: Used to store app settings and connection strings when it's running locally. This file doesn't get published to Azure. To learn more, see [local.settings.file](functions-develop-local.md#local-settings-file).
- **package.json**: Contains configuration options like a list of package dependencies, the main entrypoint, and scripts.

::: zone-end

<a name="exporting-an-async-function"></a>
<a name="exporting-a-function"></a>

## Registering a function

::: zone pivot="nodejs-model-v3"

The v3 model registers a function based on the existence of two files. First, you need a `function.json` file located in a folder one level down from the root of your app. The name of the folder determines the function's name and the file contains configuration for your function's inputs/outputs. Second, you need a JavaScript file containing your code. By default, the model looks for an `index.js` file in the same folder as your `function.json`. Your code must export a function using [`module.exports`](https://nodejs.org/api/modules.html#modules_module_exports) (or [`exports`](https://nodejs.org/api/modules.html#modules_exports)). To customize the file location or export name of your function, see [configuring your function's entry point](functions-reference-node.md#configure-function-entry-point).

The function you export should always be declared as an [`async function`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) in the v3 model. You can export a synchronous function, but then you must call [`context.done()`](#contextdone) to signal that your function is completed, which is deprecated and not recommended.

Your function is passed an [invocation `context`](#invocation-context) as the first argument and your [inputs](#inputs) as the remaining arguments.

The following example is a simple function that logs that it was triggered and responds with `Hello, world!`:

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "authLevel": "anonymous",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

```javascript
module.exports = async function (context, request) {
    context.log('Http function was triggered.');
    context.res = { body: 'Hello, world!' };
};
```

::: zone-end

::: zone pivot="nodejs-model-v4"

The programming model loads your functions based on the `main` field in your `package.json`. This field can be set to a single file like `src/index.js` or a [glob pattern](https://wikipedia.org/wiki/Glob_(programming)) specifying multiple files like `src/functions/*.js`.

In order to register a function, you must import the `app` object from the `@azure/functions` npm module and call the method specific to your trigger type. The first argument when registering a function is the function name. The second argument is an `options` object specifying configuration for your trigger, your handler, and any other inputs or outputs. In some cases where trigger configuration isn't necessary, you can pass the handler directly as the second argument instead of an `options` object.

Registering a function can be done from any file in your project, as long as that file is loaded (directly or indirectly) based on the `main` field in your `package.json` file. The function should be registered at a global scope because you can't register functions once executions have started.

The following example is a simple function that logs that it was triggered and responds with `Hello, world!`:

```javascript
const { app } = require('@azure/functions');

app.http('httpTrigger1', {
    methods: ['POST', 'GET'],
    handler: async (request, context) => {
        context.log('Http function was triggered.');
        return { body: 'Hello, world!' };
    }
});
```

::: zone-end

<a name="bindings"></a>

## Inputs and outputs

::: zone pivot="nodejs-model-v3"

Your function is required to have exactly one primary input called the trigger. It may also have secondary inputs and/or outputs. Inputs and outputs are configured in your `function.json` files and are also referred to as [bindings](./functions-triggers-bindings.md).

### Inputs

Inputs are bindings with `direction` set to `in`. The main difference between a trigger and a secondary input is that the `type` for a trigger ends in `Trigger`, for example type [`blobTrigger`](./functions-bindings-storage-blob-trigger.md) vs type [`blob`](./functions-bindings-storage-blob-input.md). Most functions only use a trigger, and not many secondary input types are supported.

Inputs can be accessed in several ways:

- **_[Recommended]_ As arguments passed to your function:** Use the arguments in the same order that they're defined in `function.json`. The `name` property defined in `function.json` doesn't need to match the name of your argument, although it's recommended for the sake of organization.

    ```javascript
    module.exports = async function (context, myTrigger, myInput, myOtherInput) { ... };
    ```

- **As properties of [`context.bindings`](#contextbindings):** Use the key matching the `name` property defined in `function.json`.

    ```javascript
    module.exports = async function (context) { 
        context.log("This is myTrigger: " + context.bindings.myTrigger);
        context.log("This is myInput: " + context.bindings.myInput);
        context.log("This is myOtherInput: " + context.bindings.myOtherInput);
    };
    ```

<a name="returning-from-the-function"></a>

### Outputs

Outputs are bindings with `direction` set to `out` and can be set in several ways:

- **_[Recommended for single output]_ Return the value directly:** If you're using an async function, you can return the value directly. You must change the `name` property of the output binding to `$return` in `function.json` like in the following example:

    ```json
    {
        "name": "$return",
        "type": "http",
        "direction": "out"
    }
    ```

    ```javascript
    module.exports = async function (context, request) {
        return {
            body: "Hello, world!"
        };
    }
    ```

- **_[Recommended for multiple outputs]_ Return an object containing all outputs:** If you're using an async function, you can return an object with a property matching the name of each binding in your `function.json`. The following example uses output bindings named "httpResponse" and "queueOutput":

    ```json
    {
        "name": "httpResponse",
        "type": "http",
        "direction": "out"
    },
    {
        "name": "queueOutput",
        "type": "queue",
        "direction": "out",
        "queueName": "helloworldqueue",
        "connection": "storage_APPSETTING"
    }
    ```

    ```javascript
    module.exports = async function (context, request) {
        let message = 'Hello, world!';
        return {
            httpResponse: {
                body: message
            },
            queueOutput: message
        };
    };
    ```

- **Set values on `context.bindings`:** If you're not using an async function or you don't want to use the previous options, you can set values directly on `context.bindings`, where the key matches the name of the binding. The following example uses output bindings named "httpResponse" and "queueOutput":

    ```json
    {
        "name": "httpResponse",
        "type": "http",
        "direction": "out"
    },
    {
        "name": "queueOutput",
        "type": "queue",
        "direction": "out",
        "queueName": "helloworldqueue",
        "connection": "storage_APPSETTING"
    }
    ```

    ```javascript
    module.exports = async function (context, request) {
        let message = 'Hello, world!';
        context.bindings.httpResponse = {
            body: message
        };
        context.bindings.queueOutput = message;
    };
    ```

### Bindings data type

You can use the `dataType` property on an input binding to change the type of your input, however it has some limitations:
- In Node.js, only `string` and `binary` are supported (`stream` isn't)
- For HTTP inputs, the `dataType` property is ignored. Instead, use properties on the `request` object to get the body in your desired format. For more information, see [HTTP request](#http-request).

In the following example of a [storage queue trigger](./functions-bindings-storage-queue-trigger.md), the default type of `myQueueItem` is a `string`, but if you set `dataType` to `binary`, the type changes to a Node.js `Buffer`.

```json
{
    "name": "myQueueItem",
    "type": "queueTrigger",
    "direction": "in",
    "queueName": "helloworldqueue",
    "connection": "storage_APPSETTING",
    "dataType": "binary"
}
```

```javascript
const { Buffer } = require('node:buffer');

module.exports = async function (context, myQueueItem) {
    if (typeof myQueueItem === 'string') {
        context.log('myQueueItem is a string');
    } else if (Buffer.isBuffer(myQueueItem)) {
        context.log('myQueueItem is a buffer');
    }
};
```

::: zone-end

::: zone pivot="nodejs-model-v4"

Your function is required to have exactly one primary input called the trigger. It may also have secondary inputs, a primary output called the return output, and/or secondary outputs. Inputs and outputs are also referred to as [bindings](./functions-triggers-bindings.md) outside the context of the Node.js programming model. Before v4 of the model, these bindings were configured in `function.json` files.

### Trigger input

The trigger is the only required input or output. For most trigger types, you register a function by using a method on the `app` object named after the trigger type. You can specify configuration specific to the trigger directly on the `options` argument. For example, an HTTP trigger allows you to specify a route. During execution, the value corresponding to this trigger is passed in as the first argument to your handler.

```javascript
const { app } = require('@azure/functions');

app.http('helloWorld1', {
    route: 'hello/world',
    handler: async (request, ...) => {
        ...
    }
});
```

### Return output

The return output is optional, and in some cases configured by default. For example, an HTTP trigger registered with `app.http` is configured to return an HTTP response output automatically. For most output types, you specify the return configuration on the `options` argument with the help of the `output` object exported from the `@azure/functions` module. During execution, you set this output by returning it from your handler.

The following example uses a [timer trigger](./functions-bindings-timer.md) and a [storage queue output](./functions-bindings-storage-queue-output.md):

```javascript
const { app, output } = require('@azure/functions');

app.timer('timerTrigger1', {
    ...
    return: output.storageQueue({
        connection: 'storage_APPSETTING',
        ...
    }),
    handler: () => {
        return { hello: 'world' }
    }
});
```

### Extra inputs and outputs

In addition to the trigger and return, you may specify extra inputs or outputs on the `options` argument when registering a function. The `input` and `output` objects exported from the `@azure/functions` module provide type-specific methods to help construct the configuration. During execution, you get or set the values with `context.extraInputs.get` or `context.extraOutputs.set`, passing in the original configuration object as the first argument.

The following example is a function triggered by a [storage queue](./functions-bindings-storage-queue-trigger.md), with an extra [storage blob input](./functions-bindings-storage-blob-input.md) that is copied to an extra [storage blob output](./functions-bindings-storage-blob-output.md). The queue message should be the name of a file and replaces `{queueTrigger}` as the blob name to be copied, with the help of a [binding expression](./functions-bindings-expressions-patterns.md).

```javascript
const { app, input, output } = require('@azure/functions');

const blobInput = input.storageBlob({
    connection: 'storage_APPSETTING',
    path: 'helloworld/{queueTrigger}',
});

const blobOutput = output.storageBlob({
    connection: 'storage_APPSETTING',
    path: 'helloworld/{queueTrigger}-copy',
});

app.storageQueue('copyBlob1', {
    queueName: 'copyblobqueue',
    connection: 'storage_APPSETTING',
    extraInputs: [blobInput],
    extraOutputs: [blobOutput],
    handler: (queueItem, context) => {
        const blobInputValue = context.extraInputs.get(blobInput);
        context.extraOutputs.set(blobOutput, blobInputValue);
    }
});
```

### Generic inputs and outputs

The `app`, `trigger`, `input`, and `output` objects exported by the `@azure/functions` module provide type-specific methods for most types. For all the types that aren't supported, a `generic` method has been provided to allow you to manually specify the configuration. The `generic` method can also be used if you want to change the default settings provided by a type-specific method.

The following example is a simple HTTP triggered function using generic methods instead of type-specific methods.

```javascript
const { app, output, trigger } = require('@azure/functions');

app.generic('helloWorld1', {
    trigger: trigger.generic({
        type: 'httpTrigger',
        methods: ['GET']
    }),
    return: output.generic({
        type: 'http'
    }),
    handler: async (request: HttpRequest, context: InvocationContext) => {
        context.log(`Http function processed request for url "${request.url}"`);

        return { body: `Hello, world!` };
    }
});
```

::: zone-end

<a name="context-object"></a>

## Invocation context

::: zone pivot="nodejs-model-v3"

Each invocation of your function is passed an invocation `context` object, used to read inputs, set outputs, write to logs, and read various metadata. In the v3 model, the context object is always the first argument passed to your handler.

The `context` object has the following properties:

| Property | Description |
| --- | --- |
| **`invocationId`** | The ID of the current function invocation. |
| **`executionContext`** | See [execution context](#contextexecutioncontext). |
| **`bindings`** | See [bindings](#contextbindings). |
| **`bindingData`** | Metadata about the trigger input for this invocation, not including the value itself. For example, an [event hub trigger](./functions-bindings-event-hubs-trigger.md) has an `enqueuedTimeUtc` property. |
| **`traceContext`** | The context for distributed tracing. For more information, see [`Trace Context`](https://www.w3.org/TR/trace-context/). |
| **`bindingDefinitions`** | The configuration of your inputs and outputs, as defined in `function.json`. |
| **`req`** | See [HTTP request](#http-request). |
| **`res`** | See [HTTP response](#http-response). |

### context.executionContext

The `context.executionContext` object has the following properties:

| Property | Description |
| --- | --- |
| **`invocationId`** | The ID of the current function invocation. |
| **`functionName`** | The name of the function that is being invoked. The name of the folder containing the `function.json` file determines the name of the function. |
| **`functionDirectory`** | The folder containing the `function.json` file. |
| **`retryContext`** | See [retry context](#contextexecutioncontextretrycontext). |

#### context.executionContext.retryContext

The `context.executionContext.retryContext` object has the following properties:

| Property | Description |
| --- | --- |
| **`retryCount`** | A number representing the current retry attempt. |
| **`maxRetryCount`** | Maximum number of times an execution is retried. A value of `-1` means to retry indefinitely. |
| **`exception`** | Exception that caused the retry. |

<a name="contextbindings-property"></a>

### context.bindings

The `context.bindings` object is used to read inputs or set outputs. The following example is a [storage queue trigger](./functions-bindings-storage-queue-trigger.md), which uses `context.bindings` to copy a [storage blob input](./functions-bindings-storage-blob-input.md) to a [storage blob output](./functions-bindings-storage-blob-output.md). The queue message's content replaces `{queueTrigger}` as the file name to be copied, with the help of a [binding expression](./functions-bindings-expressions-patterns.md).

```json
{
    "name": "myQueueItem",
    "type": "queueTrigger",
    "direction": "in",
    "connection": "storage_APPSETTING",
    "queueName": "helloworldqueue"
},
{
    "name": "myInput",
    "type": "blob",
    "direction": "in",
    "connection": "storage_APPSETTING",
    "path": "helloworld/{queueTrigger}"
},
{
    "name": "myOutput",
    "type": "blob",
    "direction": "out",
    "connection": "storage_APPSETTING",
    "path": "helloworld/{queueTrigger}-copy"
}
```

```javascript
module.exports = async function (context, myQueueItem) {
    const blobValue = context.bindings.myInput;
    context.bindings.myOutput = blobValue;
};
```

<a name="contextdone-method"></a>

### context.done

The `context.done` method is deprecated. Before async functions were supported, you would signal your function is done by calling `context.done()`:

```javascript
module.exports = function (context, request) {
    context.log("this pattern is now deprecated");
    context.done();
};
```

Now, it's recommended to remove the call to `context.done()` and mark your function as async so that it returns a promise (even if you don't `await` anything). As soon as your function finishes (in other words, the returned promise resolves), the v3 model knows your function is done.

```javascript
module.exports = async function (context, request) {
    context.log("you don't need context.done or an awaited call")
};
```

::: zone-end

::: zone pivot="nodejs-model-v4"

Each invocation of your function is passed an invocation `context` object, with information about your invocation and methods used for logging. In the v4 model, the `context` object is typically the second argument passed to your handler.

The `InvocationContext` class has the following properties:

| Property | Description |
| --- | --- |
| **`invocationId`** | The ID of the current function invocation. |
| **`functionName`** | The name of the function. |
| **`extraInputs`** | Used to get the values of extra inputs. For more information, see [extra inputs and outputs](#extra-inputs-and-outputs). |
| **`extraOutputs`** | Used to set the values of extra outputs. For more information, see [extra inputs and outputs](#extra-inputs-and-outputs). |
| **`retryContext`** | See [retry context](#retry-context). |
| **`traceContext`** | The context for distributed tracing. For more information, see [`Trace Context`](https://www.w3.org/TR/trace-context/). |
| **`triggerMetadata`** | Metadata about the trigger input for this invocation, not including the value itself. For example, an [event hub trigger](./functions-bindings-event-hubs-trigger.md) has an `enqueuedTimeUtc` property. |
| **`options`** | The options used when registering the function, after they've been validated and with defaults explicitly specified. |

### Retry context

The `retryContext` object has the following properties:

| Property | Description |
| --- | --- |
| **`retryCount`** | A number representing the current retry attempt. |
| **`maxRetryCount`** | Maximum number of times an execution is retried. A value of `-1` means to retry indefinitely. |
| **`exception`** | Exception that caused the retry. |

For more information, see [`retry-policies`](./functions-bindings-errors.md#retry-policies).

::: zone-end

<a name="contextlog-method"></a>
<a name="write-trace-output-to-logs"></a>

## Logging

In Azure Functions, it's recommended to use `context.log()` to write logs. Azure Functions integrates with Azure Application Insights to better capture your function app logs. Application Insights, part of Azure Monitor, provides facilities for collection, visual rendering, and analysis of both application logs and your trace outputs. To learn more, see [monitoring Azure Functions](functions-monitoring.md).

> [!NOTE]  
> If you use the alternative Node.js `console.log` method, those logs are tracked at the app-level and will *not* be associated with any specific function. It is *highly recommended* to use `context` for logging instead of `console` so that all logs are associated with a specific function.

The following example writes a log at the default "information" level, including the invocation ID:

```javascript
context.log(`Something has happened. Invocation ID: "${context.invocationId}"`); 
```

<a name="trace-levels"></a>

### Log levels

In addition to the default `context.log` method, the following methods are available that let you write logs at specific levels:

::: zone pivot="nodejs-model-v3"

| Method                      | Description                                    |
| --------------------------- | ---------------------------------------------- |
| **`context.log.error()`**   | Writes an error-level event to the logs.       |
| **`context.log.warn()`**    | Writes a warning-level event to the logs.      |
| **`context.log.info()`**    | Writes an information-level event to the logs. |
| **`context.log.verbose()`** | Writes a trace-level event to the logs.        |

::: zone-end

::: zone pivot="nodejs-model-v4"

| Method                | Description                                    |
| --------------------- | ---------------------------------------------- |
| **`context.trace()`** | Writes a trace-level event to the logs.        |
| **`context.debug()`** | Writes a debug-level event to the logs.        |
| **`context.info()`**  | Writes an information-level event to the logs. |
| **`context.warn()`**  | Writes a warning-level event to the logs.      |
| **`context.error()`** | Writes an error-level event to the logs.       |

::: zone-end

<a name="configure-the-trace-level-for-logging"></a>

### Configure log level

Azure Functions lets you define the threshold level to be used when tracking and viewing logs. To set the threshold, use the `logging.logLevel` property in the `host.json` file. This property lets you define a default level applied to all functions, or a threshold for each individual function. To learn more, see [How to configure monitoring for Azure Functions](configure-monitoring.md).

::: zone pivot="nodejs-model-v3"

## Track custom data

By default, Azure Functions writes output as traces to Application Insights. For more control, you can instead use the [Application Insights Node.js SDK](https://github.com/microsoft/applicationinsights-node.js) to send custom data to your Application Insights instance. 

```javascript
const appInsights = require("applicationinsights");
appInsights.setup();
const client = appInsights.defaultClient;

module.exports = async function (context, request) {
    // Use this with 'tagOverrides' to correlate custom logs to the parent function invocation.
    var operationIdOverride = {"ai.operation.id":context.traceContext.traceparent};

    client.trackEvent({name: "my custom event", tagOverrides:operationIdOverride, properties: {customProperty2: "custom property value"}});
    client.trackException({exception: new Error("handled exceptions can be logged with this method"), tagOverrides:operationIdOverride});
    client.trackMetric({name: "custom metric", value: 3, tagOverrides:operationIdOverride});
    client.trackTrace({message: "trace message", tagOverrides:operationIdOverride});
    client.trackDependency({target:"http://dbname", name:"select customers proc", data:"SELECT * FROM Customers", duration:231, resultCode:0, success: true, dependencyTypeName: "ZSQL", tagOverrides:operationIdOverride});
    client.trackRequest({name:"GET /customers", url:"http://myserver/customers", duration:309, resultCode:200, success:true, tagOverrides:operationIdOverride});
};
```

The `tagOverrides` parameter sets the `operation_Id` to the function's invocation ID. This setting enables you to correlate all of the automatically generated and custom logs for a given function invocation.

::: zone-end

<a name="http-triggers-and-bindings"></a>

## HTTP triggers

::: zone pivot="nodejs-model-v3"

HTTP and webhook triggers use request and response objects to represent HTTP messages.

::: zone-end

::: zone pivot="nodejs-model-v4"

HTTP and webhook triggers use `HttpRequest` and `HttpResponse` objects to represent HTTP messages. The classes represent a subset of the [fetch standard](https://developer.mozilla.org/docs/Web/API/fetch), using Node.js's [`undici`](https://undici.nodejs.org/) package.

::: zone-end

<a name="request-object"></a>
<a name="accessing-the-request-and-response"></a>

### HTTP Request

::: zone pivot="nodejs-model-v3"

The request can be accessed in several ways:

- **As the second argument to your function:**

    ```javascript
    module.exports = async function (context, request) {
        context.log(`Http function processed request for url "${request.url}"`);
    ```

- **From the `context.req` property:** 

    ```javascript
    module.exports = async function (context, request) {
        context.log(`Http function processed request for url "${context.req.url}"`);
    ```

- **From the named input bindings:** This option works the same as any non HTTP binding. The binding name in `function.json` must match the key on `context.bindings`, or "request1" in the following example: 

    ```json
    {
        "name": "request1",
        "type": "httpTrigger",
        "direction": "in",
        "authLevel": "anonymous",
        "methods": [
            "get",
            "post"
        ]
    }
    ```
    ```javascript
    module.exports = async function (context, request) {
        context.log(`Http function processed request for url "${context.bindings.request1.url}"`);
    ```

The `HttpRequest` object has the following properties:

| Property         | Type                     | Description |
| ---------------- | ------------------------ | ----------- |
| **`method`**     | `string` | HTTP request method used to invoke this function. |
| **`url`**        | `string` | Request URL. |
| **`headers`**    | `Record<string, string>` | HTTP request headers. This object is case sensitive. It's recommended to use `request.getHeader('header-name')` instead, which is case insensitive. |
| **`query`**      | `Record<string, string>` | Query string parameter keys and values from the URL. |
| **`params`**     | `Record<string, string>` | Route parameter keys and values. |
| **`user`**       | `HttpRequestUser | null` | Object representing logged-in user, either through Functions authentication, SWA Authentication, or null when no such user is logged in. |
| **`body`**       | `Buffer | string | any` | If the media type is "application/octet-stream" or "multipart/*", `body` is a Buffer. If the value is a JSON parse-able string, `body` is the parsed object. Otherwise, `body` is a string. |
| **`rawBody`**    | `Buffer | string` | If the media type is "application/octet-stream" or "multipart/*", `rawBody` is a Buffer. Otherwise, `rawBody` is a string. The only difference between `body` and `rawBody` is that `rawBody` doesn't JSON parse a string body. |
| **`bufferBody`** | `Buffer` | The body as a buffer. |

::: zone-end

::: zone pivot="nodejs-model-v4"

The request can be accessed as the first argument to your handler for an HTTP triggered function.

```javascript
async (request, context) => {
    context.log(`Http function processed request for url "${request.url}"`);
```

The `HttpRequest` object has the following properties:

| Property       | Type                     | Description |
| -------------- | ------------------------ | ----------- |
| **`method`**   | `string` | HTTP request method used to invoke this function. |
| **`url`**      | `string` | Request URL. |
| **`headers`**  | [`Headers`](https://developer.mozilla.org/docs/Web/API/Headers) | HTTP request headers. |
| **`query`**    | [`URLSearchParams`](https://developer.mozilla.org/docs/Web/API/URLSearchParams) | Query string parameter keys and values from the URL. |
| **`params`**   | `Record<string, string>` | Route parameter keys and values. |
| **`user`**     | `HttpRequestUser | null` | Object representing logged-in user, either through Functions authentication, SWA Authentication, or null when no such user is logged in. |
| **`body`**     | [`ReadableStream | null`](https://developer.mozilla.org/docs/Web/API/ReadableStream) | Body as a readable stream. |
| **`bodyUsed`** | `boolean` | A boolean indicating if the body has been read from already. |

In order to access a request or response's body, the following methods can be used:

| Method              | Return Type |
| ------------------- | ----------- |
| **`arrayBuffer()`** | [`Promise<ArrayBuffer>`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) |
| **`blob()`**        | [`Promise<Blob>`](https://developer.mozilla.org/docs/Web/API/Blob) |
| **`formData()`**    | [`Promise<FormData>`](https://developer.mozilla.org/docs/Web/API/FormData) |
| **`json()`**        | `Promise<unknown>` |
| **`text()`**        | `Promise<string>` |

> [!NOTE]  
> The body functions can be run only once; subsequent calls will resolve with empty strings/ArrayBuffers.

::: zone-end

<a name="response-object"></a>

### HTTP Response

::: zone pivot="nodejs-model-v3"

The response can be set in several ways:

- **Set the `context.res` property:** 

    ```javascript
    module.exports = async function (context, request) {
        context.res = { body: `Hello, world!` };
    ```

- **Return the response:** If your function is async and you set the binding name to `$return` in your `function.json`, you can return the response directly instead of setting it on `context`.

    ```json
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
    ```

    ```javascript
    module.exports = async function (context, request) {
        return { body: `Hello, world!` };
    ```

- **Set the named output binding:** This option works the same as any non HTTP binding. The binding name in `function.json` must match the key on `context.bindings`, or "response1" in the following example:

    ```json
    {
        "type": "http",
        "direction": "out",
        "name": "response1"
    }
    ```

    ```javascript
    module.exports = async function (context, request) {
        context.bindings.response1 = { body: `Hello, world!` };
    ```

- **Call `context.res.send()`:** This option is deprecated. It implicitly calls `context.done()` and can't be used in an async function.

    ```javascript
    module.exports = function (context, request) {
        context.res.send(`Hello, world!`);
    ```

If you create a new object when setting the response, that object must match the `HttpResponseSimple` interface, which has the following properties:

| Property       | Type | Description |
| -------------- | ---- | ----------- |
| **`headers`**  | `Record<string, string>` (optional) | HTTP response headers. |
| **`cookies`**  | `Cookie[]` (optional) | HTTP response cookies. |
| **`body`**     | `any` (optional) | HTTP response body. |
| **`statusCode`**   | `number` (optional) | HTTP response status code. If not set, defaults to `200`. |
| **`status`**   | `number` (optional) | The same as `statusCode`. This property is ignored if `statusCode` is set. |

You can also modify the `context.res` object without overwriting it. The default `context.res` object uses the `HttpResponseFull` interface, which supports the following methods in addition to the `HttpResponseSimple` properties:

| Method               | Description |
| -------------------- | ----------- |
| **`status()`**       | Sets the status. |
| **`setHeader()`**    | Sets a header field. NOTE: `res.set()` and `res.header()` are also supported and do the same thing. |
| **`getHeader()`**    | Get a header field. NOTE: `res.get()` is also supported and does the same thing. |
| **`removeHeader()`** | Removes a header. |
| **`type()`**         | Sets the "content-type" header. |
| **`send()`**         | This method is deprecated. It sets the body and calls `context.done()` to indicate a sync function is finished. NOTE: `res.end()` is also supported and does the same thing. |
| **`sendStatus()`**   | This method is deprecated. It sets the status code and calls `context.done()` to indicate a sync function is finished. |
| **`json()`**         | This method is deprecated. It sets the "content-type" to "application/json", sets the body, and calls `context.done()` to indicate a sync function is finished. |

::: zone-end

::: zone pivot="nodejs-model-v4"

The response can be set in several ways:

- **As a simple interface with type `HttpResponseInit`:** This option is the most concise way of returning responses.

    ```javascript
    return { body: `Hello, world!` };
    ```

    The `HttpResponseInit` interface has the following properties:

    | Property       | Type | Description |
    | -------------- | ---- | ----------- |
    | **`body`**     | `BodyInit` (optional) | HTTP response body as one of [`ArrayBuffer`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer), [`AsyncIterable<Uint8Array>`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array), [`Blob`](https://developer.mozilla.org/docs/Web/API/Blob), [`FormData`](https://developer.mozilla.org/docs/Web/API/FormData), [`Iterable<Uint8Array>`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array), [`NodeJS.ArrayBufferView`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer), [`URLSearchParams`](https://developer.mozilla.org/docs/Web/API/URLSearchParams), `null`, or `string`. |
    | **`jsonBody`** | `any` (optional) | A JSON-serializable HTTP Response body. If set, the `HttpResponseInit.body` property is ignored in favor of this property. |
    | **`status`**   | `number` (optional) | HTTP response status code. If not set, defaults to `200`. |
    | **`headers`**  | [`HeadersInit`](https://developer.mozilla.org/docs/Web/API/Headers) (optional) | HTTP response headers. |
    | **`cookies`**  | `Cookie[]` (optional) | HTTP response cookies. |

- **As a class with type `HttpResponse`:** This option provides helper methods for reading and modifying various parts of the response like the headers.

    ```javascript
    const response = new HttpResponse({ body: `Hello, world!` });
    response.headers.set('content-type', 'application/json');
    return response;
    ```

    The `HttpResponse` class accepts an optional `HttpResponseInit` as an argument to its constructor and has the following properties:
    
    | Property       | Type | Description |
    | -------------- | ---- | ----------- |
    | **`status`**   | `number` | HTTP response status code. |
    | **`headers`**  | [`Headers`](https://developer.mozilla.org/docs/Web/API/Headers) | HTTP response headers. |
    | **`cookies`**  | `Cookie[]` | HTTP response cookies. |
    | **`body`**     | [`ReadableStream | null`](https://developer.mozilla.org/docs/Web/API/ReadableStream) | Body as a readable stream. |
    | **`bodyUsed`** | `boolean` | A boolean indicating if the body has been read from already. |

::: zone-end

## Scaling and concurrency

By default, Azure Functions automatically monitors the load on your application and creates more host instances for Node.js as needed. Azure Functions uses built-in (not user configurable) thresholds for different trigger types to decide when to add instances, such as the age of messages and queue size for QueueTrigger. For more information, see [How the Consumption and Premium plans work](event-driven-scaling.md).

This scaling behavior is sufficient for many Node.js applications. For CPU-bound applications, you can improve performance further by using multiple language worker processes. You can increase the number of worker processes per host from the default of 1 up to a max of 10 by using the [FUNCTIONS_WORKER_PROCESS_COUNT](functions-app-settings.md#functions_worker_process_count) application setting. Azure Functions then tries to evenly distribute simultaneous function invocations across these workers. This behavior makes it less likely that a CPU-intensive function blocks other functions from running. The setting applies to each host that Azure Functions creates when scaling out your application to meet demand.

> [!WARNING]
> Use the `FUNCTIONS_WORKER_PROCESS_COUNT` setting with caution. Multiple processes running in the same instance can lead to unpredictable behavior and increase function load times. If you use this setting, it's *highly recommended* to offset these downsides by [running from a package file](./run-functions-from-deployment-package.md).

## Node version

You can see the current version that the runtime is using by logging `process.version` from any function. See [`supported versions`](#supported-versions) for a list of Node.js versions supported by each programming model.

### Setting the Node version

# [Windows](#tab/windows-setting-the-node-version)

For Windows function apps, target the version in Azure by setting the `WEBSITE_NODE_DEFAULT_VERSION` [app setting](functions-how-to-use-azure-function-app-settings.md#settings) to a supported LTS version, such as `~18`.

# [Linux](#tab/linux-setting-the-node-version)

For Linux function apps, run the following Azure CLI command to update the Node version.

```azurecli
az functionapp config set --linux-fx-version "node|18" --name "<MY_APP_NAME>" --resource-group "<MY_RESOURCE_GROUP_NAME>"
```

---

To learn more about Azure Functions runtime support policy, refer to this [article](./language-support-policy.md).

<a name="access-environment-variables-in-code"></a>

## Environment variables

Environment variables can be useful for operational secrets (connection strings, keys, endpoints, etc.) or environmental settings such as profiling variables. You can add environment variables in both your local and cloud environments and access them through `process.env` in your function code.

The following example logs the `WEBSITE_SITE_NAME` environment variable:

::: zone pivot="nodejs-model-v3"

```javascript
module.exports = async function (context) {
    context.log(`WEBSITE_SITE_NAME: ${process.env["WEBSITE_SITE_NAME"]}`);
}
```

::: zone-end

::: zone pivot="nodejs-model-v4"

```javascript
async function timerTrigger1(myTimer, context) {
    context.log(`WEBSITE_SITE_NAME: ${process.env["WEBSITE_SITE_NAME"]}`);
}
```

::: zone-end

### In local development environment

When you run locally, your functions project includes a [`local.settings.json` file](./functions-run-local.md), where you store your environment variables in the `Values` object. 

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "CUSTOM_ENV_VAR_1": "hello",
    "CUSTOM_ENV_VAR_2": "world"
  }
}
```

### In Azure cloud environment

When you run in Azure, the function app lets you set and use [Application settings](functions-app-settings.md), such as service connection strings, and exposes these settings as environment variables during execution. 

[!INCLUDE [Function app settings](../../includes/functions-app-settings.md)]

### Worker environment variables

There are several Functions environment variables specific to Node.js:

#### languageWorkers__node__arguments

This setting allows you to specify custom arguments when starting your Node.js process. It's most often used locally to start the worker in debug mode, but can also be used in Azure if you need custom arguments.

> [!WARNING]
> If possible, avoid using `languageWorkers__node__arguments` in Azure because it can have a negative effect on cold start times. Rather than using pre-warmed workers, the runtime has to start a new worker from scratch with your custom arguments.

#### logging__logLevel__Worker

This setting adjusts the default log level for Node.js-specific worker logs. By default, only warning or error logs are shown, but you can set it to `information` or `debug` to help diagnose issues with the Node.js worker. For more information, see [configuring log levels](./configure-monitoring.md#configure-log-levels).

## <a name="ecmascript-modules"></a>ECMAScript modules (preview)

> [!NOTE]
> As ECMAScript modules are currently a preview feature in Node.js 14 or higher in Azure Functions.

[ECMAScript modules](https://nodejs.org/docs/latest-v14.x/api/esm.html#esm_modules_ecmascript_modules) (ES modules) are the new official standard module system for Node.js. So far, the code samples in this article use the CommonJS syntax. When running Azure Functions in Node.js 14 or higher, you can choose to write your functions using ES modules syntax.

To use ES modules in a function, change its filename to use a `.mjs` extension. The following *index.mjs* file example is an HTTP triggered function that uses ES modules syntax to import the `uuid` library and return a value.

::: zone pivot="nodejs-model-v3"

```js
import { v4 as uuidv4 } from 'uuid';

async function httpTrigger1(context, request) {
    context.res.body = uuidv4();
};

export default httpTrigger1;
```

::: zone-end

::: zone pivot="nodejs-model-v4"

```js
import { v4 as uuidv4 } from 'uuid';

async function httpTrigger1(request, context) {
    context.res.body = uuidv4();
};
```

::: zone-end

::: zone pivot="nodejs-model-v3"

## Configure function entry point

The `function.json` properties `scriptFile` and `entryPoint` can be used to configure the location and name of your exported function. These properties can be important when your JavaScript is transpiled.

### Using `scriptFile`

By default, a JavaScript function is executed from `index.js`, a file that shares the same parent directory as its corresponding `function.json`.

`scriptFile` can be used to get a folder structure that looks like the following example:

```
<project_root>/
 | - node_modules/
 | - myFirstFunction/
 | | - function.json
 | - lib/
 | | - sayHello.js
 | - host.json
 | - package.json
```

The `function.json` for `myFirstFunction` should include a `scriptFile` property pointing to the file with the exported function to run.

```json
{
  "scriptFile": "../lib/sayHello.js",
  "bindings": [
    ...
  ]
}
```

### Using `entryPoint`

In the v3 model, a function must be exported using `module.exports` in order to be found and run. By default, the function that executes when triggered is the only export from that file, the export named `run`, or the export named `index`. The following example sets `entryPoint` in `function.json` to a custom value, "logHello":

```json
{
  "entryPoint": "logHello",
  "bindings": [
    ...
  ]
}
```

```javascript
async function logHello(context) { 
    context.log('Hello, world!'); 
}

module.exports = { logHello };
```

::: zone-end

## Local debugging

It's recommended to use VS Code for local debugging, which starts your Node.js process in debug mode automatically and attaches to the process for you. For more information, see [run the function locally](./create-first-function-vs-code-node.md#run-the-function-locally).

If you're using a different tool for debugging or want to start your Node.js process in debug mode manually, add `"languageWorkers__node__arguments": "--inspect"` under `Values` in your [local.settings.json](./functions-develop-local.md#local-settings-file). The `--inspect` argument tells Node.js to listen for a debug client, on port 9229 by default. For more information, see the [Node.js debugging guide](https://nodejs.org/en/docs/guides/debugging-getting-started).

## TypeScript

Both [Azure Functions for Visual Studio Code](./create-first-function-cli-typescript.md) and the [Azure Functions Core Tools](functions-run-local.md) let you create function apps using a template that supports TypeScript function app projects. The template generates `package.json` and `tsconfig.json` project files that make it easier to transpile, run, and publish JavaScript functions from TypeScript code with these tools.

A generated `.funcignore` file is used to indicate which files are excluded when a project is published to Azure.  

::: zone pivot="nodejs-model-v3"

TypeScript files (.ts) are transpiled into JavaScript files (.js) in the `dist` output directory. TypeScript templates use the [`scriptFile` parameter](#using-scriptfile) in `function.json` to indicate the location of the corresponding .js file in the `dist` folder. The setting `outDir` in your `tsconfig.json` file controls the output location. If you change this setting or the name of the folder, the runtime isn't able to find the code to run.

::: zone-end

The way that you locally develop and deploy from a TypeScript project depends on your development tool.

### Visual Studio Code

The [Azure Functions for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) extension lets you develop your functions using TypeScript. The Core Tools is a requirement of the Azure Functions extension.

To create a TypeScript function app in Visual Studio Code, choose `TypeScript` as your language when you create a function app.

When you press **F5** to run the app locally, transpilation is done before the host (func.exe) is initialized. 

When you deploy your function app to Azure using the **Deploy to function app...** button, the Azure Functions extension first generates a production-ready build of JavaScript files from the TypeScript source files.

### Azure Functions Core Tools

There are several ways in which a TypeScript project differs from a JavaScript project when using the Core Tools.

#### Create project

To create a TypeScript function app project using Core Tools, you must specify the TypeScript language option when you create your function app. You can create an app in one of the following ways:

::: zone pivot="nodejs-model-v3"

- Run the `func init` command, select `node` as your language stack, and then select `typescript`.
- Run the `func init --worker-runtime typescript` command.

::: zone-end

::: zone pivot="nodejs-model-v4"

- Run the `func init --model v4` command, select `node` as your language stack, and then select `typescript`.
- Run the `func init  --model v4 --worker-runtime typescript` command.

::: zone-end

#### Run local

To run your function app code locally using Core Tools, use the following commands instead of `func host start`: 

```command
npm install
npm start
```

The `npm start` command is equivalent to the following commands:

- `npm run build`
- `tsc`
- `func start`

#### Publish to Azure

Before you use the [`func azure functionapp publish`] command to deploy to Azure, you create a production-ready build of JavaScript files from the TypeScript source files. 

The following commands prepare and publish your TypeScript project using Core Tools: 

```command
npm run build 
func azure functionapp publish <APP_NAME>
```

In this command, replace `<APP_NAME>` with the name of your function app.

<a name="considerations-for-javascript-functions"></a>

## Recommendations

This section describes several impactful patterns for Node.js apps that we recommend you follow.

### Choose single-vCPU App Service plans

When you create a function app that uses the App Service plan, we recommend that you select a single-vCPU plan rather than a plan with multiple vCPUs. Today, Functions runs JavaScript functions more efficiently on single-vCPU VMs, and using larger VMs doesn't produce the expected performance improvements. When necessary, you can manually scale out by adding more single-vCPU VM instances, or you can enable autoscale. For more information, see [Scale instance count manually or automatically](../azure-monitor/autoscale/autoscale-get-started.md?toc=/azure/app-service/toc.json).

<a name="cold-start"></a>

### Run from a package file

When you develop Azure Functions in the serverless hosting model, cold starts are a reality. *Cold start* refers to the first time your function app starts after a period of inactivity, taking longer to start up. For JavaScript apps with large dependency trees in particular, cold start can be significant. To speed up the cold start process, [run your functions as a package file](run-functions-from-deployment-package.md) when possible. Many deployment methods use this model by default, but if you're experiencing large cold starts you should check to make sure you're running this way.

<a name="connection-limits"></a>

### Use a single static client

When you use a service-specific client in an Azure Functions application, don't create a new client with every function invocation because you can hit connection limits. Instead, create a single, static client in the global scope. For more information, see [managing connections in Azure Functions](manage-connections.md).

::: zone pivot="nodejs-model-v3"

### Use `async` and `await`

When writing Azure Functions in JavaScript, you should write code using the `async` and `await` keywords. Writing code using `async` and `await` instead of callbacks or `.then` and `.catch` with Promises helps avoid two common problems:
 - Throwing uncaught exceptions that [crash the Node.js process](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly), potentially affecting the execution of other functions.
 - Unexpected behavior, such as missing logs from `context.log`, caused by asynchronous calls that aren't properly awaited.

In the following example, the asynchronous method `fs.readFile` is invoked with an error-first callback function as its second parameter. This code causes both of the issues previously mentioned. An exception that isn't explicitly caught in the correct scope can crash the entire process (issue #1). Calling the deprecated `context.done()` method outside of the scope of the callback can signal the function is finished before the file is read (issue #2). In this example, calling `context.done()` too early results in missing log entries starting with `Data from file:`.

```javascript
// NOT RECOMMENDED PATTERN
const fs = require('fs');

module.exports = function (context) {
    fs.readFile('./hello.txt', (err, data) => {
        if (err) {
            context.log.error('ERROR', err);
            // BUG #1: This will result in an uncaught exception that crashes the entire process
            throw err;
        }
        context.log(`Data from file: ${data}`);
        // context.done() should be called here
    });
    // BUG #2: Data is not guaranteed to be read before the Azure Function's invocation ends
    context.done();
}
```

Use the `async` and `await` keywords to help avoid both of these issues. Most APIs in the Node.js ecosystem have been converted to support promises in some form. For example, starting in v14, Node.js provides an `fs/promises` API to replace the `fs` callback API.

In the following example, any unhandled exceptions thrown during the function execution only fail the individual invocation that raised the exception. The `await` keyword means that steps following `readFile` only execute after it's complete. With `async` and `await`, you also don't need to call the `context.done()` callback.

```javascript
// Recommended pattern
const fs = require('fs/promises');

module.exports = async function (context) {
    let data;
    try {
        data = await fs.readFile('./hello.txt');
    } catch (err) {
        context.log.error('ERROR', err);
        // This rethrown exception will be handled by the Functions Runtime and will only fail the individual invocation
        throw err;
    }
    context.log(`Data from file: ${data}`);
}
```

::: zone-end

## Next steps

For more information, see the following resources:

- [Best practices for Azure Functions](functions-best-practices.md)
- [Azure Functions developer reference](functions-reference.md)
- [Azure Functions triggers and bindings](functions-triggers-bindings.md)
