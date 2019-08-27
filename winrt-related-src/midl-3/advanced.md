---
author: stevewhims
description: Advanced topics, and shorthand syntax.
title: Advanced topics, and shorthand
ms.author: stwhi
ms.date: 08/07/2019
ms.topic: reference
keywords: windows 10, uwp, winrt, api, reference, idl, midl, 3.0, 3, midl3
ms.localizationpriority: medium
---

# Advanced topics, and shorthand

## Shorthand

If you use a parameterized type without specifying a namespace, then the MIDL 3.0 compiler looks for it in the **Windows.Foundation.Collections namespace**. In practice, that means that you can use the following shorthand.

|Short version|Long version|
|-|-|
| IIterable<T> | Windows.Foundation.Collections.IIterable<T> |
| IIterator<T> | Windows.Foundation.Collections.IIterator<T> |
| IKeyValuePair<K, V> | Windows.Foundation.Collections.IKeyValuePair<K, V> |
| IMap<K, V> | Windows.Foundation.Collections.IMap<K, V> |
| IMapChangedEventArgs<K> | Windows.Foundation.Collections.IMapChangedEventArgs<K> |
| IMapView<K, V> | Windows.Foundation.Collections.IMapView<K, V> |
| IObservableMap<K, V> | Windows.Foundation.Collections.IObservableMap<K, V> |
| IObservableVector<T> | Windows.Foundation.Collections.IObservableVector<T> |
| IVector<T> | Windows.Foundation.Collections.IVector<T> |
| IVectorView<T> | Windows.Foundation.Collections.IVectorView<T> |
| MapChangedEventHandler<K, V> | Windows.Foundation.Collections.MapChangedEventHandler<K, V> |
| VectorChangedEventHandler<T> | Windows.Foundation.Collections.VectorChangedEventHandler<T> |
	
This mechanism doesn't apply to the **Windows.Foundation** namespace. For example, you have to write the full name **Windows.Foundation.IAsyncAction**.

## Overloads

The default behavior for overloaded methods and constructors is to append a numeric suffix to the ABI names for the second and subsequent overloads within an interface.

```idl
[contract(Windows.Foundation.UniversalApiContract, 1)]
runtimeclass Sample
{
    // ABI name is "DoSomething"
    void DoSomething();

    // ABI name is "DoSomething2"
    void DoSomething(Int32 intensity);

    [contract(Windows.Foundation.UniversalApiContract, 2)]
    {
        // ABI name is "DoSomething" (new interface)
        void DoSomething(Int32 intensity, String label);
    }
}
```

This default naming doesn't match recommended API design guidelines, so override it with the [method_name] attribute.

```idl
[contract(Windows.Foundation.UniversalApiContract, 1)]
runtimeclass Sample
{
    void DoSomething();

    [method_name("DoSomethingWithIntensity")]
    void DoSomething(Int32 intensity);

    [contract(Windows.Foundation.UniversalApiContract, 2)]
    {
        [method_name("DoSomethingWithIntensityAndLabel")]
        void DoSomething(Int32 intensity, String label);
    }
}
```

## Implement a non-exclusiveto interface

Deriving your runtime class from an interface automatically declares the members of that interface. Don't redeclare them. If you do, then the MIDL 3.0 compiler assumes that you want to implement a separate method **M()** that hides the one from the interface.

```idl
interface I
{
    void M();
}

runtimeclass C : I
{
    // Don't redeclare M(). It's automatically inherited from interface I.
    // void M();
}
```

## Specify the default interface

If you don't specify a default interface, then the MIDL 3.0 compiler chooses the first instance interface. To override this selection, insert the attribute `[default]` before the interface you want to be the default interface.

```idl
// Declaring an external interface as the default
runtimeclass C : [default]I { ... }

// Declaring a specific exclusiveto interface as the default.
// This is very unusual.
runtimeclass C
{
    ...

    [default][interface_name(...)]
    {
        ...
    }
}
```

## Backward compatibility attributes

If you're converting MIDL 1.0 or MIDL 2.0 to MIDL 3.0 (also see [Transition to MIDL 3.0 from classic MIDLRT](from-midlrt.md)), then you'll need to customize things that are normally autogenerated so that the autogenerated values match the existing ones.

- To specify the name and UUID of an interface, use the `[interface_name("fully.qualified.name", UUID)]` attribute.
- To specify the name and UUID of a factory interface, use the` [constructor_name("fully.qualified.name", UUID)]` attribute.
- To specify the name and UUID of a static interface, use the `[static_name("fully.qualified.name", UUID)]` attribute.
- To specify the name of an output parameter, use the `[return_name("name")]`attribute.
- To specify the name of a method, use the `[method_name("name")]` attribute.

The "UUID" part of the `interface_name`, `constructor_name`, and `static_name` attributes is optional. If omitted, MIDL will auto-generate an IID.

```idl
[contract(Windows.Foundation.UniversalApiContract, 1)]
[interface_name("ISample", ceb27355-f772-407c-9540-6467a7199bc7)]
[constructor_name("ISampleFactory", 863B201F-BC7B-471E-A066-6425E8E639EC)]
[static_name("ISampleStatics", 07254c86-3b01-4e24-b52b-14e832c15483)]
runtimeclass Sample
{
    [method_name("CreateWithIntensity")]
    Sample(Int32 intensity);

    static Boolean ShowConfigurationUI();

    [return_name("count")]
    Int32 GetCount();

    [constructor_name("ISampleFactory2", FEA29CEC-7768-41DE-9A46-CAAAA4622588)]
    [static_name("ISampleStatics2", 191235b5-a7b5-456f-86ea-abd1a735c6ab)]
    [interface_name("ISample2", d870ed2e-915a-48a2-ad17-c05efa123db7)]
    [contract(Windows.Foundation.UniversalApiContract, 2)]
    {
        [method_name("CreateWithIntensityAndLabel")]
        Sample(Int32 intensity, String label);

        static Boolean IsSupported();

        [return_name("success")]
        Boolean TrySomething();
    }
}
```

The MIDL 3.0 compiler won't warn you if you get your `xxx_name` annotation confused. For example, the following example compiles without error, even though there are no instance members to put in the `interface_name` interface. The presence of the `interface_name` attribute causes an empty interface named **ISampleFactory2** to be generated.

```idl
[contract(Windows.Foundation.UniversalApiContract, 1)]
[interface_name("ISample", ceb27355-f772-407c-9540-6467a7199bc7)]
runtimeclass Sample
{
    [return_name("count")]
    Int32 GetCount();

    // !WRONG! Should be constructor_name.
    [interface_name("ISampleFactory2", FEA29CEC-7768-41DE-9A46-CAAAA4622588)]
    [contract(Windows.Foundation.UniversalApiContract, 2)]
    {
        // MIDL will autogenerate ISampleFactory since there is no [constructor_name]
        Sample(Int32 intensity);
   }
}
```

## Empty classes

While this usage is somewhat obscure, it's sometimes necessary to author an empty class (a class with no members), or an empty factory class. A common example of this occurs with an **EventArgs** class. If an event is introduced, sometimes there's no need for arguments to the event (the event being signaled doesn't require additional context). Our API design guidelines recommend strongly that an **EventArgs** class be provided, allowing the class to add new event arguments in the future. However, consider this empty class.

```idl
runtimeclass MyEventsEventArgs
{
}
```

That class produces this error.

```console
error MIDL5056 : [msg]a runtime class without a default attribute cannot be used as a parameter. Runtime classes must have methods or be flagged as marker classes if they are used as a parameter [context]: Windows.Widgets.MyEventsEventArgs [ RuntimeClass 'Windows.Widgets.MyEventsEventArgs' ( Parameter 'result' ) ]
```

There are several ways to fix this. The simplest one is to use the `[default_interface]` attribute to express that the lack of methods is intentional, and not an authoring error. Here's how to do that.

```idl
// An empty runtime class needs a [default_interface] tag to indicate that the 
// emptiness is intentional.
[default_interface] 
runtimeclass MyEventsEventArgs
{
}
```

Another way to fix is this is with the `[interface_name]` attribute. If MIDL encounters the `[interface_name]` on a class with no normal methods (or a versioned block with no normal methods), then it generates an empty interface for that block. Similarly, if the` [static_name]` or `[activatable_name]` attribute is present on a class or versioned block with no static (or constructors), then it will generate an empty interface for that static interface or constructor.

Be careful not to confuse an empty class with a static class. You can have instances of an empty class (although they don't do anything), but you cannot have instances of a static class.

## Empty interfaces

An empty interface (also called a marker interface) must specify an explicit `[uuid(...)]`.

```idl
// An empty interface must specify an explicit [uuid] to ensure uniqueness.
[uuid("94569FA9-D3BB-4D01-BF7C-B8E1D8F8B30C")]
[contract(Windows.Foundation.UniversalApiContract, 1)]
interface ISomethingMarker
{
}
```

If you forget, then this error is produced.

```console
error MIDL4010 : [msg]Cannot find the guid attribute of an interface or a delegate. [context]Windows.Widgets.ISomethingMarker
```

Autogenerated UUIDs are a hash of the interface contents, but if that were done for empty interfaces, then all marker interfaces would end up with the same UUID.

## Scoped enums

If you pass the `/enum_class` command switch to the MIDL 3.0 compiler, then the enumerations emitted by the compiler are declared as scoped enums (enum class). Don't use scoped enums for public types.

## Composition and activation

For more info about **composable** classes, see [XAML controls; bind to a C++/WinRT property](/windows/uwp/cpp-and-winrt-apis/binding-property).

You can specify `unsealed runtimeclass` to create a composable class. Furthermore, you can specify `unsealed runtimeclass unsealed` to indicate whether the class uses COM aggregation, or regular activation. This is significant for base classes with public constructors.

## Interpreting error messages

```console
error MIDL2025: [msg]syntax error [context]: expecting > or, near ">>"
```

If you write `IAsyncOperation<IVectorView<Something>>`, then the `>>` is interpreted as a right-shift operator. To work around this, put a space between the two greater-than signs to give `IAsyncOperation<IVectorView<Something> >`.

```console
error MIDL2025: [msg]syntax error [context]: expecting . near ","
```

This error occurs if you specify a non-existent contract, possibly due to a typo.

```idl
[contract(Windows.Foundation.UniversalApiContact, 5)]
                                         ^^^^^^^ typo
```

```console
error MIDL5082: [msg]the version qualifying an enum's field cannot be less than the version of the enum itself
```

This error message is generated not only for the reason in the error message, but also if you try to put the fields of an enum into different contracts. It's legal to have the fields of an enum belong to different versions of the same contract, but they can't be in different contracts entirely.

```console
error MIDL5161: [msg]Invalid method parameter name [context]: Parameter 'result' (or 'operation' or 'value')
```

Parameter names `result` and `operation` are reserved in methods. The parameter name `value` is reserved in constructors.

```console
error MIDL5023: [msg]the arguments to the parameterized interface are not valid
```

Check that you spelled the interface name correctly.

## Don't mix MIDL 2.0 and MIDL 3.0 within a single interface

Each interface and runtime class must be either completely MIDL 2.0, or completely MIDL 3.0. It *is* legal to reference a MIDL 3.0 interface from a MIDL 2.0 runtime class.

If you try to mix MIDL 2.0 and MIDL 3.0, the compiler treats the entire entity as MIDL 2.0, which results in compiler errors. You can run into this issue if you accidentally use MIDL 2.0 syntax when you intended to use MIDL 3.0.

```idl
interface ICollapsible
{
    void Collapse();

    boolean IsCollapsed { get; } // WRONG!
 // ^^^^^^^ Lowercase "boolean" is MIDL 2.0.

    Boolean IsCollapsed { get; } // RIGHT!
 // ^^^^^^^ Uppercase "Boolean" is MIDL 3.0.
};
```

## Delegates returning **HRESULT**

A delegate that returns an **HRESULT** is ambiguous. It could be a classic (pre-MIDL 3.0) declaration of a delegate that nominally returns void (where the **HRESULT** is used to propagate an exception), or it could be a modern (MIDL 3.0) declaration of a delegate that nominally returns an **HRESULT**.

The compiler resolves the ambiguity by looking at other parts of the declaration. For example, if the parameters are declared with classic syntax, then the declaration is assumed to be classic. If the parameters are declared with modern syntax, then the declaration is assumed to be modern.

```idl
delegate HRESULT AmbiguousDelegate(INT32 value, RuntimeClassName* r);
```

- Parameters use classic syntax, so this is assumed to be a classic declaration.
- The modern equivalent is `delegate void AmbiguousDelegate(Int32 value, RuntimeClassName r);`.

```idl
delegate HRESULT AmbiguousDelegate(Int32 value, RuntimeClassName r);
```

- Parameters use modern syntax, so this is assumed to be a modern declaration.
- The classic equivalent is `delegate HRESULT AmbiguousDelegate(Int32 value, RuntimeClassName* r, [out, retval] HRESULT* result);`.

Sometimes, the parameter list is insufficient to resolve the ambiguity. For example, an empty parameter list, or a parameter list that consists only of enums is legal in both classic and modern syntax. In such cases, the MIDL 3.0 compiler defaults to classic.

```idl
delegate HRESULT AmbiguousDelegate(MyEnum e);
```

- Interpreted as a classic delegate, where the delegate nominally returns void, and the HRESULT is for propagating an exception.
- If you really wanted a delegate that returned an **HRESULT**, you need to use classic syntax: `delegate HRESULT AmbiguousDelegate(MyEnum e, [out, retval] HRESULT* result);`.

Fortunately, it's rare to have a delegate that nominally returns HRESULT.