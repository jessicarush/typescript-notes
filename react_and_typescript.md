# React + TypeScript

- [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)
- [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)

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
const handleSubmit = (event: React.SyntheticEvent) => {
    event.preventDefault();
    // ...
  };
```

If you need to access `e.target.value` you will need to add a generic type of element using elements from the standard HTMLElement interface. For example:

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

## HTML elements

Element | Description
:------ | :---------- | 
HTMLAnchorElement | 
HTMLAreaElement | 
HTMLAudioElement | 
HTMLBRElement | 
HTMLBaseElement | 
HTMLBodyElement | 
HTMLButtonElement | 
HTMLCanvasElement | 
HTMLDListElement | 
HTMLDataElement | 
HTMLDataListElement | 
HTMLDialogElement | 
HTMLDivElement | 
HTMLDocument | 
HTMLEmbedElement | 
HTMLFieldSetElement | 
HTMLFormControlsCollection | 
HTMLFormElement | 
HTMLFrameSetElement | 
HTMLHRElement | 
HTMLHeadElement | 
HTMLHeadingElement | 
HTMLHtmlElement | 
HTMLIFrameElement | 
HTMLImageElement | 
HTMLInputElement | 
HTMLLIElement | 
HTMLLabelElement | 
HTMLLegendElement | 
HTMLLinkElement | 
HTMLMapElement | 
HTMLMediaElement | 
HTMLMetaElement | 
HTMLMeterElement | 
HTMLModElement | 
HTMLOListElement | 
HTMLObjectElement | 
HTMLOptGroupElement | 
HTMLOptionElement | 
HTMLOptionsCollection | 
HTMLOutputElement | 
HTMLParagraphElement | 
HTMLPictureElement | 
HTMLPreElement | 
HTMLProgressElement | 
HTMLQuoteElement | 
HTMLScriptElement | 
HTMLSelectElement | 
HTMLSourceElement | 
HTMLSpanElement | 
HTMLStyleElement | 
HTMLTableCaptionElement | 
HTMLTableCellElement | 
HTMLTableColElement | 
HTMLTableElement | 
HTMLTableRowElement | 
HTMLTableSectionElement | 
HTMLTemplateElement | 
HTMLTextAreaElement | 
HTMLTimeElement | 
HTMLTitleElement | 
HTMLTrackElement | 
HTMLUListElement | 
HTMLUnknownElement | 
HTMLVideoElement | 
 |