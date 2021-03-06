# JavaScript Companion Classes

<!-- toc -->

## Rationale

Some components do not require any custom JavaScript code, for example components which combine other controls (such as a date components made of separate input fields or dropdown menus). In such cases, you implement all the logic with XForms.

On the other hand some components are introduced to encapsulate functionality mainly implemented in JavaScript. Orbeon Forms provides an easy way to interface with the JavaScript side. Each JavaScript-based component must define a JavaScript class used to handle the component's lifecycle as well as hold custom data and functions. This class is called the component's *companion class*. One instance of this class is created by Orbeon Forms for each instance of relevant (visible) control. We call these instances *companion instances*.

## Directory layout

You place your JavaScript files alongside your XBL file. See [Directory layout](bindings.md#directory-layout) for details.

To include a companion JavaScript file, use the `<xbl:script>` element directly within the `<xbl:xbl>` element:


```xml
<xbl:xbl
    xmlns:xf="http://www.w3.org/2002/xforms"
    xmlns:acme="http://www.acme.com/xbl"
    xmlns:xbl="http://www.w3.org/ns/xbl">

    <xbl:script src="/xbl/acme/multi-tool/multi-tool.js"/>

    <xbl:binding
        id="acme-multi-tool"
        element="acme|multi-tool">

        ...binding definition here...

    </xbl:binding>
</xbl>
```

## Creating and declaring a companion class

### With Orbeon Forms 2016.1 and newer

Orbeon Forms 2016.1 provides a simple way to declare a companion class. Here is the overall structure:

```javascript
(function() {

    // Optional shortcut to jQuery
    var $ = ORBEON.jQuery;

    // Register your companion class by its binding name
    ORBEON.xforms.XBL.declareCompanion('acme|multi-tool', {

        // Your custom data goes here
        myField: null,

        init: function() {
            // Perform your JavaScript initialization here
        },
        destroy: function() {
            // Perform your JavaScript clean-up here
        },
        xformsUpdateReadonly: function(readonly) {
            // Orbeon Forms calls this when the control's readonly status changes
        },
        xformsUpdateValue: function(newValue) {
            // Orbeon Forms calls this when the control's value changes
        },
        xformsGetValue: function() {
            // Orbeon Forms calls this to obtain the control's value
        },
        xformsFocus: function() {
            // Orbeon Forms calls this when the control is handed focus
        },
        // Your custom functions go here
        myFunction: function() {
            ...
        },
    });
})();

```

The first parameter to `declareCompanion()` must match the component's binding name, for example:

- if your component's binding is `acme|multi-tool`
    - pass `acme|multi-tool`
    - you place the JavaScript file under `/xbl/acme/multi-tool/multi-tool.js`
- if your component's binding is `foo|bar`
    - pass `foo|bar`
    - you place the JavaScript file under `/xbl/foo/bar/bar.js`

### With Orbeon Forms 4.10 and earlier

In the JavaScript file corresponding to your component, declare a companion class as follows:

```javascript
(function() {

    // Optional shortcut to jQuery
    var $ = ORBEON.jQuery;

    YAHOO.namespace("xbl.acme");
    YAHOO.xbl.acme.MultiTool = function() {};
    ORBEON.xforms.XBL.declareClass(YAHOO.xbl.acme.MultiTool, "xbl-acme-multi-tool");
    YAHOO.xbl.acme.MultiTool.prototype = {

        // Your custom data goes here
        myField: null,

        init: function() {
            // Perform your JavaScript initialization here
        },
        destroy: function() {
            // Perform your JavaScript clean-up here
        },
        xformsFocus: function() {
            // Orbeon Forms calls this when the control is handed focus
        },
        // Your custom functions go here
        myFunction: function() {
            ...
        },

        ...
    };
})();
```

* `YAHOO.namespace("xbl.acme")` defines a namespace for your class. All the XBL components components that ship with Orbeon Forms are in the `xbl.fr` namespace. If you are defining a component for your company or project named Acme, you could use the namespace `xbl.acme`.
* ` ORBEON.xforms.XBL.declareClass()` defines your class as an XBL class:
    * It takes 2 parameters: your class, and the CSS class found on the outermost HTML element that contains the markup for your components. This element is generated by Orbeon Forms, and the class name is derived from the by-name binding of your `<xbl:binding>`. For example, if the binding is `acme|multi-tool`, the class name is `xbl-acme-multi-tool`.

### The companion class

Both `declareCompanion()` and `declareClass()` create a JavaScript class and:

- add a static `instance()` method.
    - This is a factory method, which you use to get or create an object corresponding to the "current" component.
    - It returns an instance of the JavaScript class corresponding to the "current" component from which it is called.
    - It creates class instances as necessary, keeping track of existing instances and maintaining a 1-to-1 mapping between instances of the XBL component in the form and instance of your JavaScript class.
- add a `container` attribute.
    - In your JavaScript code, you can refer to `this.container` to retrieve the outermost HTML element corresponding to your component.

For example, if you know you have an input field with the class `acme-my-input` inside your component, you get the HTML element corresponding to that input with the following jQuery:

```javascript
$(this.container).find('.acme-my-input')[0]
```

### Summary of companion class methods

| Method                 | Description              | Mode                                 | Since              | Status |
|------------------------|--------------------------|--------------------------------------|--------------------|--------|
| `init`                 | initialize               | `javascript-lifecycle`<sup>1</sup>   | 2016.1<sup>1</sup> | fresh  |
| `destroy`              | clean-up                 | `javascript-lifecycle`               | 2016.1             | fresh  |
| `xformsUpdateReadonly` | change readonly status   | `javascript-lifecycle`               | 2016.1             | fresh  |
| `xformsUpdateValue`    | update value             | `external-value`                     | 2016.1             | fresh  |
| `xformsGetValue`       | get value                | `external-value`                     | 2016.1             | fresh  |
| `xformsFocus`          | hand focus               | `focus`                              | 2016.1             | fresh  |
| `setFocus`             | hand focus               | `focus`                              | 4.0                | legacy |
| `enabled`              | enable after full update |                                      | 4.0                | legacy |

1. The `init()` method is not new in Orbeon Forms 2016.1, but when using the `javascript-lifecycle` mode it is called automatically. Prior to Orbeon Forms 2016.1, or when not using the `javascript-lifecycle`  mode, it is called either via XForms event handlers, or as a side-effect of calls to `setFocus()` or `enabled()`.

## Calling methods upon XForms events

### With Orbeon Forms 2016.1 and newer

You can call a JavaScript method defined in your JavaScript class when an XForms event occurs. For example, to call the `myFunction()` method on `xforms-enabled`, write:

```xml
<xxf:action type="javascript" event="xforms-enabled">
    ORBEON.xforms.XBL.instanceForControl(this).myFunction();
</xxf:action>
```

### With Orbeon Forms 4.10 and earlier

With Orbeon Forms 4.10 and earlier, you obtain the class using the JavaScript namespaces you declared alongside the class, and directly call the `instance()` factory function:

```xml
<xxf:action type="javascript" event="xforms-enabled">
    YAHOO.xbl.acme.MultiTool.instance(this).myFunction();
</xxf:action>
```

## Support for the external-value mode

### Introduction

[SINCE Orbeon Forms 2016.1]

When the [`external-value` mode](modes.md#the-externalvalue-mode) is enabled, the following two methods must be provided:

- `xformsUpdateValue()`
- `xformsGetValue()`

For an example, see the implementation of the `fr:code-mirror` component: [`code-mirror.xbl`](https://github.com/orbeon/orbeon-forms/blob/master/form-runner/jvm/src/main/resources/xbl/orbeon/code-mirror/code-mirror.xbl) and [`code-mirror.js`](https://github.com/orbeon/orbeon-forms/blob/master/form-runner/jvm/src/main/assets/xbl/orbeon/code-mirror/code-mirror.js).

### The xformsUpdateValue method

The XForms engine calls this method:

- if the `javascript-lifecycle` mode is enabled, just after the control is initialized,
- when the internal value of the control changes,
- and in response to calls to `ORBEON.xforms.Document.setValue()`.

`xformsUpdateValue()` receives a string and must update the associated JavaScript control, making the value accessible to the user.

If the value is not set synchronously, `xformsUpdateValue()` must return a deferred object whose `done()` method must be called once the value is known to have been fully applied. For example, using jQuery:

```javascript
var editor   = this.editor;
var deferred = $.Deferred();
setTimeout(function() {
    editor.setValue(newValue);
    deferred.resolve();
}, 0);
return deferred.promise();
```

This allows the XForms engine to know when it is safe to call `xformsGetValue()` after a new value has been set.

When updating the value is synchronous, `xformsUpdateValue()` must simply return `undefined` (which is the default for JavaScript functions).

### The xformsGetValue method

The XForms engine calls this method when:

- it needs the control's value,
- and in response to calls to `ORBEON.xforms.Document.getValue()`.

`xformsGetValue()` returns a string obtained from the associated JavaScript control.

## Support for the javascript-lifecycle mode

### Introduction

[SINCE Orbeon Forms 2016.1]

When the [`javascript-lifecycle` mode](modes.md#the-javascriptlifecycle-mode) is enabled, the following methods should be provided:

- `init()`
- `destroy()`
- `xformsUpdateReadonly()`

*NOTE: The XForms engine does not call these methods if they are not present.*

On the JavaScript side, the lifecycle of a companion instance does not exactly follow that of the XForms controls when repeats are involved.

For an example, see [the implementation of the `fr:code-mirror` component](https://github.com/orbeon/orbeon-forms/blob/master/form-runner/jvm/src/main/resources/xbl/orbeon/code-mirror/code-mirror.xbl).

### The init method

The `init()` method is called when the control becomes relevant, including:

- when the page first loads and the control is initially relevant
- when the control becomes relevant at a later time
- when a new repeat iteration is added
- when `xxf:full-update` or `xxf:dynamic` replace an entire block of HTML on the client

### The destroy method

The `destroy()` method is called when the control becomes non-relevant, including:

- when the control becomes non-relevant after the page has loaded

As of Orbeon Forms 2016.1, it is *not* called:

- when a repeat iteration is removed
- when `xxf:full-update` or `xxf:dynamic` replace an entire block of HTML on the client

The assumption is that, when HTML elements are removed from the browser DOM, the associated JavaScript resources are garbage-collected. This means that you have to be careful about clean-up of event handlers in particular in such cases.

### The xformsUpdateReadonly method

The `xformsUpdateReadonly()` method is called when the control's readonly status changes.

It takes a boolean parameter set to `true` if the control becomes readonly and to `false` if the control becomes readwrite.

It is *not* called just after the control is initialized.

## Read-only parameters

So your JavaScript can access the current value of parameters and be notified when their value changes, include the `oxf:/oxf/xslt/utils/xbl.xsl` XSL file, and call `xxbl:parameter()` function for each parameter, as in:

```xml
<xbl:xbl>
    <xbl:script src="/xbl/orbeon/currency/currency.js"/>
    <xbl:binding id="fr-currency" element="fr|currency">
        <xbl:template xxbl:transform="oxf:unsafe-xslt">
            <xsl:transform version="2.0">
                <xsl:import href="oxf:/oxf/xslt/utils/xbl.xsl"/>
                <xsl:template match="/*">
                    ...
                    <xsl:copy-of select="xxbl:parameter(., 'prefix')"/>
                    <xsl:copy-of select="xxbl:parameter(., 'digits-after-decimal')"/>
                    ...
                </xsl:template>
            </xsl:transform>
        </xbl:template>
    </xbl:binding>
</xbl:xbl>
```

The arguments of `xxbl:parameter()` are:

1. The element corresponding to your component, e.g. the `<fr:currency>` element written by the user of your component. If your template matches on `/*`, this will be the current node.
2. The name of the parameter.

Then in JavaScript, you can access the current value of the property with:

```javascript
var prefixElement =
    $(this.container).find('.xbl-fr-currency-prefix')[0];

var prefix =
    ORBEON.xforms.Document.getValue(prefixElement.id);
```

Whenever the value of a parameter changes, a method of your JavaScript class is called. The name of this method is ` parameterFooChanged` if "foo" is the name of your property. Parameters names are in lowercase and use dash as a word separator, while the method names use camel case. E.g. if your parameter name is `digits-after-decimal`, you will defined a method `parameterDigitsAfterDecimalChanged`.

## Sending events from JavaScript

You can dispatch custom events to bindings from JavaScript using the `ORBEON.xforms.Document.dispatchEvent()` function. If  you are calling it with custom events, make sure you are allowing the custom event names on the binding first:

```xml
<xbl:binding xxf:external-events="acme-super-event acme-famous-event">
    <xbl:handlers>
        <xbl:handler event="acme-super-event" phase="target">
            ...
        </xbl:handler>
    </xbl:handlers
    ...
</xbl:binding>
```
