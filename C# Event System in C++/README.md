# C#-Style Event System in C++

## Overview

A single-header library that brings C#/.NET-style events to embedded C++.  The primary benefits include no STL, no RTTI and no exceptions, which makes it suitable for AVR-class microcontrollers and it compiles as C++03, so it works with older embedded toolchains.

## Usage

```cpp
// Declare an event (0, 1 or 2 arguments)
Event<Button*> Clicked;

// Subscribe a free function or a member function
const IEventHandler* h = myButton.Clicked += new EventHandler<GlobalFunction, Button*>(OnClicked);
myButton.Clicked += new EventHandler<Form, Button*>(&myForm, &Form::OnClicked);

// Raise it
Clicked(this);

// Unsubscribe
myButton.Clicked -= h;
```

See [Event.h](Event.h) for full usage notes.

## Implementation Notes

- **Type erasure without RTTI:** `EventHandler<T, U, V>` derives from a non-template `IEventHandler` interface.  This lets one `Event` hold handlers for free functions and for member functions of any class.  Arguments cross the interface wrapped in a templated `EventArg<T>` and are recovered with a `static_cast`, so `dynamic_cast` is never needed.
- **Compact handler storage:** the six supported callback signatures (global or member, with 0, 1 or 2 parameters) share a single `union` of function pointers, which keeps each handler object small.
- **Simple memory model:** subscribers live in a dynamically grown array of handler pointers.  The event owns its handlers and frees them when they're unsubscribed or when the event is destroyed. Each handler gets an ID when it subscribes, which `-=` uses to remove it.
