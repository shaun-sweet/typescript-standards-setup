---
id: overriding-styled-components-theme
title: Usage with theme & styled-components
---

Next, we will walk you thru overriding the default styled components theme to get intellisense within your styled components.

## Creating the typings folder

In the root of your project make a folder called `typings`.  It must be labeled typings because the `tsconfig.json` file you copied in the previous section points to this. 


## Making the type definition
Make a new file in this folder named `styled-components.d.ts`

Inside here lets grab our theme type defs and override styled-components default theme

```typescript
import {YourTheme} from '@your/theme'

type ThemeInterface = typeof YourTheme;

declare module "styled-components" {
  interface DefaultTheme extends ThemeInterface {}
}
```

## Success

Now you should have type definitions and auto completion in your styled component callbacks to theme :)