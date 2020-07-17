# FP-TS Cheatsheet

## Option

1. Construct Option from `null | undefined`

<!-- verifier:include-node-module:fp-ts -->
```ts
import * as O from 'fp-ts/lib/Option'

const myVar = undefined
O.fromNullable(myVar)
```
