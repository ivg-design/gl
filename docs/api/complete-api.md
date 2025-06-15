# Golden Layout Comprehensive API

This document lists the main classes and methods exposed by the **golden-layout** package.
It is compiled from the TypeScript source and the existing documentation.
Each section explains when and how to use the methods along with small examples.

## LayoutManager

`LayoutManager` is the abstract base for the runtime layout manager. It provides
core functions for creating, loading and modifying layouts. Typically you work
with the concrete subclasses `GoldenLayout` or `VirtualLayout`.

### Key Methods

- **constructor(container?: HTMLElement)** – create a new manager. Pass the DOM element that should contain the layout.
- **loadLayout(layoutConfig)** – load a new `LayoutConfig` tree.
- **saveLayout()** – return the current layout as a resolved configuration so it can be saved and later reloaded.
- **setSize(width, height)** – set the pixel size of the layout. Automatically called when the container resizes if `resizeWithContainerAutomatically` is `true`.
- **addComponent(componentType, state?, title?)** – add a component using default location selectors. Returns the new location.
- **addComponentAtLocation(componentType, state?, title?, selectors?)** – insert a component at the first valid selector location.
- **newComponent(...)** – same as `addComponent` but returns the `ComponentItem` instance.
- **newItem(itemConfig)** – programmatically create stacks, rows, columns or components.
- **newDragSource(element, configCallback)** – make a DOM element a drag source that creates new items when dropped.
- **removeDragSource(dragSource)** – remove a previously created drag source.
- **focusComponent(item)** / **clearComponentFocus()** – manage which component has keyboard focus.
- **closeAllOpenPopouts()** – close any opened pop‑out windows (useful on page unload).

### Properties

- **resizeWithContainerAutomatically** – if `true` (default for `body` containers) the layout updates when the container is resized.
- **resizeDebounceInterval** – delay (ms) before resize after a container change.
- **resizeDebounceExtendedWhenPossible** – extend debounce when the container is still being resized.
- **eventHub** – an `EventHub` instance used for broadcasting events between windows.

### Example

```ts
const layout = new GoldenLayout(document.getElementById('layout'));
layout.loadLayout({
    root: {
        type: 'row',
        content: [{ type: 'component', componentType: 'Example' }]
    }
});
```

## VirtualLayout

`VirtualLayout` extends `LayoutManager` and supports **virtual components**. This binding
approach avoids reparenting DOM nodes and is recommended when integrating with frameworks
such as Angular or Vue.

### Additional Methods

- **bindComponentEvent(container, itemConfig)** – optional handler to create or fetch components when the layout requires them. Should return `{ component, virtual }`.
- **unbindComponentEvent(container)** – optional handler called when a component is removed.
- **checkAddDefaultPopinButton()** – utility that checks whether a default "pop in" button should be added to pop‑outs.
- **clearHtmlAndAdjustStylesForSubWindow()** – helper used internally when creating sub‑windows.

### Usage Tips

Assign `bindComponentEvent` and `unbindComponentEvent` when using frameworks so that
components are created and positioned by your application. In the handler you can attach
event listeners for `virtualRectingRequiredEvent` and related events to react to size and
visibility changes.

## GoldenLayout

`GoldenLayout` extends `VirtualLayout` and provides helper registration methods.
Use it when your components can be instantiated by the layout manager directly.

### Additional Methods

- **registerComponentConstructor(typeName, constructor, virtual?)** – register a component class.
- **registerComponentFactoryFunction(typeName, factory, virtual?)** – register a factory function returning a component instance.
- **registerGetComponentConstructorCallback(callback)** – fallback mechanism that returns a constructor based on the resolved config.
- **getRegisteredComponentTypeNames()** – get the list of registered component types.

### Example

```ts
const layout = new GoldenLayout(document.getElementById('layout'));
layout.registerComponentConstructor('Example', (container) => {
    container.element.textContent = 'Hello world';
});
layout.loadLayout({
    root: { type: 'component', componentType: 'Example' }
});
```

## ComponentContainer

Each component is wrapped in a `ComponentContainer`. The container handles sizing,
visibility and focus for the component.

### Important Methods

- **hide()** / **show()** – control visibility of the component item.
- **focus()** / **blur()** – give or remove keyboard focus.
- **close()** – close the component if it is closable.
- **replaceComponent(config)** – replace the current component instance without
  affecting layout.
- **setTitle(title)** – change the tab/title of the component.
- **setSize(width, height)** – programmatically change its size from within the component.

### Events

Handlers can be assigned to:

- **stateRequestEvent** – called during `saveLayout()` so the component can supply
  its latest state.
- **virtualRectingRequiredEvent** – fired when a virtual component needs to update
  its position and size.
- **virtualVisibilityChangeRequiredEvent** – fired when visibility changes.
- **virtualZIndexChangeRequiredEvent** – fired when z‑index should change.

## DragSource

`DragSource` turns a DOM element into a source for drag‑and‑drop creation of items.
Create one with `LayoutManager.newDragSource(element, callback)` and call
`removeDragSource()` when cleaning up.

## BrowserPopout

`BrowserPopout` represents a window created by popping out a stack. The pop‑out
inherits the same API as the root layout. Use `LayoutManager.closeAllOpenPopouts()`
to ensure all pop‑outs are closed when your application exits.

## EventHub

Use `eventHub.emitUserBroadcast(data)` to send messages between all open layouts
(main window and pop‑outs). Listen for `userBroadcast` events to receive them.

## Stack and Row/Column

`Stack` and `RowOrColumn` items make up the layout tree. They provide methods
for managing contained content:

- **addChild(itemConfig, index?)** – add a child item.
- **removeChild(item)** – remove a child.
- **setActiveComponentItem(item)** – on a `Stack`, focus a particular component.

## Tips for Building Layouts

1. Register components (or set up bind/unbind events).
2. Create a `GoldenLayout` or `VirtualLayout` instance and load a layout config.
3. Use `addComponent` or `newComponent` to add items programmatically.
4. Listen for events such as `stateRequestEvent` to persist component state.
5. Call `saveLayout()` to capture the current arrangement for later use.

This reference collects all major APIs for quick access without needing to
consult multiple files.
