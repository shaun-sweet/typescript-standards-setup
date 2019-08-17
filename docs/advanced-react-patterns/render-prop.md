---
id: render-prop
title: Render Prop
---

Do not use an interface for initialState.  Always *infer* the type data from a value when possible

### Examples

<details><summary style="cursor: pointer;">Bad Example (click to see)</summary>
<div>
#### Bad

```tsx
import React from 'react';

interface NameProviderProps { 
  children: (state: NameProviderState) => React.ReactNode; 
}

interface NameProviderState { 
  readonly name: string; 
}

export class NameProvider extends React.Component<NameProviderProps, NameProviderState> {
  readonly state: NameProviderState = { name: 'Piotr' };

  render() {
    return this.props.children(this.state);
  }
}
```

Lets pick this apart a bit.

```tsx
interface NameProviderProps { 
  children: (state: NameProviderState) => React.ReactNode; 
}
```
- interface usage.  
- stick to a similar naming convention "OwnProps" across our components

> TODO: Lets get consensus on this? maybe we will run into issues where this won't be beneficial


```tsx
interface NameProviderState { 
  readonly name: string; 
}
```
- no need to declare a state shape
- you can just declare the value and infer the type from the value
- no need for read only, that comes from the react types inherently
</div>
</details>

#### Good

```tsx
import React from 'react';

type OwnProps = {
  children: (state: typeof initialState) => React.ReactNode;
}

const initialState = {
  name: "Ernie"
}

export class NameProvider extends React.Component<OwnProps, typoc
eof initialState> {
  state = initialState;

  render() {
    return this.props.children(this.state);
  }
}
```
