# XBL Modes

<!-- toc -->

## Introduction

The XBL component binding defined with `<xbl:binding>` supports the `xxbl:mode` attribute, which contains an optional space-separated list of tokens. Each token enables a mode, as described below. Modes change the behavior of the component.

Example:

```xml
<xbl:binding
    id="fr-code-mirror"
    element="fr|code-mirror"
    xxbl:mode="lhha binding value external-value focus">
```

## The binding mode

The `binding` mode enables an optional XForms single-item binding. This means that the component supports the XForms binding attributes:

* `model`
* `context`
* `ref`
* `bind`

When a component has a binding, UI events are dispatched depending on the bound item:

* `xforms-enabled` / `xforms-disabled`
* `xforms-readonly` / `xforms-readwrite`
* `xforms-optional` / `xforms-required`
* `xforms-valid` / `xforms-invalid`

You can access the actual bound node via the `xxf:binding()` function:

```xml
xxf:binding('fr-foo')
```

The id passed must be the id of the `xbl:binding` element.

The `xxf:binding-context()` function returns the XPath evaluation context of the binding

```xml
xxf:binding-context('fr-foo')
```

For an example, see [Creating a single-node binding](http://doc.orbeon.com/xforms/xbl/tutorial.html#creating-a-single-node-binding).

## The value mode

The `value` mode makes the component hold a value through its binding. This means the component behaves like `<xf:input>` and other controls and dispatches `xforms-value-changed` events.

You use this mode in addition to `binding`.

For an example, see [Adding support for a value](http://doc.orbeon.com/xforms/xbl/tutorial.html#adding-support-for-a-value).

## The external-value mode

[SINCE Orbeon Forms 2016.1]

You use the `external-value` mode in addition to the `binding` and `value` modes.

You use this mode when the component implementation is mostly done in JavaScript and not with nested XForms controls.

By default, `value` doesn't expose the control's value to the client. By adding the `external-value` mode, the control's value is made available to the client and accessible via JavaScript.

For more details, see [Support for the external-value mode](javascript.md#support-for-the-externalvalue-mode).

## The javascript-lifecycle mode

[SINCE Orbeon Forms 2016.1]

You use this mode when the component implementation is mostly done in JavaScript and not with nested XForms controls.

The `javascript-lifecycle` mode lets Orbeon Forms handle more of a JavaScript companion class's lifecycle, including initialization, destruction, and state changes. You often use it in conjunction with `external-value`.

For example prior to Orbeon Forms 2016.1, you would call the component's `init()` method from XForms event handlers. With `javascript-lifecycle`, this is no longer needed.

For more details, see [Support for the javascript-lifecycle mode](javascript.md#support-for-the-javascriptlifecycle-mode).

## The lhha and custom-lhha modes

The `lhha` mode allows the component to support the `<xh:label>`, `<xh:hint>`, `<xh:help>` and `<xh:alert>` element, whether:

- directly nested under the component's bound element
- or using the `for` attribute

By default, markup is output for the LHHA elements. You can disable this with the additional `custom-lhha` mode.

For an example, see [Adding LHHA elements](http://doc.orbeon.com/xforms/xbl/tutorial.html#adding-lhha-elements).

[SINCE Orbeon Forms 4.5]

Sometimes, a component implementation uses HTML form controls, and you would like a `<label>` element pointing to it with a `for` attribute in the generated HTML markup.

You enable this with the `xxbl:label-for` attribute.

The value of the attribute must be the id of a nested XForms control or a nested HTML control element.

For examples, see [some of the Orbeon Forms XBL components](https://github.com/orbeon/orbeon-forms/tree/master/form-runner/jvm/src/main/resources/xbl/orbeon).
                                                            https://github.com/orbeon/orbeon-forms/tree/master/form-runner/jvm/src/main/resources/xbl/orbeon

## The focus mode

You use this mode when the component implementation is mostly done in JavaScript and not with nested XForms controls.

The `focus` mode allows the component to handle keyboard focus natively, so that the XForms engine is aware of focus.

[SINCE Orbeon Forms 2016.1]

When the component receives focus, for example following an XForms `<xf:setfocus>` action, the JavaScript companion class's `xformsFocus()` method is called if present. If the method is missing, then the legacy `setFocus()` method is called.

[UNTIL Orbeon Forms 4.10 included]

The JavaScript companion class's `setFocus()` method is called if present.

## The nohandlers mode

The `nohandlers` mode disables automatic processing of nested event handlers. You should only need this for very special components.

For an example and more details, see [Component user: attaching event handlers to the bound node](http://doc.orbeon.com/xforms/xbl/event-handling.html#component-user-attaching-event-handlers-to-the-bound-node).
