# Webcomponents

## What are Web Components, and why would you choose them over React components when building a shared component library for multiple frontend applications?

=> Web Components are useful when building shared component libraries across multiple applications, especially in a `micro-frontend architecture`.

For example, different teams might build Admin, Dashboard, and User Profile applications using React, Angular, or Vue. Instead of implementing common UI elements like buttons, modals, and dropdowns separately, we can create reusable Web Components using native browser standards.

These components can then be consumed by different frameworks or even plain HTML applications, making them framework-independent.

This improves consistency, maintainability, and code reuse.

**React components cannot directly be consumed by Angular or plain HTML without React and its runtime. However, React components can be wrapped in Web Components to make them interoperable.**

## Custom Elements

There are two important concepts:
1. Defining a custom element.
2. Understanding its lifecycle.

```
class UserCard extends HTMLElement {

  constructor() {
    super();
    console.log("Constructor called");
  }

  connectedCallback() {
    console.log("Element added to DOM");
  }

  disconnectedCallback() {
    console.log("Element removed from DOM");
  }
}

customElements.define("user-card", UserCard);
```

|Lifecycle method |	When it executes |
|------------------|-------------------|
|constructor() | When the element is created or upgraded|
|connectedCallback() | When inserted into the DOM|
|disconnectedCallback() | When removed from the DOM|
|attributeChangedCallback() | When an observed attribute changes|
|adoptedCallback() | When moved to another document|

**One important detail: to receive attributeChangedCallback(), you must define observedAttributes.**

```
static get observedAttributes() {
  return ["name"];
}

attributeChangedCallback(name, oldValue, newValue) {
  console.log(oldValue, newValue);
}

card.setAttribute("name", "Rahul");
```

- setAttribute() triggers attribute changes.
- Changing a JavaScript property directly does not automatically trigger attributeChangedCallback().

## Shadow DOM

**Shadow DOM allows us to create an isolated DOM tree inside an element.**

It provides encapsulation for:
- DOM structure.
- CSS styles.
- Component implementation details.

#### Architecture

- `Light DOM`: Normal DOM content provided by the application.

- `Shadow DOM`: Internal DOM structure of the component.

- `Shadow Host`: The element to which the Shadow DOM is attached.

- `Shadow Root`: The root of the encapsulated DOM tree.

With open mode, you can access the Shadow Root. With closed mode, it returns null.

*Note: Closed mode is not a security boundary; it only restricts ordinary access through the host's shadowRoot property.*

#### Can global CSS styles penetrate Shadow DOM?

=> Generally, no. However, there are exceptions:
- CSS custom properties (variables) can inherit through the host.
- Properties inherited from the host, such as color and font-family, can affect shadow content.
- The ::part() pseudo-element allows controlled external styling of exposed internal elements.

## HTML Templates and Slots

1. *HTML Template*

Imagine you want to create 100 user cards. You don't want to write the same HTML repeatedly. HTML provides the `<template>` element, which stores reusable HTML that isn't rendered immediately.

```
<template id="user-template">
  <div class="card">
    <h2>User Name</h2>
    <p>User Description</p>
  </div>
</template>
```

2. *What are Slots?*

Imagine a reusable card component. Different applications need different content inside that same card. Instead of hardcoding content, we can use slots.

```
<user-card>
  <h2 slot="title">Megha</h2>
  <p>Frontend Developer</p>
</user-card>
```

shadow DOM container

```
<div>
  <slot name="title"></slot>
  <slot></slot>
</div>
```

Questions:
1. Where will Megha appear?
2. Where will Frontend Developer appear?
3. What happens if we remove `<slot></slot>` from the Shadow DOM?
4. What is the difference between a normal slot and a named slot?

=> Ans: 
1. megha appeared on named slot 
2. frontend developer appear on normal slot.
3. then frontend developer will not be printed.
4. `Default slot`: An unnamed `<slot>` receives child elements that don't specify a slot attribute (unless another matching slot is available). `Named slot`: Receives only the content whose slot attribute matches its name.

## Attributes vs Properties and Custom Events

1. Attributes Vs Properties

```
<user-card name="Megha"></user-card>

this.getAttribute("name");

// in js property
const card = document.querySelector("user-card");

card.name = "Megha";
```

Attributes and properties are related, but they are not automatically synchronized in both directions. You can implement synchronization yourself.

```
class UserCard extends HTMLElement {

  static get observedAttributes() {
    return ["name"];
  }

  get name() {
    return this.getAttribute("name");
  }

  set name(value) {
    this.setAttribute("name", value);
  }

  attributeChangedCallback(name, oldValue, newValue) {
    console.log("Name changed:", newValue);
  }
}

customElements.define("user-card", UserCard);

card.name = "Rahul";
```

2. Custom Events

Imagine your reusable button is used inside another application. When someone clicks it, the parent application needs to know. In React, you might pass an onClick callback. In Web Components, we commonly use CustomEvent.

```
class MyButton extends HTMLElement {

  constructor() {
    super();

    const shadow = this.attachShadow({
      mode: "open"
    });

    shadow.innerHTML = `
      <button>Submit</button>
    `;

    shadow.querySelector("button")
      .addEventListener("click", () => {

        this.dispatchEvent(
          new CustomEvent("user-submit", {
            detail: {
              userId: 101
            },
            bubbles: true,
            composed: true
          })
        );

      });
  }
}

customElements.define("my-button", MyButton);
```

```
document.addEventListener("user-submit", (event) => {
  console.log(event.detail.userId);
});
```

Property | Purpose
---------|--------
detail	| Passes custom data
bubbles | 	Allows event to travel up the DOM tree
composed |	Allows event to cross Shadow DOM boundaries

Q. Suppose we have:

```
<user-card name="Megha"></user-card>

const card = document.querySelector("user-card");
card.setAttribute("name", "Rahul");
card.name = "Amit";

static get observedAttributes() {
  return ["name"];
}

attributeChangedCallback(name, oldValue, newValue) {
  console.log(newValue);
}
```
1. Which values will be printed?

=>  Rahul, Amit

- setAttribute() changes the attribute to Rahul and triggers attributeChangedCallback().
- card.name = "Amit" invokes the setter.
- The setter calls setAttribute() again, triggering the callback with Amit.

If name were an ordinary JavaScript property without this setter, assigning Amit would not automatically trigger the attribute callback.

2. Does changing a property trigger attributeChangedCallback?

=> Property changes don't inherently trigger attribute callbacks. It depends on whether the property setter synchronizes with the attribute.

# Lit framework

Lit is a lightweight library for building Web Components. Instead of manually managing DOM creation, updates, event listeners, and rendering using vanilla JavaScript, Lit provides a declarative programming model.

It provides reactive properties, efficient rendering, HTML templates, and lifecycle methods, which makes component development easier, cleaner, and more maintainable.

```
class UserCard extends LitElement {
  static properties = {
    name: { type: String }
  };

  constructor() {
    super();
    this.name = "Megha";
  }

  render() {
    return html`
      <h2>${this.name}</h2>
    `;
  }
}

const card = document.querySelector("user-card");

card.name = "Rahul";
```

1. name is declared as a reactive property.
2. Initially, name = "Megha".
3. When you execute card.name = "Rahul", Lit detects the property change.
4. Lit schedules an update.
5. The render() method runs again.
6. The UI displays Rahul.

## @property vs @state

Decorator |	Purpose
--------|----------
@property() |	Defines a reactive public property, configurable for attribute reflection
@state() |  Defines internal reactive component state

*Both trigger Lit updates when their values change* 

```
import { LitElement, html } from 'lit';
import { property, state } from 'lit/decorators.js';

class UserCard extends LitElement {

  @property()
  name = "Megha";

  @state()
  private count = 0;

  render() {
    return html`
      <h2>Hello ${this.name}</h2>

      <button @click=${() => this.count++}>
        Count: ${this.count}
      </button>
    `;
  }
}
```

## Lifecycle: constructor, connectedCallback, firstUpdated(), updated()

Method | Purpose
-------|---------
constructor() |	Initializes component
connectedCallback() |	Called when connected to DOM
disconnectedCallback() |	Called when removed from DOM
willUpdate() |	Runs before an update
updated() |	Runs after an update
firstUpdated() | Runs after the first render
render() |	Defines component UI

- Parent needs to configure or control it → @property().
- Only the component needs to manage it → @state().

**Lit batches synchronous property changes into a single update cycle.**

- firstUpdated() — runs only once, after the component's first render.
- updated() — runs after every subsequent completed update triggered by reactive property or state changes.
