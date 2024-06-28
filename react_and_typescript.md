# React + TypeScript

- [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)
- [HTMLElement interface](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)
- [HTML Living Standard](https://html.spec.whatwg.org/multipage/indices.html)

## Table of contents

<!-- toc -->

- [Introduction](#introduction)
- [Props](#props)
- [Style Properties](#style-properties)
- [Children](#children)
- [State](#state)
- [Button components](#button-components)
- [useReducer](#usereducer)
- [useContext](#usecontext)
- [useMemo](#usememo)
- [useCallback](#usecallback)
- [Arrow function syntax with generics](#arrow-function-syntax-with-generics)
- [Forms and events](#forms-and-events)
- [useRef](#useref)
- [forwardRef](#forwardref)
- [Events](#events)
- [HTML element interfaces](#html-element-interfaces)

<!-- tocstop -->

## Introduction

Out of the box, TypeScript supports JSX and you can get full React Web support by adding `@types/react` and `@types/react-dom` to your project. These will get added automatically if you use `npm create vite@latest`.

```json
{
  // ...
  "devDependencies": {
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    // ...
  }
}
```

Every file containing JSX must use the `.tsx` file extension. This is a TypeScript-specific extension that tells TypeScript that this file contains JSX.

## Props 

There are many syntactical ways to handle this. The first is to define the props types separately in a `type` or `interface`:

```tsx
type ExampleProps = {
  message: string;
  color: string;
}

function Example(props: ExampleProps) {
  const { message, color } = props;
  return (
    <div className='Example'>{message} {color}</div>
  );
}

export default Example;
```

or:

```tsx
type ExampleProps = {
  message: string;
  color: string;
}

function Example({ message, color }: ExampleProps) {
  return (
    <div className='Example'>{message} {color}</div>
  );
}

export default Example;
```

The same can be done with an arrow function assignment:

```tsx
type ExampleProps = {
  message: string;
  color: string;
}

const Example = ({ message, color }: ExampleProps) => {
  return (
    <div className='Example'>{message} {color}</div>
  );
}

export default Example;
```

With these methods we are directly annotating the props in the function parameters. This direct annotation is often preferred for its simplicity.

However, you can also use a syntax that uses generic function types:

```tsx
const Example: GenericFn<T> = (arg: T) => {
  return arg;
};
```

For functional components, there is generic type provided by the React type definitions: `React.FC` (or `React.FunctionComponent`, which is an alias for `React.FC`). The `React.FC` generic type alias is used to type a constant that holds a function (whether expressed as an arrow function or a named function expression).

```tsx
type ExampleProps = {
  message: string;
  color: string;
}

const Example: React.FC<ExampleProps> = ({ message, color }) => {
  return (
    <div className='Example'>{message} {color}</div>
  );
}

export default Example;
```

My least favourite approach is when devs will include the prop types inline with the generic, I find this way to cluttered to read:

```tsx
const Example: React.FC<{ message: string; color: string }> = ({ message, color }) => {
  return (
    <div className='Example'>
      {message} {color}
    </div>
  );
};

export default Example;
```

Some developers prefer using `React.FC` because:

- **Explicitness**: `React.FC` explicitly marks the component as a React functional component. Could be helpful especially for new developers or in large codebases.
- **Implicit `children` Prop**: `React.FC` automatically includes the `children` prop in your component, even if you don't explicitly define it. 

## Style Properties

When using inline styles in React, you can use `React.CSSProperties` to describe the object passed to the `style` prop. This type is a union of all the possible CSS properties, and is a good way to ensure you are passing valid CSS properties and to get auto-complete in your editor.

```tsx
function Example() {

  const testStyle: React.CSSProperties = {
    background: '#e9d8e9',
    borderRadius: '3px',
    color: '#292032',
    padding: '.2em'
  };

  return (
    <div className='Example'>
      <p style={testStyle}>Hello.</p>
    </div>
  );
}
```

Note if you're setting css variables this way you will also need to use an *assertion* because TypeScript is expecting valid css property names:

```tsx
  const cssVariables: React.CSSProperties = { '--width': width } as React.CSSProperties;
```

## Children

There are two common paths to describing the children of a component. The first is to use the `React.ReactNode` type, which is a union of all the possible types that can be passed as children in JSX. The second is to use the `React.ReactElement` type, which is only JSX elements and not JavaScript primitives like strings or numbers:

```tsx
interface ModalProps {
  title: string;
  children: React.ReactElement;
}
```

Note, that you cannot use TypeScript to describe that the children are a certain type of JSX elements, e.g. a component which only accepts `<li>` children.

If you define a component using `React.FC` as shown above, then you don't need to include the children in your props type.

## State 

```tsx
import React, { useState, useEffect } from 'react';
// import './Example.css';

type ExampleProps = {
  message: string;
  color: string;
}

function Example({ message, color }: ExampleProps) {
  // state - implicit (inferred) type string
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

In the example above, the inferred type is fine, but a common case where you may want to provide a type is when you have a union. For example, `status` here can be one of a few different strings:

```tsx
type Status = "idle" | "loading" | "success" | "error";

const [status, setStatus] = useState<Status>("idle");
```

Or, as recommended in [Principles for structuring state](https://react.dev/learn/choosing-the-state-structure#principles-for-structuring-state), you can group related state as an object:

```tsx
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success', data: any }
  | { status: 'error', error: Error };

const [requestState, setRequestState] = useState<RequestState>({ status: 'idle' });
```

### useState setter function type

If you are passing the values returned from `useState` to another component, you have to options:

```tsx
'use client';

import { useState } from 'react';
import Drawer from '@/app/_ui/drawer';

export default function DashboardMenu() {
  const [isOpen, setIsOpen] = useState(true);
  return (
      <Drawer open={isOpen} setIsOpen={setIsOpen}>
        <p>content</p>
      </Drawer>
    </>
  );
}

type DrawerProps = {
  children?: React.ReactNode;
  open: boolean;
  setIsOpen: React.Dispatch<React.SetStateAction<boolean>>;
  // setIsOpen: (open: boolean) => void;
};

export default function Drawer({ children }: DrawerProps) {
  return <div className=''>{children}</div>;
}
```

`setIsOpen` can be defined as `setIsOpen: (open: boolean) => void;` or `setIsOpen: React.Dispatch<React.SetStateAction<boolean>>;`

Similarities:

- Both are function types that don't return a value (void).
- Both are intended to be used for updating a boolean state.

Differences:

`React.Dispatch<React.SetStateAction<boolean>>`:

This is the exact type returned by useState for a boolean state.
It's more flexible and allows for both direct value updates and functional updates.
It can accept either a boolean value or a function that receives the previous state and returns a new boolean value.
Example usages:
`setIsOpen(true)` or `setIsOpen(prev => !prev)`


`(open: boolean) => void`:

This is a more specific function type that only accepts a boolean parameter.
It's simpler and more straightforward to understand at a glance.
It only allows for direct value updates, not functional updates.
Example usage: setIsOpen(true)

## Button components

If you build a button component that should accept any valid button attribute as a prop:

```tsx
import clsx from 'clsx';
import styles from './Button.module.css';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  children: React.ReactNode;
}

export function Button({ children, className, ...rest }: ButtonProps) {
  return (
    <button
      {...rest}
      className={clsx(styles.default, className)}
    >
      {children}
    </button>
  );
}
```

## useReducer 

The types for the reducer function are inferred from the initial state. You can optionally provide a type argument to the useReducer call to provide a type for the state, but it is often better to set the type on the initial state instead.

First, my non-TypeScript version:

```jsx
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + 1 };
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </>
  );
}
```

Now in TypeScript:

```tsx
import { useReducer } from 'react';

// Create a type for the state
type State = {
  count: number;
};

// Create a type for the action
type CounterAction =
  | { type: 'reset' }
  | { type: 'increment' }
  | { type: 'decrement' };

const initialState: State = { count: 0 };  // Set the type on the initial state

function reducer(state: State, action: CounterAction) {
  switch (action.type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + 1 };
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
    default: {
      throw new Error('Unknown action'); // We can't assume there is a type property
    }
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </>
  );
}
```

If I had additional properties in my dispatch action object:

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + action.amount };
    }
    case 'decrement': {
      return { count: state.count - action.amount};
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`);
    }
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement', amount: 1})}>-</button>
      <button onClick={() => dispatch({type: 'increment', amount: 1})}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </>
  );
}
```

...then those would need to also be included in the type for the action:

```tsx
type State = {
  count: number;
};

type CounterAction =
  | { type: 'reset' }
  | { type: 'increment', amount: number }
  | { type: 'decrement', amount: number };

const initialState: State = { count: 0 };

function reducer(state: State, action: CounterAction) {
  switch (action.type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + action.amount };
    }
    case 'decrement': {
      return { count: state.count - action.amount };
    }
    default: {
      throw new Error('Unknown action');
    }
  }
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement', amount: 1 })}>-</button>
      <button onClick={() => dispatch({ type: 'increment', amount: 1 })}>+</button>
      <button onClick={() => dispatch({ type: 'reset' })}>reset</button>
    </>
  );
}
```

An explicit alternative to setting the type on the `initialState` is to provide a type argument to `useReducer`:

```tsx
  // ...
  const [state, dispatch] = useReducer<State>(reducer, initialState);
```

## useContext 

Let's start with non-TypeScript:

```jsx
// _contexts/ThemeContext.jsx

import { useState, createContext } from "react";

const ThemeContext = createContext();

function ThemeProvider(props) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{theme, setTheme}}>
      {props.children}
    </ThemeContext.Provider>
  );
}

export {ThemeContext, ThemeProvider};
```

Then in my App.jsx:

```jsx
import { ThemeProvider } from './_contexts/ThemeContext';
import Example from './_components/Example';

function App() {
  return (
    <div className="App">
      <ThemeProvider>
        <Example />
      </ThemeProvider>
    </div>
  );
}

export default App;
```

Then in my components:

```jsx
import { useContext } from 'react';
import { ThemeContext } from '../_contexts/ThemeContext';

function Example() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div className='Example'>
      <p>Theme is: {theme}</p>
    </div>
  );
}

export default Example;
```

Now in TypeScript. Let's first start with an example where we only want to use the `theme` context, not `setTheme`.

```tsx
import React, { useState, createContext } from "react";

type Theme = 'light' | 'dark' | 'system';            // create the type for the context value
const ThemeContext = createContext<Theme>('system'); // provide the type and a default value

type ThemeProps = {                                  // create type for props.children
  children: React.ReactElement;
};

function ThemeProvider(props: ThemeProps) {           // provide type for props
  const [theme, setTheme] = useState<Theme>('light'); // provide type, otherwise it's interred <string>

  return (
    <ThemeContext.Provider value={theme}>
      {props.children}
    </ThemeContext.Provider>
  );
}

export {ThemeContext, ThemeProvider};
```

My App.tsx remains the same:

```tsx
import { ThemeProvider } from './_contexts/ThemeContext';
import Example from './_components/Example';

function App() {
  return (
    <div className="App">
      <ThemeProvider>
        <Example />
      </ThemeProvider>
    </div>
  );
}

export default App;
```

Then in my components:

```tsx
import { useContext } from 'react';
import { ThemeContext } from '../_contexts/ThemeContext';

function Example() {
  // const { theme, setTheme } = useContext(ThemeContext);
  const theme = useContext(ThemeContext);

  return (
    <div className='Example'>
      <p>Theme is: {theme}</p>
    </div>
  );
}

export default Example;
```

**Important:** The type of the value provided by the context is inferred from the value passed to the `createContext` call: `const ThemeContext = createContext<Theme>('system');`.

> `createContext(defaultValue)` defaultValue: The value that you want the context to have when there is no matching context provider in the tree above the component that reads context. If you don’t have any meaningful default value, specify null. The default value is meant as a “last resort” fallback. It is static and never changes over time. [Source](https://react.dev/reference/react/createContext).

Now let's add the `setTheme` function to the provider:

```tsx
import React, { useState, createContext } from "react";

type Theme = 'light' | 'dark' | 'system';
// Create a type that includes the setTheme function
type ThemeContextType = {
  theme: Theme;
  setTheme: (theme: Theme) => void;
};

type ThemeProps = {
  children: React.ReactElement;
};

// update type and defaultValue:
const ThemeContext = createContext<ThemeContextType>({theme: 'system', setTheme: () => {}}); 

function ThemeProvider(props: ThemeProps) { 
  const [theme, setTheme] = useState<Theme>('light');

  return (
    <ThemeContext.Provider value={{theme, setTheme}}>
      {props.children}
    </ThemeContext.Provider>
  );
}

export {ThemeContext, ThemeProvider};
```

The App.tsx remains the same and the components that consume the context can now go back to how they were in the original example:

```tsx
import { useContext } from 'react';
import { ThemeContext } from '../_contexts/ThemeContext';

function Example() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div className='Example'>
      <p>Theme is: {theme}</p>
    </div>
  );
}

export default Example;
```

This technique works when you have a default value - but there are occasionally cases when you do not, and in those cases `null` can feel reasonable as a default value. See the [react docs](https://react.dev/learn/typescript#typing-usecontext) for an example of this.

## useMemo

The useMemo Hooks will create/re-access a memorized value from a function call, re-running the function only when dependencies passed as the 2nd parameter are changed. The result of calling the Hook is inferred from the **return value from the function in the first parameter**. 

```tsx
// The type of visibleTodos is inferred from the return value of filterTodos
const visibleTodos = useMemo(() => filterTodos(todos, filter), [todos, filter]);
```

You can be more explicit by providing a type argument to the Hook:

```tsx
// Explicitly specifying the return type of useMemo as Todo[]
  const visibleTodos = useMemo<Todo[]>(() => filterTodos(todos, filter), [todos, filter]);
```

Here's a fleshed out example:

```tsx
type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

type TodoListProps = {
  todos: Todo[];
  filter: string;
};

function filterTodos(todos: Todo[], filter: string): Todo[] {
  // Implementation of filtering logic
  return todos.filter((todo) => todo.text.includes(filter));
}

const TodoList = ({ todos, filter }: TodoListProps) => {
  // Explicitly specifying the return type of useMemo as Todo[]
  const visibleTodos = useMemo<Todo[]>(() => filterTodos(todos, filter), [todos, filter]);

  return (
    <ul className='TodoList'>
      {visibleTodos.map((todo) => <li>{todo.text}</li>)}
    </ul>
  );
};
```

## useCallback

Like `useMemo`, the function’s type is inferred from the return value of the function in the first parameter, and you can be more explicit by providing a type argument to the Hook. 

When working in TypeScript strict mode `useCallback` requires adding types for the parameters in your callback. This is because the type of the callback is inferred from the return value of the function, and without parameters the type cannot be fully understood.

Depending on your code-style preferences, you could use the `*EventHandler` functions from the React types (see [Events](#events) below) to provide the type for the event handler at the same time as defining the callback:

```tsx
import { useState, useCallback } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  const handleChange = useCallback<React.ChangeEventHandler<HTMLInputElement>>((e) => {
    setValue(e.currentTarget.value);
  }, [setValue])
  
  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>Value: {value}</p>
    </>
  );
}
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
    setName(e.currentTarget.value);
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

## useRef

There may be situations where you need to tell `useRef` what type of element it's being used on by giving it an [html element interface](#html-element-interfaces).

For example, see this modal:

```jsx
function Modal({ children }) {
  const overlay = useRef();
  // ...

  const onClick = useCallback((e) => {
    if (e.target === overlay.current) closeModal();
  }, [closeModal, overlay]);

  return (
    <div ref={overlay} onClick={onClick}>
      {children}
      <button onClick={closeModal}>close</button>
    </div>
  );
};
```

In typescript:

```tsx
function Modal({ children }) {
  const overlay = useRef<HTMLDivElement>(null);
  // ...

  const onClick = useCallback((e: React.MouseEvent<HTMLDivElement>) => {
    if (e.target === overlay.current) closeModal();
  }, [closeModal, overlay]);

  return (
    <div ref={overlay} onClick={onClick}>
      {children}
      <button onClick={closeModal}>close</button>
    </div>
  );
};
```

## forwardRef 

When using the `forwardRef` function to pass down a `ref`, it has to be typed using generics and the `ref` type has to come first even though the the args are in the opposite order:

```tsx
const MyComponent = forwardRef<HTMLDivElement, MyProps>((props, ref) => {
  // Now TypeScript knows `ref` is specifically a `Ref<HTMLDivElement>`
  // and `props` are exactly `MyComponentProps`
  return <div ref={ref}>Hello</div>;
});
MyComponent.displayName = 'MyComponent';

// or using a named function
const MyComponent = forwardRef<HTMLDivElement, MyProps>(function MyComponent(props, ref) {
  return <div ref={ref}>Hello</div>;
});
```

Here's an example from [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forward_and_create_ref/):

```tsx
import { forwardRef, ReactNode } from "react";

interface Props {
  children?: ReactNode;
  type: "submit" | "button";
}
type Ref = HTMLButtonElement;

export const FancyButton = forwardRef<Ref, Props>((props, ref) => (
  <button ref={ref} className="MyClassName" type={props.type}>
    {props.children}
  </button>
));
```

## Events 

See the [full list of events provided in the React types](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/b580df54c0819ec9df62b0835a315dd48b8594a9/types/react/index.d.ts#L1247C1-L1373).

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
