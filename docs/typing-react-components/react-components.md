---
id: react-components
title: Typing Components
sidebar_label: Typing Components
---
> TODO: Pull this out into the cheatsheet detailing -> React Functional Component, React Class Component, with hooks etc.  In addition have proper tooling (vscode extension) to support these patterns
## Interfaces
If type inference can be used, do not use an interface. This creates a better developer experience by providing better editor hints that contain the entire object shape

### Examples
<details><summary style="cursor: pointer;">Bad Example (click to see)</summary>
<p>


#### Bad
```tsx
import React from 'react';

interface OwnProps  { // << Avoid interfaces
  message: string
};

export const NoInterfaceComponent: React.FC<OwnProps> = props => {
  const { children, ...restProps } = props;

  return <div {...restProps}>{children}</div>;
};
```
</p>
</details>

#### Good

```tsx
import React from 'react';

type OwnProps = {
  message: string
};

export const NoInterfaceComponent: React.FC<OwnProps> = props => {
  const { children, ...restProps } = props;

  return <div {...restProps}>{children}</div>;
};
```

## Initial State

Do not use an interface for initialState.  Always *infer* the type data from a value when possible

### Examples

<details><summary style="cursor: pointer;">Bad Example (click to see)</summary>

#### Bad

```tsx
import React from 'react';

type OwnProps = {
  label: string;
};

type initialState = { // <<<<< bad
  count: number;
};

export class ClassCounter extends React.Component<OwnProps, initialState> {
  state = { count: 0 };

  handleIncrement = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    const { handleIncrement } = this;
    const { label } = this.props;
    const { count } = this.state;

    return (
      <div>
        <span>
          {label}: {count}
        </span>
        <button type="button" onClick={handleIncrement}>
          {'Increment'}
        </button>
      </div>
    );
  }
}

```
</details>

#### Good

```tsx
import React from 'react';

type OwnProps = {
  label: string;
};

const initialState = {
  count: 0
};

export class ClassCounter extends React.Component<OwnProps, typeof initialState> {
  state = initialState;

  handleIncrement = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    const { handleIncrement } = this;
    const { label } = this.props;
    const { count } = this.state;

    return (
      <div>
        <span>
          {label}: {count}
        </span>
        <button type="button" onClick={handleIncrement}>
          {'Increment'}
        </button>
      </div>
    );
  }
}
```



