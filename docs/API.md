# TinyDOM API Reference

## Core Interface

The `DOM` interface is the main entry point for interacting with the browser. It is designed to be injected into your components.

```go
package tinydom

type DOM interface {
	// Get retrieves an element by its ID.
	// It uses an internal cache to avoid repeated DOM lookups.
	Get(id string) (Element, bool)

	// Mount injects a component into a parent element.
	// 1. It calls component.RenderHTML()
	// 2. It sets the InnerHTML of the parent element (found by parentID)
	// 3. It calls component.OnMount() to bind events
	//
	// Note: The parentID element MUST exist in the DOM before calling Mount.
	// If it is not found, Mount returns an error.
	Mount(parentID string, component Component) error

	// Unmount removes a component from the DOM (by clearing the parent's HTML or removing the node)
	// and cleans up any event listeners registered via the Element interface.
	Unmount(component Component)
}
```

## Element Interface

The `Element` interface represents a DOM node with methods for content manipulation, styling, and event handling.

**📖 Full API Documentation**: See [`element.go`](../element.go) for complete interface definition with detailed examples.

### Key Features

All content methods (`SetText`, `SetHTML`, `AppendHTML`, `SetAttr`, `SetValue`) accept variadic arguments and support:
- **String concatenation** without spaces
- **Printf-style formatting** with `%` specifiers
- **Localized content** using `D.*` dictionary
- **Mixed types** (strings, numbers, etc.)

### Quick Examples

```go
// Simple concatenation
elem.SetText("Count: ", 42)              // -> "Count: 42"

// HTML with format strings
elem.SetHTML("<h1>%v</h1>", title)       // -> "<h1>My Title</h1>"

// Localized content
elem.SetText(D.Hello)                    // -> "Hello" (EN) or "Hola" (ES)

// Multiline HTML components
elem.SetHTML(`<div class='card'>
	<h2>%L</h2>
	<p>%v</p>
</div>`, D.Title, count)

// Attributes
elem.SetAttr("id", "item-", 42)          // -> id="item-42"
elem.SetAttr("href", "/page/", pageNum)  // -> href="/page/5"
```

For complete method signatures and more examples, see [`element.go`](../element.go).


## Event Interface

The `Event` interface wraps the native browser event to provide a safe, simplified API.

```go
type Event interface {
	// PreventDefault prevents the default action of the event.
	PreventDefault()

	// StopPropagation stops the event from bubbling up the DOM tree.
	StopPropagation()

	// TargetValue returns the value of the event's target element.
	// Useful for input, textarea, and select elements.
	TargetValue() string
}
```


## Component Interface

The minimal interface that all components must implement for both SSR (backend) and WASM (frontend):

```go
type Component interface {
	// ID returns the unique identifier of the component's root element.
	ID() string

	// RenderHTML returns the full HTML string of the component.
	// The root element of this HTML MUST have the id returned by ID().
	RenderHTML() string
}
```

### WASM-Only: Mountable

For interactive components in the browser, implement the `Mountable` interface:

```go
//go:build wasm

type Mountable interface {
	Component

	// OnMount is called after the HTML has been injected into the DOM.
	// The DOM instance is passed so the component can bind events and interact with elements.
	OnMount(dom DOM)

	// OnUnmount is called before the component is removed from the DOM.
	OnUnmount()
}
```

**Key Change**: Components now receive the `DOM` instance as a parameter in `OnMount()` instead of storing it as a field.

### Backend-Only: CSS and JS Rendering

For SSR with styles and scripts, optionally implement these interfaces:

```go
//go:build !wasm

type CSSRenderer interface {
	Component
	RenderCSS() string
}

type JSRenderer interface {
	Component
	RenderJS() string
}
```

These methods are only called on the backend for server-side rendering.
