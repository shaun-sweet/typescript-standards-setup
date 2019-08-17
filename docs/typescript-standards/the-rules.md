---
id: the-rules
title: The rules
---

## Enums

**Don't**

_Why?_

Enums are not a valid Javascript construct. We want to stay with only features that currently exist in Javascript outside typing features.

We are not using Typescript to have C# everywhere, but using it to leverage its robust typing system.

- It’s not standard/idiomatic JavaScript
- [babel typescript parsing does not support it](https://babeljs.io/docs/en/babel-plugin-transform-typescript)

## `public` accessor within classes

_Don't_

```tsx
class App extends Component {
  public handleChange() {}
  public render() {
    return <div>Hello</div>;
  }
}
```

_Do_

```tsx
class App extends Component {
  handleChange() {}
  render() {
    return <div>Hello</div>;
  }
}
```

_Why?_
All members in a class a public by default which makes this pattern redundant. This is also not valid or idiomatic Javascript.

## `private` accessor within react component

_Don't_

```tsx
class App extends Component {
  private handleChange = () => {};
  render() {
    return (
      <div>
        <input onChange={this.handleChange()} />
      </div>
    );
  }
}
```

_Do_

```tsx
class App extends Component {
  _handleChange = () => {};
  render() {
    return (
      <div>
        <input onChange={this.handleChange()} />
      </div>
    );
  }
}
```

_Better_

```tsx
class App extends Component {
  #handleChange = () => {}
  render() {
    return (
      <div>
        <input onChange={this.handleChange()} />
      </div>
    );
  }
}
```

_Why?_

The `private` accessor won't make properties or methods on your class private during runtime, only during compile time.  
We want to default to idiomatic Javascript and [this syntax is in stage 3](https://github.com/tc39/proposal-class-fields)

## `protected` accessor within react component

_Don't_

```tsx
type OwnProps = {
  config: object;
  who: string;
};

class Base extends Component<OwnProps> {
  protected renderOtherItem(config: OwnProps["config"]) {
    return (
      <>
        <div>Other Item</div>
        <div>{config}</div>
      </>
    );
  }
}

class SomeOtherThing extends Base {
  render() {
    return <div>{this.renderOtherItem(this.props)}</div>;
  }
}
```

_Do_

```tsx
class Base extends Component<{ config: object }> {
  render() {
    return (
      <>
        <div>Other Item</div>
        <div>{this.props.config}</div>
      </>
    );
  }
}

type OwnProps = { who: string } & Base["props"];

class SomeOtherThing extends Component<OwnProps> {
  render() {
    return (
      <div>
        <Base config={this.props.config} />
      </div>
    );
  }
}
```

_Why?_

Using protected is an immediate "RED ALERT" 🚨🚨🚨 in terms of functional patterns leverage with React. There are more effective patterns like this for extending behaviour of some component. You can use:

- just extract the logic to separate component and use it as seen above
- HoC (high order function) and functional composition.
- Render Prop pattern

## `constructor` in class components

_Don't_

```tsx
const initialState = { count: 0 };
type OwnProps = {};

class Counter extends Component<OwnProps, typeof initialState> {
  constructor(props: OwnProps) {
    super(props);
    this.state = initialState;
  }
}
```

_Do_

```tsx
const initialState = { count: 0 };
type OwnProps = {};

class Counter extends Component<OwnProps, typeof initialState> {
  state = initialState;
}
```

_Why?_

There is really no need to use constructor within React Component. It is completely redundant.

#### What if i need to initialize state with some logic

Of course you may ask, what if I need to introduce some logic to initialize component state, or even to initialize the state from some dependant values, like props for example.
The answer to your question is rather straightforward.
Just define a pure function outside the component with your custom logic (as a “side effect” you’ll get easily tested code as well 👌).

## Decorators

_Don't_

```tsx
@connect(mapStateToProps)
export class Container extends Component {}
```

_Do_

```tsx
class Container extends Component {}
export connect(mapStateToProps)(Container)
```

_Why?_

- You won’t be able to get original/clean version of your class.
- TypeScript uses old version of decorator proposal which isn’t gonna be implemented in ECMAscript standard 🚨.
- It adds additional runtime code and processing time execution to your app.
- What is most important though, in terms of type checking within JSX, is, that decorators don’t extend class type definition. That means (in our example), that our Container component, will have absolutely no type information for consumer about added/removed props.

## Use lookup types for accessing component state/props

[lookup types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-1.html#keyof-and-lookup-types)

_Don't_

- export component prop types

_Do_

```tsx
// counter.tsx
type State = { counter: number };
type OwnProps = { someProps: string };

export class Counter extends Component<OwnProps, State> {}

// consumer.tsx

import { Counter } from "./counter";

type TypeYouWant = Counter["props"] & {};

class Consumer extends Component<TypeYouWant, {}> {
  render() {
    return <Counter />;
  }
}
```

## React `children`

_Don't_

```jsx
type OwnProps = {
  children: ReactNode
};
```

_Do_

```jsx
type OwnProps = {};
const Component: React.FC<OwnProps> = props => {
  return <div />;
};
```

_Why?_

- allow the react types to add the type data

## Use type inference for defining Component State

_Don't_

```tsx
type State = { count: number };
type OwnProps = { someProps: string };

class Counter extends Component<OwnProps, State> {
  state = {
    count: 0
  };
}
```

_Do_

```tsx
type State = typeof initialState;
type OwnProps = { someProps: string };
const initialState = { count: 0 };

class Counter extends Component<OwnProps, State> {
  state = initialState;
}
```

_Why?_

- Type information is always synced with implementation as source of truth is only one thing 👉 THE IMPLEMENTATION! 💙
- Less type boilerplate
- More readable code

## Use default import to import `React`

_Don't_

```tsx
import * as React from "react";
```

_Do_

```tsx
import React, { Component } from "react";
```

To support recommended behaviour you need to set following config within your `tsconfig.json` file:

```json
{
  "compilerOptions": {
    /* 
       Enables emit interoperability between CommonJS and ES Modules 
       via creation of namespace objects for all imports. 
       Implies 'allowSyntheticDefaultImports'. 
    */
    "esModuleInterop": true
  }
}
```

## Don't use `namespace`

- with ES2015 modules, we do not need them anymore
- cannot be used with [babel for transpiling](https://babeljs.io/docs/en/babel-plugin-transform-typescript)

## Declare types before run-time implementation

_Don't_

```tsx
import { Component } from "react";

const initialState = {
  count: 0
};

type State = typeof initialState;
type OwnProps = { count?: number };

class Counter extends Component<OwnProps, State> {
  state = initialState;
}
```

_Do_

```tsx
import { Component } from "react";

// Types first
type State = typeof initialState;
type OwnProps = { count?: number };

// Runtime second!
const initialState = {
  count: 0
};

class Counter extends Component<OwnProps, State> {
  state = initialState;
}
```

_Why?_
- first lines of document clearly state what kind of types are used within current module. Also those types are compile only code
- run-time and compile time declarations are clearly separated
- in component user immediately knows what the component “API” looks like without scrolling

## Don’t use `number` for indexable type key

_Don't_

```tsx
interface UserIndexedDictionary {
  [id: number]: User;
}

interface User {
  email: string;
}

const dictionary: UserIndexedDictionary = {
  1: {
    email: "lets@go.com"
  },
  2: {
    email: "no@go.com"
  }
};
```

_Do_

```tsx
interface UserIndexedDictionary {
  [id: string]: User;
}

interface User {
  email: string;
}

const dictionary: UserIndexedDictionary = {
  dgdsafasd: {
    email: "lets@go.com"
  },
  jykytes: {
    email: "no@go.com"
  }
};
```

_Why_

- In JavaScript object properties are always typeof string! don't create false type predicates within your apps!

## Don’t use `JSX.Element` to annotate function/component return type or children/props

_Don't_

```tsx
type Data = {
  id: string
  email: string
  age: number
}

const renderListHelper = (data: Data[]): JSX:Element => {
  return (
    <div>
      {
        data.map((item) => (
          <div>{item.email}</div>
        ))
      }
    </div>
  )
}
```

_Do_

```tsx
type Data = {
  id: string
  email: string
  age: number
}

const renderListHelper = (data: Data[]): ReactElement => {
  return (
    <div>
      {
        data.map((item) => (
          <div>{item.email}</div>
        ))
      }
    </div>
  )
}
```
_Why?_

- explicit types over generalized ones

## Use type alias instead of `interface` for declaring Props/State

_Don't_

```tsx
interface State {
  counter: number;
}

interface OwnProps {
  color: string;
}

class Counter extends Component<OwnProps, State> {}
```

_Do_

```tsx
type State = {
  counter: number;
};

type OwnProps = {
  color: string;
};

class Counter extends Component<OwnProps, State> {}
```

_Why?_
 
- consistency/clearness
- Let's say we use the interface.  If you would like to use the interface and also consider implementation = source of truth, you're out of luck as you cannot do that.

```tsx
// $ExpectError ❌
interface State extends typeof initialState {}
const initialState = {
  counter: 0
}
```