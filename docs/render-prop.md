---
id: render-prop
title: Render Prop
---

Do not use an interface for initialState.  Always *infer* the type data from a value when possible

### Examples

<details><summary style="cursor: pointer;">Bad Example (click to see)</summary>

#### Bad

```tsx
import React from 'react';

type OwnProps = {
  label: string;
};

type initialState = {
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
