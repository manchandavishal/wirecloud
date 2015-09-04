FIWARE Application Mashup - Widget API Specification
====================================================
Date: 30th September 2015

This version:

    https://github.com/Wirecloud/wirecloud/tree/0.8.x/docs/widgetapi/

Previous version:

    https://github.com/Wirecloud/wirecloud/tree/0.7.x/docs/widgetapi/

Latest version:

    https://github.com/Wirecloud/wirecloud/tree/develop/docs/widgetapi/

The Application Mashup GE offers two separate APIs that cannot be combined
because of their different nature: The Widget API (the subject of this
document) is a JavaScript API, while the Application Mashup API is a
RESTful one. You can find the Application Mashup RESTful API in the
following link:

    http://docs.fiwareapplicationmashup.apiary.io/

The Widget API is a JavaScript API that allows deployed widgets in a Application
Mashup server to gain access to its functionalities. It does not make sense
to expose it as a RESTful API since it needs to be consumed by a widget in its
own local execution environment. Amongst other functionalities, this API allows
the widgets to gain access to remote resources. For example, in order to gain
access to a remote REST API or to resolve cross-domain problems, a widget needs
to use a proxy through the Widget API

## Editors

  + Álvaro Arranz, Universidad Politécnica de Madrid

## Acknowledgements

The editors would like to express their gratitude to the following people who actively contributed to this specification:
Aitor Magan and Francisco de la Vega

## Status

This is a work in progress and is changing on a daily basis. You can check the latest
available version on: https://github.com/Wirecloud/wirecloud/tree/develop.
Please send your comments to wirecloud@conwet.com

This specification is licensed under the [FIWARE Open Specification License (implicit patents license)](https://forge.fiware.org/plugins/mediawiki/wiki/fiware/index.php/FI-WARE_Open_Specification_Legal_Notice_%28implicit_patents_license%29).

## Copyright

* Copyright © 2012-2015 by Universidad Politécnica de Madrid

All rights reserved. No part of this publication may be reproduced, distributed, or
transmitted in any form or by any means, including photocopying, recording, or
other electronic or mechanical methods, without the prior written permission of the
publisher, except in the case of brief quotations embodied in critical reviews and
certain other noncommercial uses permitted by copyright law. For permission
requests, write to the publisher, addressed “Attention: Permissions Coordinator,” at
the address below.

## Widget API

### MashupPlatform.http

This module provides some methods for handling HTTP requests including support
for using the cross domain proxy.

Currently this module is composed of two methods:

- [`buildProxyURL`](#buildproxyurl-method)
- [`makeRequest`](#makerequest-method)

#### `buildProxyURL` method

This method builds a URL suitable for working around the cross-domain problem.
It is usually handled using the WireCloud proxy but it also can be handled using
the access control request headers if the browser has support for them. If all
the needed requirements are meet, this function will return a URL without using
the proxy.

```javascript
MashupPlatform.http.buildProxyURL(url, options)
```

* `url` is the target URL
* `options` is an object with request options (shown later)

**Example usage:**

```javascript
var internal_url = 'http://server.intranet.com/image/a.png';
var url = MashupPlatform.http.buildProxyURL(internal_url, {forceProxy: true});
var img = document.createElement('img');
img.src = url;
```

#### `makeRequest` method

Sends an HTTP request. This method internally calls the buildProxyURL method for
working around any possible problem related with the same-origin policy followed
by browser (allowing CORS requests).

```javascript
MashupPlatform.http.makeRequest(url, options)
```

* `url` is the URL to which to send the request
* `options` is an object with a list of request options (shown later)

**Example usage:**

```javascript
$('loading').show();
MashupPlatform.http.makeRequest('http://api.example.com', {
    method: "POST",
    postBody: JSON.stringify({key: value}),
    contentType: "application/json",
    onSuccess: function (response) {
        // Everything went ok
    },
    onFailure: function (response) {
        // Something went wrong
    },
    onComplete: function () {
        $('loading').hide();
    }
});
```

#### Request options: General options

- `contentType` (String): The Content-Type header for your request. If it is not
  provided, the content-type header will be extrapolated from the value of the
  `postBody` option (defaulting to `application/x-www-form-urlencoded` if the
  `postBody` value is a String). Specify this option if you want to force the
  value of the `Content-Type` header.
- `encoding` (String; default `UTF-8`): The encoding for the contents of your
  request. It is best left as-is, but should weird encoding issues arise, you
  may have to tweak this.
- `method` (String; default `POST`): The HTTP method to use for the request.
- `responseType` (String; default: ""): Can be set to change the response type.
  The valid values for this options are: "", "arraybuffer", "blob" and "text".
- `parameters` (Object): The parameters for the request, which will be encoded
  into the URL for a `GET` method, or into the request body when using the `PUT`
  and `POST` methods.
- `postBody` (`ArrayBufferView`, `Blob`, `Document`, `String`, `FormData`): The
  contents to use as request body (usually used in post and put requests, but
  can be used by any request). If it is not provided, the contents of the
  parameters option will be used instead. Finally, if there are not parameters,
  request will not have a body.
- `requestHeaders` (Object): A set of key-value pairs, with properties
  representing header names.
- `supportsAccessControl` (Boolean; default `false`): Indicates whether the
  target server supports the [Access
  Control](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
  headers, and thus, you want to make a request without using the cross-domain
  proxy if possible.
- `forceProxy` (Boolean; default `false`): Sends the request through the proxy
  regardless of the other options passed.
- `context` (Object; default `null`): The value to be passed as the this parameter
  to the callbacks. If context is `null` the `this` parameter of the callbacks
  is left intact.

#### Request options: Callback options

- `onSuccess`: Invoked when a request completes and its status code belongs in the 2xy family. This is skipped if a code-specific callback is defined, and happens before `onComplete`. Receives the response object as the first argument
- `onFailure`: Invoked when a request completes and its status code exists but is not in the 2xy family. This is skipped if a code-specific callback is defined, and happens before `onComplete`. Receives the response object as the first argument
- `onXYZ` (with XYZ representing any HTTP status code): Invoked just after the response is complete if the status code is the exact code used in the callback name. Prevents execution of onSuccess and `onFailure`. Happens before `onComplete`. Receives the response object as the first argument
- `onComplete`: Triggered at the very end of a request's life-cycle, after the
  request completes, status-specific callbacks are called, and possible
  automatic behaviours are processed. Guaranteed to run regardless of what
  happened during the request. Receives the response object as the first
  argument
- `onException`: Triggered whenever an exception arises running any of the
  `onXYZ`, `onSuccess`, `onFailure` and `onComplete` callbacks. Receives the
  request as the first argument, and the exception object as the second one

#### Response object

- `request` (Request): The request for the current response
- `status` (Number): The status of the response to the request. This is the HTTP
  result code
- `statusText` (String): The response string returned by the HTTP server. Unlike
  status, this includes the entire text of the response message
- `response` (ArrayBuffer, Blob, String): The response entity body according to
  responseType, as an `ArrayBuffer`, `Blob` or `String`. This is `null` if the
  request is not complete, was not successful or the responseType option of the
  requests was ""
- `responseText` (String): The response to the request as text, or `null` if the
  request was unsuccessful or the responseType option of the requests was
  different to ""


### MashupPlatform.log

This module contains the following constants:

* **ERROR:** Used for indicating an Error level
* **WARN:** Used for indicating a Warning level
* **INFO:** Used for indicating an Info level

Those constants can be used when calling to the `MashupPlatform.widget.log` and
`MashupPlatform.operator.log` methods.

### MashupPlatform.prefs

This module provides methods for managing the preferences defined on the
mashable application component description file (`config.xml` file).

Currently, this module provides three methods:

- [`get`](#get)
- [`registerCallback`](#registercallback-method)
- [`set`](#set-method)

and one exception:

`PreferenceDoesNotExistError`

#### `get` method

This method retrieves the value of a preference. The type of the value returned
by this method depends on the type of the preference.

```javascript
MashupPlatform.prefs.get(key)
```

- `key` is the name of the preference as defined on the `config.xml` file.

This method can raise the following exceptions:

- `MashupPlatform.prefs.PreferenceDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.prefs.get('boolean-pref'); // true or false
MashupPlatform.prefs.get('number-prefs'); // a number value
MashupPlatform.prefs.get('text-prefs');   // a string value
```

#### `registerCallback` method

This method registers a callback for listening for preference changes.

```javascript
MashupPlatform.prefs.registerCallback(callback)
```

- `callback` function to be called when a change in one or more preferences is
  detected. This callback will receive a key-value object with the changed
  preferences and their new values.

**Example usage:**

```javascript
MashupPlatform.prefs.registerCallback(function (new_values) {
    if ('some-pref' in new_values) {
        // some-pref has been changed
        // new_values['some-pref'] contains the new value
    }
});
```

#### `set` method

This method sets the value of a preference.

```javascript
MashupPlatform.prefs.set(key, value)
```

- `key` is the name of the preference as defined on the `config.xml` file.
- `value` is the new value for the preference. The acceptable values for this
  parameter depends on the type of the preference.

This method can raise the following exceptions:

- `MashupPlatform.prefs.PreferenceDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.prefs.set('boolean-pref', true);
```

### `PreferenceDoesNotExistError` exception

This exception is raised when a preference is not found.

```javascript
MashupPlatform.prefs.PreferenceDoesNotExistError
```

### MashupPlatform.mashup

The mashup module contains one attribute:

- `context`

and a the following two methods:

- `addWidget`
- `createWorkspace`

#### `context` attribute

This attribute contains the context manager of the mashup. See the
documentation about context managers](#slide45) for more information.

**Example usage:**

```javascript
MashupPlatform.mashup.context.get('title');
```

#### `addWidget` method
> new in WireCloud 0.8.0

This method allows widgets and operators to add new temporal widgets into the
current workspace.

This method is only available when making use of the `DashboardManagement`
feature.

```javascript
MashupPlatform.mashup.addWidget(widget_ref, options)
```

- `widget_ref` (String): id (vendor/name/version) of the widget to use.
- `options` (Object, optional): object with extra options.

Supported options:

- `title` (String): Title for the widget. If not provided, the default title
  provided in the widget description will be used.
- `permissions` (Object): Object with the permissions to apply to the widget.
  Currently, the following permissions are available: `close`, `configure`,
  `minimize`, `move`, `resize`.
- `preferences` (Object): Object with the initial configuration for the
  preferences. If not provided, the default configuration of the widget is used.
- `properties` (Object): Object with the initial configuration for the
  properties. If not provided, the default configuration of the widget is used.
- `refposition` (ClientRect): Element position to use as reference for placing
  the new widget. You can obtain such a object using the
  [`getBoundingClientRect`][getBoundingClientRect] method.
- `top` (String, default: `0px`): This option specifies the distance between the top margin edge
  of the element and the top edge of the dashboard. This value will be ignored
  if you provide a value for the `refposition` option.
- `left` (String, default: `0px`): This option specifies the distance between the left margin edge
  of the element and the top edge of the dashboard. This value will be ignored
  if you provide a value for the `refposition` option.
- `width` (String, default: `null`): This options specifies the width of the
  widget. If not provided, the default width will be used.
- `height` (String, default: `null`): This options specifies the height of the
  widget. If not provided, the default height will be used.


**Example usage:**

```javascript
var widget = MashupPlatform.mashup.addWidget('CoNWeT/kuernto-one2one/1.0', {
    "permissions": {
        "close": false,
        "configure": false
    },
    "preferences": {
        "stand-alone": {
            "value": false
        }
    },
    "top": "0px",
    "left": "66%"
});
```

[getBoundingClientRect]: https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect

#### `createWorkspace` method
> new in WireCloud 0.8.0

This method allows widgets and operators to create new workspaces for the
current user. This method is asynchronous.

This method is only available when making use of the `DashboardManagement`
feature.

```javascript
MashupPlatform.mashup.createWorkspace(options)
```

- `options` (Object): object with the options to use for creating the workspace.
  At least one of the following options has to be provided: `name`, `mashup`,
  `workspace`.

Supported options:

- `name` (String): Name for the new workspace. This option is required if
  neither `mashup` nor `workspace` option is used. If not provided, the name
  of the mashup will be taken from the mashup or from the workspace to use
  as template.
- `mashup` (String): id (vendor/name/version) of the mashup to use as
  template for creating the new workspace. Both the `mashup` and `workspace`
  attributes are optional, but should not be specified together.
- `workspace` (String): id (owner/name) of the workspace to use as template
  for creating the new workspace. Both the `mashup` and `workspace`
  attributes are optional, but should not be specified together.
- `allowrenaming` (Boolean, default `false`): if this option is `true`,
  WireCloud will rename the workspace if a workspace with the same name
  already exists.
- `preferences` (Object): Object with the initial values for the workspace
  preferences. If not provided, the default values will be used.
- `onSuccess` (Function): callback to invoke if the workspace is created successfully.
- `onFailure` (Function): callback to invoke if some error is raised while creating the
  workspace

**Example usage:**

```javascript
MashupPlatform.mashup.createWorkspace({
    name: 'New workspace',
    mashup: 'CoNWeT/ckan-graph-mashup/1.0',
    onSuccess: function (workspace) {
        alert(workspace.owner + "/" + workspace.name + " created successfully");
    }
});
```

### MashupPlatform.operator

This module is only available when running inside an operator. Currently,
WireCloud provides the following attributes:

- [`id`](#id-attribute)
- [`context`](#context-atribute)

and a the following method:

- [`log`](#log-method)

#### `id` attribute

This attribute contains the operator's id.

```javascript
MashupPlatform.operator.id
```

#### `context` attribute

This attribute contains the context manager of the operator. See the
[documentation about context managers](#slide54) for more information.

**Example usage:**

```javascript
MashupPlatform.operator.context.get('version');
```

#### `log` method

This method writes a message into the WireCloud's log console.

```javascript
MashupPlatform.operator.log(msg, level)
```

- `msg` (String): is the text of the message to log.
- `level` (optional, default: `MashupPlatform.log.ERROR`): This optional
  parameter specifies the level to use for logging the message. See
  [MashupPlatform.log](#slide15) for available log levels.

**Example usage:**

```javascript
MashupPlatform.operator.log('error message description');
MashupPlatform.operator.log('info message description', MashupPlatform.log.INFO);
```


### MashupPlatform.widget

This module is only available when running inside a widget. Currently,
WireCloud provides the following attributes:

- [`id`](#id-attribute-1)
- [`context`](#context-atribute-1)

and the following methods:

- [`getVariable`](#getvariable-method)
- [`drawAttention`](#drawAttention-method)
- [`log`](#log-method-1)

#### `id` attribute

This attribute contains the widget's id.

```javascript
MashupPlatform.widget.id
```

#### `context` attribute

This attribute contains the context manager of the widget. See the
[documentation about context managers](#slide54) for more information.

```javascript
MashupPlatform.widget.context
```

**Example usage:**

```javascript
MashupPlatform.widget.context.get('version');
```

#### `getVariable` method

Returns a widget variable by its name.

```javascript
MashupPlatform.widget.getVariable(name)
```

- `name` is the name of the persistent variable as defined on the `config.xml`
  file.

**Example usage:**

```javascript
var variable = MashupPlatform.widget.getVariable('persistent-var');
variable.set(JSON.stringify(data));
```

#### `drawAttention` method

Makes WireCloud notify that the widget needs user's attention.

```javascript
MashupPlatform.widget.drawAttention()
```
#### `log` method

Writes a message into the WireCloud's log console.

```javascript
MashupPlatform.widget.log(msg, level)
```

- `msg` (String): is the text of the message to log.
- `level` (optional, default: `MashupPlatform.log.ERROR`): This optional
  parameter specifies the level to use for logging the message. See
  [MashupPlatform.log](#slide15) for available log levels.

**Example usage:**

```javascript
MashupPlatform.operator.log('error message description');
MashupPlatform.operator.log('warning message description', MashupPlatform.log.WARN);
```

### MashupPlatform.wiring

This module provides some methods for handling the communication between widgets
an operators through the wiring.

Currently this module is composed of four methods:

- [`hasInputConnections`](#slide46)
- [`hasOutputConnections`](#slide47)
- [`pushEvent`](#slide48)
- [`registerCallback`](#slide49)
- [`registerStatusCallback`](#slide50)

and three exceptions:

- [`EndpointDoesNotExistError`](#slide51)
- [`EndpointTypeError`](#slide52)
- [`EndpointValueError`](#slide53)

#### `hasInputConnections` method
> new in WireCloud 0.8.0

Sends an event through the wiring.

```javascript
MashupPlatform.wiring.hasInputConnections(inputName)
```

This method can raise the following exceptions:

- `MashupPlatform.wiring.EndpointDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.wiring.hasInputConnections('inputendpoint');
```

#### `hasOutputConnections` method
> new in WireCloud 0.8.0

Sends an event through the wiring.

```javascript
MashupPlatform.wiring.hasOutputConnections(outputName)
```

This method can raise the following exceptions:

- `MashupPlatform.wiring.EndpointDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.wiring.hasOutputConnections('outputendpoint');
```

#### `pushEvent` method

Sends an event through the wiring.

```javascript
MashupPlatform.wiring.pushEvent(outputName, data)
```

This method can raise the following exceptions:

- `MashupPlatform.wiring.EndpointDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.wiring.pushEvent('outputendpoint', 'event data');
```

#### `registerCallback` method

Registers a callback for a given input endpoint. If the given endpoint already
has registered a callback, it will be replaced by the new one.

```javascript
MashupPlatform.wiring.registerCallback(inputName, callback)
```

This method can raise the following exceptions:

- `MashupPlatform.wiring.EndpointDoesNotExistError`

**Example usage:**

```javascript
MashupPlatform.wiring.registerCallback('inputendpoint', function (data_string) {
    var data = JSON.parse(data_string);
    ...
});
```

#### `registerStatusCallback` method

Registers a callback that will be called everytime the wiring status of the
mashup changes.

```javascript
MashupPlatform.wiring.registerCallback(inputName, callback)
```

**Example usage:**

```javascript
MashupPlatform.wiring.registerStatusCallback(function () {
    ...
});
```

#### `EndpointDoesNotExistError` exception

This exception is raised when an input/output endpoint is not found.

```javascript
MashupPlatform.wiring.EndpointDoesNotExistError
```

#### `EndpointTypeError` exception
> new in WireCloud 0.8.0

Widgets/operators can throw this exception if they detect that the data arriving
an input endpoint is not of the expected type.

```javascript
MashupPlatform.wiring.EndpointTypeError
```

**Example usage:**

```javascript
MashupPlatform.wiring.registerCallback('inputendpoint', function (data) {
    try {
        data = JSON.parse(data);
    } catch (error) {
        throw new MashupPlatform.wiring.EndpointTypeError('data should be encoded as JSON');
    }

    ...
});
```

#### `EndpointValueError` exception
> new in WireCloud 0.8.0

Widgets/operators can throw this exception if they detect that the data arriving
an input endpoint, although has the right type, contains inappropriate values.

```javascript
MashupPlatform.wiring.EndpointValueError
```

**Example usage:**

```javascript
MashupPlatform.wiring.registerCallback('inputendpoint', function (data) {
    ...

    if (data.level > 4 || data.level < 0) {
        throw new MashupPlatform.wiring.EndpointTypeError('level out of range');
    }

    ...
});
```

### Context Managers

Each context managers supports three methods:

* [`getAvailableContext`](#get-available-method)
* [`get`](#slide57)
* [`registerCallback`](#slide58)

#### `getAvailableContext` method

This method provides information about what concepts are available for the given level

**Example usage:**

```javascript
MashupPlatform.context.getAvailableContext();
```

#### `get` method

Retrieves the current value for a concept

**Example usage:**

```javascript
MashupPlatform.widget.context.get('heightInPixels');
MashupPlatform.mashup.context.get('name');
MashupPlatform.context.get('username');
```

#### `registerCallback` method

Allows to register a callback that will be called when any of the concepts are modified

**Example usage:**

```javascript
MashupPlatform.widget.context.registerCallback(function (new_values) {
    if ('some-context-concept' in new_values) {
        // some-context-concept has been changed
        // new_values['some-context-concept'] contains the new value
    }
});
```