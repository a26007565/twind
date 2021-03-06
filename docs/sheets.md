# twind/sheets [![Module Size](https://flat.badgen.net/badgesize/brotli/https:/unpkg.com/twind/sheets/sheets.js?icon=jsdelivr&label&color=blue&cache=10800)](https://unpkg.com/twind/sheets/sheets.js 'brotli module size') [![Github](https://flat.badgen.net/badge/icon/tw-in-js%2Ftwind%2Fsrc%2Fsheets?icon=github&label)](https://github.com/tw-in-js/twind/tree/main/src/sheets) [![Typescript](https://flat.badgen.net/badge/icon/included?icon=typescript&label)](https://unpkg.com/browse/twind/sheets/sheets.d.ts)

This module provides [virtualSheet](#virtual-sheet) and [domSheet](#dom-sheet) which can be used with [setup({ sheet })](./setup.md#sheet).

<details><summary>Table Of Contents (Click To Expand)</summary>

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Virtual Sheet](#virtual-sheet)
- [DOM Sheet](#dom-sheet)
- [Custom Sheet Implementation](#custom-sheet-implementation)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
</details>

## Virtual Sheet

The virtual sheet collects style rules into an array. This is most useful during testing (see blow) and [server side rendering (SSR)](./ssr.md).

Additionally it provides an API to reset the current internal state of its `tw` function.

> The example below uses [uvu](https://github.com/lukeed/uvu). Please adjust the test code to your testing framework.

```js
import { suite } from 'uvu'
import * as assert from 'uvu/assert'

import { create } from 'twind'
import { virtualSheet } from 'twind/sheets'

const test = suite('using virtual sheet')

// Setup code to be run once before all tests
test.before((context) => {
  context.sheet = virtualSheet()

  const instance = create({
    sheet: context.sheet,
    // Fail tests on unknown rules or theme values
    mode: 'strict',
    // Prevent preflight rules to be added into sheet
    preflight: false,
    // Do not prefix properties and values
    prefix: false,
  })

  context.tw = instance.tw
})

// Clear the state before each test
test.before.each(({ sheet }) => {
  sheet.reset()
})

test('render one rule', ({ tw, sheet }) => {
  assert.is(tw`text(center)`, 'text-center')
  assert.equal(sheet.target, '.text-center{text-align:center}')
})
```

## DOM Sheet

This sheet uses DOM Text nodes to insert the CSS rules into the stylesheet. Using DOM manipulation makes this way slower than the [default sheet](./setup.md#cssom-sheet) but allows to see the generated CSS in to DOM. Most modern browser display CSS rules from the speedy default sheet using their CSS inspector.

> In production it is advised to use [speedy default sheet](./setup.md#cssom-sheet).

If the `domSheet` is passed no `target` it looks for an style element with the id `__twind`. If no such element is found it will create one and append it to the `document.head`.

```js
import { setup } from 'twind'
import { domSheet } from 'twind/sheets'

setup({ ...sharedOptions, sheet: domSheet() })
```

## Custom Sheet Implementation

In case the builtin sheet implementations do not solve your use case, you can create your own:

```js
import { setup } from 'twind'

const customSheet = (target = []) => ({
  target,
  insert: (rule, index) => {
    // rule: the CSS rule to insert
    // index: the rule's position
    target.splice(index, 0, rule)
  },
})

setup({ ...sharedOptions, sheet: customSheet() })
```

<hr/>

Continue to [Static Extraction (SSR)](./ssr.md)
