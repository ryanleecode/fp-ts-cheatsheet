# FP-TS Cheatsheet

## Begin, Commit, Rollback

<!-- verifier:include-node-module:fp-ts -->
```ts
import { pipe } from 'fp-ts/lib/function'
import * as TE from 'fp-ts/lib/TaskEither'

declare function begin(): Promise<void>
declare function commit(): Promise<void>
declare function rollback(): Promise<void>

pipe(
  TE.bracket(
    TE.tryCatch(
      () => begin(),
      (err) => new Error(`begin txn failed: ${err}`),
    ),
    () =>
      TE.tryCatch(
        () => commit(),
        (err) => new Error(`commit txn failed: ${err}`),
      ),
    (_, either) =>
      pipe(
        either,
        TE.fromEither,
        TE.orElse(() =>
          TE.tryCatch(
            () => rollback(),
            (err) => new Error(`rollback txn failed: ${err}`),
          ),
        ),
      ),
  ),
)
```

## Convert Multiple Eithers into a Single Either

<!-- verifier:include-node-module:fp-ts -->
```ts
import * as E from 'fp-ts/lib/Either'
import { sequenceS, sequenceT } from 'fp-ts/lib/Apply'

declare function func1(): E.Either<Error, number>
declare function func2(): E.Either<Error, string>
declare function func3(): E.Either<Error, object>

// Produces an Object { a: number., b: string, c: object }
sequenceS(E.either)({
  a: func1(),
  b: func2(),
  c: func3(),
})

// Produces a Tuple [number, string, object]
sequenceT(E.either)(func1(), func2(), func3())
```

## Check for Errors in an Array

<!-- verifier:include-node-module:fp-ts -->
```ts
import * as A from 'fp-ts/lib/Array'
import * as E from 'fp-ts/lib/Either'
import { sequenceS, sequenceT } from 'fp-ts/lib/Apply'

const arr = [1,2,3]

// Type signature is E.Either<Error, number[]>
A.array.traverse(E.either)(arr, (val) => {
  if (val === 2) {
    return E.left(new Error('2 is invalid'))
  }

  return E.right(val)
})
```

## Handle a TaskEither Error with another TaskEither

<!-- verifier:include-node-module:fp-ts -->
```ts
import { pipe } from 'fp-ts/lib/function'
import { IOEither } from 'fp-ts/lib/IOEither'
import {
  swap,
  TaskEither,
  chainFirstW,
  fromIOEither,
  chain,
} from 'fp-ts/lib/TaskEither'

declare const log: (message: string, err?: Error) => IOEither<Error, void>

declare const foo: TaskEither<Error, string>

const handleError = <E, U>(fe: (e: E) => TaskEither<E, U>) => <A>(
  te: TaskEither<E, A>,
) => {
  return pipe(
    // 1. Put the error on the right side of the railway
    swap(te),
    // 2. Perform an action on the error, if there is one
    chainFirstW(fe),
    // 3. Put the error back on the left side
    swap,
    // 4. Place the original TaskEither back on the railway
    chain(() => te),
  )
}

pipe(
  foo,
  handleError((err) => pipe(log('Execution failed', err), fromIOEither)),
) // TaskEither<Error, string>
```
