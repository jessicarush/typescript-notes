# React + TypeScript

- [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)
- [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)
- [HTML Living Standard](https://html.spec.whatwg.org/multipage/indices.html)

## Table of contents

<!-- toc -->

- [Introduction](#introduction)
- [Props](#props)
- [State](#state)
- [Arrow function syntax with generics](#arrow-function-syntax-with-generics)
- [Forms and events](#forms-and-events)
  * [Events](#events)
- [HTML elements](#html-elements)

<!-- tocstop -->

## Introduction

- use the `.tsx` extension


## Props 

```tsx
import React, { useState, useEffect } from 'react';

type ExampleProps = {
  message: string;
  color: string;
}

function Example(props: ExampleProps) {
  const { message, color } = props;
  // state - implicit type string

  return (
    <div className='Example'>
      {/* Here's my comment */}
    </div>
  );
}

export default Example;
```

or 

```tsx
import React, { useState, useEffect } from 'react';
// import './Example.css';

type ExampleProps = {
  message: string;
  color: string;
}

function Example({ message, color }: ExampleProps) {

  return (
    <div className='Example'>
      {/* Here's my comment */}
    </div>
  );
}

export default Example;
```

## State 

```tsx
import React, { useState, useEffect } from 'react';
// import './Example.css';

type ExampleProps = {
  message: string;
  color: string;
}

function Example({ message, color }: ExampleProps) {
  // state - implicit type string
  const [name, setName] = useState('');
  // state - explicit type string
  // const [name, setName] = useState<string>('');

  const updateName = (e: React.ChangeEvent<HTMLInputElement>) => {
    setName(e.target.value);
  };

  const onSubmit = () => {
    console.log(`name: ${name}`);
  };

  return (
    <div className='Example'>
      <input type='text' value={name} onChange={updateName} />
      <button onClick={onSubmit}>ok</button>
    </div>
  );
}

export default Example;
```

## Arrow function syntax with generics 

Normally we would write a function with generics like this:

```typescript
// Function declaration
function someFunction<T>(arg: T) {
  console.log(arg);
}
// Arrow function syntax (function expression)
const otherFunction = <T>(arg: T) => {
  console.log(arg);
};
```

However, in `tsx` files, the compiler gets confused by the `<T>` syntax in arrow functions and will report an error. To fix this, either add a comma `,` or extend unknown:

```tsx
// ✅ Function declaration
function someFunction<T>(arg: T) {
  console.log(arg);
}
// ❌ Arrow function syntax (function expression)
// const otherFunction = <T>(arg: T) => {
//   console.log(arg);
// };
// ✅ Arrow function syntax (function expression)
const otherFunction = <T,>(arg: T) => {
  console.log(arg);
};
// ✅ Arrow function syntax (function expression)
const otherFunction = <T extends unknown>(arg: T) => {
  console.log(arg);
};
```

## Forms and events 

```tsx
// SyntheticEvent is the base event for all other (more specific) events
const handleSubmit = (event: React.SyntheticEvent) => {
    event.preventDefault();
    // ...
  };
```

If you need to access `e.target.value` you will need to add a generic type of element using elements from the standard [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement). For example:

```tsx
React.ChangeEvent<HTMLInputElement>
React.ChangeEvent<HTMLTextAreaElement>
React.ChangeEvent<HTMLInputSelect>
```

See the HTML elements section below for more.

```tsx
import React, { useState, useEffect } from 'react';
// import './Example.css';

type ExampleProps = {
  message: string;
  color: string;
};

function Example({ message, color }: ExampleProps) {
  const [name, setName] = useState('');

  const updateName = (e: React.ChangeEvent<HTMLInputElement>) => {
    setName(e.target.value);
  };

  const onSubmit = () => {
    console.log(`name: ${name}`);
  };

  return (
    <div className='Example'>
      <p style={{ color: color }}>{message} {name || '...'}</p>
      <input type='text' value={name} onChange={updateName} />
      <button onClick={onSubmit}>ok</button>
    </div>
  );
}

export default Example;
```

### Events 

Event Type | Description
:--------- | :----------
AnimationEvent | CSS Animations.
ChangeEvent | Changing the value of `<input>`, `<select>` and `<textarea>` elements.
ClipboardEvent | Using copy, paste and cut.
CompositionEvent | Events that occur due to the user indirectly entering text (e.g. depending on Browser and PC setup, a popup window may appear with additional characters if you want to type Japanese on a US Keyboard)
DragEvent | Drag and drop interaction with a pointer device (e.g. mouse).
FocusEvent | Event that occurs when elements get or loses focus.
FormEvent | Event that occurs whenever a form or form element gets/loses focus, a form element value is changed or the form is submitted.
InvalidEvent | Fired when validity restrictions of an input fails (e.g `<input type="number" max="10">` and someone would insert number 20).
KeyboardEvent | User interaction with the keyboard. Each event describes a single key interaction.
MouseEvent | Events that occur due to the user interacting with a pointing device (e.g. mouse)
PointerEvent | Events that occur due to user interaction with a variety pointing of devices such as mouse, pen/stylus, a touchscreen and which also supports multi-touch. Unless you develop for older browsers (IE10 or Safari 12), pointer events are recommended. Extends UIEvent.
TouchEvent | Events that occur due to the user interacting with a touch device. Extends UIEvent.
TransitionEvent | CSS Transition. Not fully browser supported. Extends UIEvent.
UIEvent | Base Event for Mouse, Touch and Pointer events.
WheelEvent | Scrolling on a mouse wheel or similar input device. (Note: wheel event should not be confused with the scroll event).
SyntheticEvent | The base event for all above events. Should be used when unsure about event type.

## HTML element interfaces

See the [HTML Living Standard](https://html.spec.whatwg.org/multipage/indices.html) for a tidy list. 

I couldn't really find a list like this on MDN. MDN has a complete [elements list](https://developer.mozilla.org/en-US/docs/Web/HTML/Element) but it doesn't show the interfaces. The closest thing to an interface list is if you go to the [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement), then look at the **Related pages for HTML DOM** section in the sidebar.

Element | Description
:------ | :----------
HTMLElement | `<abbr>`, 
HTMLAnchorElement | `<a>`
HTMLAreaElement | `<area>`
HTMLAudioElement | `<audio>`
HTMLBRElement | `<br>`
HTMLBaseElement | `<base>`
HTMLBodyElement | `<body>`
HTMLButtonElement | `<button>`
HTMLCanvasElement | `<canvas>`
HTMLDListElement | `<dl>`
HTMLDataElement | `<data>`
HTMLDataListElement | `<datalist>`
HTMLDialogElement | `<dialog>`
HTMLDivElement | `<div>`
HTMLEmbedElement | `<embed>`
HTMLFieldSetElement | `<fieldset>`
HTMLFormControlsCollection | The HTMLFormControlsCollection interface is used for collections of listed elements in form elements.
HTMLFormElement | `<form>`
HTMLFrameSetElement | The frameset element acts as the body element in documents that use frames.
HTMLHRElement | `<hr>`
HTMLHeadElement | `<head>`
HTMLHeadingElement | `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`
HTMLHtmlElement | `<html>`
HTMLIFrameElement | `<iframe>`
HTMLImageElement | `<img>`
HTMLInputElement | `<input>`
HTMLLIElement | `<li>`
HTMLLabelElement | `<label>`
HTMLLegendElement | `<legend>`
HTMLLinkElement | `<link>`
HTMLMapElement | `<map>`
HTMLMediaElement | `<audio>`, `<video>`
HTMLMetaElement | `<meta>`
HTMLMeterElement | `<meter>`
HTMLModElement | `<del>`, `<ins>`
HTMLOListElement | `<ol>`
HTMLObjectElement | `<object>`
HTMLOptGroupElement | `<optgroup>`
HTMLOptionElement | `<option>`
HTMLOptionsCollection | represents a collection of `<option>` elements. This object is returned only by the options property of `select`.
HTMLOutputElement | `<output>`
HTMLParagraphElement | `<p>`
HTMLPictureElement | `<picture>`
HTMLPreElement | `<pre>`
HTMLProgressElement | `<progress>`
HTMLQuoteElement | `<q>`
HTMLScriptElement | `<script>`
HTMLSelectElement | `<select>`
HTMLSourceElement | `<source>`
HTMLSpanElement | `<span>`
HTMLStyleElement | `<style>`
HTMLTableCaptionElement | `<caption>`
HTMLTableCellElement | `<td>`, `<th>`
HTMLTableColElement | The HTMLTableColElement interface provides properties for manipulating single or grouped table column elements.
HTMLTableElement | `<table>`
HTMLTableRowElement | `<tr>`
HTMLTableSectionElement | `<thead>`
HTMLTemplateElement | `<template>`
HTMLTextAreaElement | `<textarea>`
HTMLTimeElement | `<time>`
HTMLTitleElement | `<title>`
HTMLTrackElement | `<track>`
HTMLUListElement | `<ul>`
HTMLUnknownElement | The HTMLUnknownElement interface represents an invalid HTML element and derives from the HTMLElement interface, but without implementing any additional properties or methods.
HTMLVideoElement | `<video>`

All other elements use the base `HTMLElement` interface.

---
