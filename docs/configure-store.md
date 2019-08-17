---
id: configure-store
title: Configuring redux store
---
## Install required dependencies

```sh
yarn add typesafe-actions @types/redux-thunk
```
or npm
```sh
npm i typesafe-actions @types/redux-thunk -S
```
## Root Types
Through our application we need a top level type to be automatically inferred for things like the entire redux state or all the available actions to dispatch.

This can be an awful process where you model the entire redux state tree, but we won't do that.  That is against our philosophy.  Instead lets infer them.

We will start by going back to our `typings` folder at the root of our project.

Make a new file named `modules.d.ts` and add the following

```ts
// typings/modules.d.ts
declare module 'RootTypes';
```

#### Redux thunk overrides
Next we will override redux-thunk with a custom type to play nice.

Again in the `typings` folder lets create a folder named `redux-thunk` and add a file in that folder `index.d.ts`

Add to that file the following snippet:
```ts

import {
  Action,
  ActionCreatorsMapObject,
  AnyAction,
  Dispatch,
  Middleware,
} from 'redux';

export interface ThunkDispatch<S, E, A extends Action> {
  <R>(thunkAction: ThunkAction<R, S, E, A>): R;
  <T extends A>(action: T): T;
}

export type ThunkAction<R, S, E, A extends Action> = (
  dispatch: ThunkDispatch<S, E, A>,
  getState: () => S,
  extraArgument: E
) => R;

/**
* Takes a ThunkAction and returns a function signature which matches how it would appear when processed using
* bindActionCreators
*
* @template T ThunkAction to be wrapped
*/
export type ThunkActionDispatch<
T extends (...args: any[]) => ThunkAction<any, any, any, any>
> = (...args: Parameters<T>) => ReturnType<ReturnType<T>>;

export type ThunkMiddleware<
S = {},
A extends Action = AnyAction,
E = undefined
> = Middleware<ThunkDispatch<S, E, A>, S, ThunkDispatch<S, E, A>>;

declare const thunk: ThunkMiddleware & {
withExtraArgument<E>(extraArgument: E): ThunkMiddleware<{}, AnyAction, E>;
};

export default thunk;

/**
* Redux behaviour changed by middleware, so overloads here
*/
declare module 'redux' {
  /**
   * Overload for bindActionCreators redux function, returns expects responses
   * from thunk actions
   */
  function bindActionCreators<M extends ActionCreatorsMapObject<any>>(
      actionCreators: M,
      dispatch: Dispatch
  ): {
      [N in keyof M]: ReturnType<M[N]> extends ThunkAction<any, any, any, any>
          ? (...args: Parameters<M[N]>) => ReturnType<ReturnType<M[N]>>
          : M[N]
  };
}
```

This looks a bit nuts, but thats fine.  The typings folder is not something we interact with on a daily basis so the typical Clean Code standards we won't really apply.  We are mostly interested in type saftey.

## Configuring the RootTypes

Inside whatever folder you redux store configuration lives add the file `types.d.ts`

Add another file named `rootAction.ts`

This file *needs* to collect all the actions in your app.  So import every action in your app into this file and export them back out... Something like this for example:

```ts
import * as personalDetailsActions from "../tabs/PersonalDetails/ducks/actions";
import { screenResize} from '../store/middleware/screenResize'


export default {
  personalDetailsActions,
  screenResize
}
```

Now that we have our `rootAction.ts` set-up lets go back to our `types.d.ts` in the store folder.  Add the following:

```ts
import { StateType, ActionType } from "typesafe-actions";


declare module "RootTypes" {
  export type Store = StateType<typeof import("./configureStore").default>;
  export type Actions = ActionType<typeof import("./rootAction").default>;
  export type State = ReturnType<StateType<typeof import("./rootReducer").default>>;
}

```

These will serve to collect up our type information and expose it to the entire application at any layer.


We're done configuring it!  But it's not much use without some type data... let's do that