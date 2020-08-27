---
title: Option and Monocle-ts
---

## FP foundation

When referring to Functional Programming (FP), people usually mean programming paradigm where the function is the main building block of the programs (for example in Object Oriented porgramming the main building block is an object).
As you know, programs are usually cosntructed by creating and combining variables, passing them to the methods, functions and procedures.
Since function is the main building block in FP, we need to be able to store functions in variables, pass them to other functions and return them from functions. Programming language providing this capabilities usually referred to as having **first-class functions**.

1. Storing functions to variables and passing them to other functions:

```javascript
// function decalration
export function add1Decl(a) {
  return a + 1
}

;[1, 2, 3].map(add1Decl)
// Array.prototype.forEach
// Array.prototype.filter
// Array.prototype.reduce

export const one = 1

// function expressions
export const add1Expr = function add1Expr(a) {
  return a + 1
}
;[1, 2, 3].map(add1Expr)

// same with more fancy arrow functions
export const add1ExprArr = a => a + 1
;[1, 2, 3].map(add1ExprArr)

// You can also use expressions immediately
;[1, 2, 3].map(function(a) {
  return a + 1
})
```

2. Returning functions from functions:

```typescript
function mapNums(mapper: (value: number) => any) {
  return function(items: Array<number>) {
    return items.map(mapper)
  }
}

// incrementItems is also a plain old function expecting an array of integers
const incrementItems = mapNums(a => a + 1)

test("incrementItems is a function and it does it's thing", () => {
  expect(incrementItems).toBeInstanceOf(Function)

  const incremented = incrementItems([1, 2, 3])

  expect(incremented).toEqual([2, 3, 4])
})
```

The idea of returning functions from functions is also knows as _currying_, which in turn means **postponing function evaluation**. Take a look at the example:

```javascript
// currying
const add = (a, b) => a + b

const addCurried = a => b => a + b

const add1 = addCurried(1)

test("add1", () => {
  expect(add1(10)).toBe(11)
  expect(add1(20)).toBe(21)
})
```

`add` function is **eager** in sense that it evaluates the logic stored in the body of the function immediately as it receives both of it arguments.

```javascript
add(1, 2) // immediately returns 3
```

`addCurried` on the other hand is **lazy**, meaning it does not evaluate until some codition is fullfilled. In this case it will not evaluate until received two arguments

```javascript
// you can pass two arguments right away
addCurried(1)(2)

// which is equivalent to
add1 = addCurried(1) // add1 is a function that exepcts one argument
add1(2)

// another benifit is that add1 function is now reusable and you could use it somewhere else!
;[1, 2, 3].map(add1) // returns [2, 3, 4]
```

Currying is heavily used in most of the functional libraries such as `Ramda`, `crocks`, `ft-ts` and many more. In some languages, such as Haskell all the functions are curried by default

### Functional domain modelling

Standard approach to model the data is to consolidate all the meaningful states in one interface and process it depending on the business logic. Here's a small example of how we could model loading state:

```typescript
interface DataFetchState {
  transactions?: Array<{ date: Date; amount: number }>
  isLoading: boolean
  timeoutMs?: number // it becomes even more interesting if we want to add other cases
  error?: string
}
```

Looks reasonable right? And here's what our possible states might look like:

```typescript
export const notFetchedState: DataFetchState = {
isLoading: false
};

export const loadingState: DataFetchState = {
isLoading: true
};

export const loadedState: DataFetchState = {
isLoading: false,
transactions: [{ date: new Date(), amount: 123 }]
};

export const errorState: DataFetchState = {
isLoading: false,
error: "Something went wrong"
};

export const timedOutState: DataFetchState = {
    isLoading: false
    timeoutMs: 3000
}
```

Typescript will prevent us from creating incorrect states or introducing typos in our code:

```typescript
// this won't compile
export const timedOutState: DataFetchState = {
    isLoading: "false" // should be bool
    timeoutms: 3000 // no such key `timeoutms`, should be `timeoutMs`
}
```

Though totally able to prevent the basic errors, typescript, however is not smart enough to prevent us from creatin **unsound** states, which are correct from the type definition side, but impossible or should be impossible according to business domain. Take a look at these examples:

```typescript
// pretty strange to have loading state while we still have a timeout
export const timedOutStateButStillLoading: DataFetchState = {
    isLoading: true
    timeoutMs: 3000
}

// super strange state, but can totaly hape in runtime
export const stateThatMakesNoSense = {
transactions: [{ date: new Date(), amount: 123 }],
isLoading: true,
error: "I am crazy"
};

// having a function like this will return us transactions even from such strange state
// we would have to cover all the strange cases with unit tests and we'll have to remember to cover them
export const getTransacts = (state: DataFetchState) => state?.transactions
```

There's a better way! Since we're using typescript we could try to use it's features to help us and that's where union types come into play

A small refresher on what union types are:

```typescript
// primitive types
type StrOrNum = string | number

export const num: StrOrNum = 1
export const str: StrOrNum = "foo"

// class instances
class Foo {}
class Bar {}

type FooBar = Foo | Bar
export const foobar: FooBar = new Foo()

// Another version of this pattern is a so called 'discriminated union'
// a field shared by all the instances is called a discriminant
type DiscrFooBar = { tag: "foo" } | { tag: "bar" }
```

### ADT

In FP there is concept an Algebraic Data Type (ADT). ADTs are divided into _Sum_ and _Product Types_ which are differentiated by the number of inhabitant types (sometimes also called _Arity_)

#### Sum types

`DiscrFooBar` is a Sum type, the number of its inhabitant types is equal to the sum of independent instances

```typescript
export const foo: DiscrFooBar = { tag: "foo" }
export const bar: DiscrFooBar = { tag: "bar" }

export const fizz: DiscrFooBar = { tag: "fizz" } // compile error - typescript won' let us represent unsupported state
```

The number of inhabitants of a Sum type is, pun intended, equal to sum of separate unique instances of it's subtypes. In case of `DiscrFooBar` it is `1 + 1 = 2`. The **arity** of `DiscrFooBar` is 2.

#### Product types

A product type in typescript can be represented by a tuple or by an object

```typescript
type TupleProduct = [number, string]

type ObjectProduct = {
  a: number
  b: string
}
```

The arity of product type is determined by multiplication of the number of separate instances a product type can have.
The amount of separate instances in case of `TupleProduct` can be approximated as follows:

```typescript
const OneAndString: TupleProduct = [1, ""]
const TwoAndString: TupleProduct = [2, ""]
// and so on until Number.POSITIVE_INFINITY

// Same goes to strings
const OneAndFoo: TupleProduct = [1, "foo"]
const OneAndBar: TupleProduct = [1, "bar"]
// and so on well for all other possible strings, which is effectively and infinitely big amount
```

So the arity of `TupleProduct` can be approximated by the following formula:

```javascript
const tupleProdArity =
  (Number.POSITIVE_INFINITY - Number.NEGATIVE_INFITIY) * Number.PositiveInfitiy
```

Not very helpful, but at least gives the understanding that the usual amount of possible cases is much bigger than for sum types.
In practice it means that along with expected combinations you'll get un-expected and unwanted combintaions, as you've seen in the example with `DataFetchState`.

### ADT in real life

With all said above, Let's try to model loading state using union types

```typescript
export type BetterDataFetchState =
| {
type: "noFetched";
}
| {
type: "loading";
}
| {
type: "loadingError";
message: string;
}
| {
  type "loaded";
  transactions: Array<{ date: Date, amount: number }>
}
```

Here are the upsides of this approach:

1. The number of possible states is decreased - only 4 possible separate states
2. Unsound types are impossible:

```typescript
// this will be a compiler error
const strangeType: BetterDataFetchState = {
  type: "loading";
  transactions: [{ date: new Date(), amount: 123 }]
  message: "I am crazy";
}
```

3. We get type guarding and type inference to work for us:

- we get auto-completion for `"loaded", "notFetched", "loading"` and `"loadingError" strings
- we get compile-time checking for typos and accessible properties
- with helper function `assertUnreachable` we make sure that we explicitly handled all the possible cases

```typescript
const assertUnreachable = (shouldNotArriveHere: never) => {
  throw new Error("non-exhaustive checking of BetterDataFetchState")
}

export function getTransactions(state: BetterDataFetchState) {
  switch (state.type) {
    case "loaded":
      return state.transactions

    case "notFetched":
    case "loading":
    case "loadingError":
      return []

    default:
      assertUnreachable(state)
  }
}
```

Functional programming relies heavily on these concepts and uses Sum types in a lot of patterns. One of them is handling the absence of the value also knows as Option (sometimes refferred to as Maybe)

## Option

There most popular and arguably the worst way (google for `A billion dollar mistake`) to handle possibly absent values in the notion of _nullability_. In most of the popular languages there is a special value (null, undefined, Nil, nil) to denote the absence of the value. Usually it means that you should check while trying to access it's value or call it (remember you favourite `undefined is not a function`???? I know you do). In staticly typed languages it's usually checked by a compiler and the program will no compile until you provide an evidence that the value is in fact there.
Though it's completely viable option it has it's problems:

- Even compiler is not able to prevent `NullPointerException`s to happen in runtime in some cases
- Making sequential computations with possibly nullable values becomes a burden and it's easy to overlook something in the dynamic language
- It makes a bit of a **leaky** abstraction, where you can no longer program to interface and still have to focus on implementation details

Since FP is all about reusable and composable pieces of code, there is a notion of _Option_.
The implementation can be anything and you don't need to know anyhting about it, you just need to know that it is some container that can have a value of some type inside.

We could illustrate it with the following type:

```typescript
type Option<T> = { tag: "some"; data: T } | { tag: "none" }

// This type is not very usable without functions to work with it, so along with this ADT
// there are a bunch of functions defined

// creates an instance with the data inside
export const some = <T>(data: T): Option<T> => ({ tag: "some", data })
// a singleton instance denoting the absence of the value
export const none: Option<never> = { tag: "none" }

// a predicate to determine if the container is empty or not
const isSome = <T>(instance: Option<T>): instance is { tag: "some"; data: T } =>
  instance.tag === "some"
```

Having such abstraction is somewhat useful, but the real goodness comes when you learn about the ways to manipulate it with special functions that are defined for this specific purpose.

**A small disclaimer: it's just an illustration of how these functions can be defined. There are much more in the _ft-ts_ package and FP in general has much more abstractions just as useful as this one, but they are more advanced and require more groundwork and theory to be able to wrap head around. Since we're trying to make our code approachable without a PhD, we're only using a tip-of-an-iceberg subset of what's available in fp-ts**

### map

`map` function is used to transform an instance of an `Option` of some type `Option<T>` to the instance of `Option` of some other type `Option<R>`, by applying a function from type `T` to `R` (`T => R`) to an instance of `Option<T>`
A formal definition:

```typescript
function map<T, R>(fn: (value: T) => R) {
  return (instance: Option<T>): Option<R> =>
    isSome(instance) ? some(fn(instance.data)) : none
}
```

As you can see the function is curried and it receives the mapping function as the first argument.

Usage with ft-ts:

```typescript
import { some, none, map, Option } from "fp-ts/lib/Option"

type Transaction = {
  date: Date
  amount: number
}

const todaysTransaction = (amount: number): Transaction => ({
  date: new Date(),
  amount,
})

const createTransaction = map(todaysTransaction) // `createTransaction` is a function which applies a function from type `number` to type `Transaction` to an `Option<number>` and returns an `Option<Transaction>`.

// construct an Option<number>
test("map", () => {
  const hundredOption = some(100)
  console.log(hundredOption) // {_tag: "Some", value: 100}

  const transaction: Option<Transaction> = createTransaction(hundredOption)
  console.log(transaction) // {_tag: "Some", value: { date: <today date>, amount: 100 } }

  const emptyAmount: Option<number> = none // since `none` is Option<never> it can be assigned to any type - Option<number>, Option<string>, etc..

  const emptyResult: Option<Transaction> = createTransaction(emptyAmount)
  console.log(emptyResult) // {_tag: "None"}
})
```

When applied to an empty instance of `Option<T>`, the mapping function `T => R` is not executed, but the returned instance is `Option<R>` meaning that the type of the internal value is changed, which let's us use in further computations (i.e `emptyResult` is still and `Option<Transcation>` though it doesn't hold such value in reality).

### alt

`alt` function can be used to generate an alternative with function that does not accept any arguments and returns `Option<T>` for the empty instance of `Option<T>`.

```typescript
function alt<T>(or: () => Option<T>) {
  return (instance: Option<T>): Option<T> =>
    isSome(instance) ? instance : or()
}
```

Usage with ft-ts:

```typescript
import { some, none, alt } from "fp-ts/lib/Option"
// imagine how we can have a function getting some URL from the config, and it returns an Option<strig>,
// since this url can be absent in the config

const getUrlFromConfig = () => some("http://config.value.com")

const fallbackToUrlFromConfig = alt(() => getUrlFromConfig())

test("alt", () => {
  console.log(fallbackToUrlFromConfig(some("http://example.com"))) // {_tag: "Some", value: "http://example.com"}

  console.log(fallbackToUrlFromConfig(none)) // {_tag: "Some", value: "http://config.value.com"}
})
```

Notice that in first case, where we apply `fallbackToUrlFromConfig` to non-empty `Option<string>` which is `some("http://example.com")`, the alternative function `getAlternative` is never executed.

### chain

`chain` function is used to transform an instance of `Option<T>` to the instance of some other type `Option<R>`, by applying a function from type `T` to `Option<R>` (`T => Option<R>`) to an instance of `Option<T>`
A formal definition and a usage example:

```typescript
function chain<T, R>(fn: (value: T) => Option<R>) {
  return (instance: Option<T>): Option<R> =>
    isSome(instance) ? fn(instance.data) : none
}
```

Usage with ft-ts:

```typescript
import { some, chain, none, Option } from "fp-ts/lib/Option"

// let's imagine that somewhere in our system we need to parse strings similar to "EUR -3231", "USD 1232.23232123"
// we would also like to be able to access currency code and amount after we parsed it
type ParsedTransaction = {
  code: string
  amount: number
}

// we could achieve it with the following parser
const parseTransaction = (
  rawTransaction: string
): Option<ParsedTransaction> => {
  const [, code, rawAmt] = /^(EUR|USD)\W(.*)$/.exec(rawTransaction) || []
  // since rawAmt can be any combination of symbols, we need to parse a float from it
  const amount = parseFloat(rawAmt)

  return Number.isNaN(amount)
    ? none
    : some({
        code,
        amount,
      })
}

// suppose down in the business logic we would like to access the fractional amount of parsedTransaction
// since not all numbers have fractional digits, we can use Option again
// this is exactly the place where using chain might help
const decimals = (parsed: ParsedTransaction): Option<number> => {
  const decimalRemainder = parseFloat(
    `${parsed.amount}`.slice(parsed.amount.toFixed().length) || "0"
  )
  return decimalRemainder === 0 ? none : some(decimalRemainder)
}

test("chain", () => {
  const transactionDecimals = chain(decimals)

  const goodTransactionOption = parseTransaction("EUR -3231")
  console.log(goodTransactionOption) // { _tag: "Some", value: { code: "EUR", amount: -3231 } }
  console.log(transactionDecimals(goodTransactionOption)) // {_tag: "None"} - because 3231 has no fractional part

  const malformedTransactionOption = parseTransaction("EUR a23a")
  console.log(malformedTransactionOption) // {_tag: "None"} - failed to parse initial payload
  console.log(transactionDecimals(malformedTransactionOption)) // {_tag: "None"} - because initially there was an empty Option<ParsedTransaction>

  const goodTransactionOptionWithFractionalAmt = parseTransaction(
    "EUR 52323.2133232"
  )
  console.log(goodTransactionOptionWithFractionalAmt) // { _tag: "Some", value: { code: "EUR", amount: 52323.2133232 } }
  console.log(transactionDecimals(goodTransactionOptionWithFractionalAmt)) // {_tag: "Some", value: 0.2133232 }
})
```

Please note that in case of `malformedTransactionOption` the `decimals` function body is never called because we had an empty `Option<ParsedTransaction>`

Having clever ways to compose possibly empty results is cool but it seems a bit problematic to manipulate such containers all the time and be able to sometimes extract a value out of them. That's where function such as `fold` and `getOrElse` come to support

### getOrElse and fold

`getOrElse` let's you to extract a value of type `T` from the instance of `Option<T>` by providing a function returning a fallback value of the same type `T`.

```typescript
function getOrElse<T>(or: () => T) {
  return (instance: Option<T>): T => (isSome(instance) ? instance.data : or())
}
```

`fold` lets you to extract a value of type `R` from instance of `Option<T>` by running a function from from `T => R` on a value inside an option, or generating an alternative value of type `R` from the callback function

```typescript
function fold<T, R>(or: () => R, withValue: (value: T) => R) {
  return (instance: Option<T>): R =>
    isSome(instance) ? withValue(instance.data) : or()
}
```

Usage with fp-ts:

```typescript
import { none, some, getOrElse, fold } from "fp-ts/lib/Option"

const safeNumber = (raw: string) => {
  const parsed = parseFloat(raw)
  return Number.isNaN(parsed) ? none : some(parsed)
}

test("getOrElse and fold", () => {
  const goodNumber = safeNumber("123.4")
  const badNumber = safeNumber("foo")

  const numberOrMeaningOf42 = getOrElse<number>(() => 42)
  console.log(numberOrMeaningOf42(goodNumber)) // 123.4
  console.log(numberOrMeaningOf42(badNumber)) // 42

  const dividedBy2Or42 = fold<number, number>(
    () => 42,
    num => num / 2
  )
  console.log(dividedBy2Or42(goodNumber)) // 61.7
  console.log(dividedBy2Or42(badNumber)) // 42
})
```

## Optics

In most of Functional Programming languages and approaches there is a notion of immutability. Meaning that instead of mutating objects, we re-create them with the updated properties (think of Redux reducers). It makes it easier to test and parallelize the code, but also comes with the cost in memory allocation and copying stuff everywhere. It also brings a lot of boilerplate and sometimes the code becomes hardly readable. Let's take a look at a super common example:

```typescript
// somewhere in your Redux reducer

type NestedState = {
  account: {
    transactions: Array<{ date: Date; amount: number }> | null
  }
  otherStuff: any
}

const state: NestedState = {
  account: {
    transactions: null,
  },
  otherStuff: {},
}

// How does updating the second transaction's ammount looks like?
const updatedState: NestedObject = {
  ...state, // need to remember about the rest of the state
  account: {
    transactions:
      transactions && // need to check if transactions are null
      transactions.map((transaction, idx) =>
        idx === 1
          ? {
              ...transaction, // need to update only one field
              amount: 555,
            }
          : transaction
      ),
  },
}
```

Not that bad, but to really understand what's going on here, you need to dig through all these updates.
Compare this to mutable approach:

```typescript
state.account.transcations?.[1]?.amount = 555
```

You're not the first to ask if it's even worth it to make your life harder for, what might seem not a good reason. It does and of course people came up with the ways to make this possible.
In particular in typescript world - there is a great library called `immer` which let's you make immutable updates using **mutable** syntax as above. Another approach is to use the concept of _Optics_

The mnemonic behind the **optics** is that they let you to `focus` on some parts of your data and manipulate it.
There are a lot of libraries in the wild, `Ramda` has it's basic optics too, but since we're relying on `fp-ts` and its ecosystem a library called `monocle-ts` is used.

`monocle-ts` provides a great variety of optics and they provide the functionality to cover any possibly complex manipulation of data, but we're only using a subset of those: `Lens`, `Prism` and `Optional`. Of course using plain old TS for immutable updates is completely fine, but when you need to read and update nested structure it is a great thing to get compiler support for it and more expressive syntax.
All of the optics provided by `ft-ts` come in form of classes and functions to conveniently create their instances. All the classes are greatly composable with each other.

### Lens

Lenses are the basic optics. They let you to conveniently get and set values of object properties, regardless of how deep they are

#### Lens generators

To accomodate various use cases `monocle-ts`'s `Lens` class has a bunch of static methods which are responsible for creation of various lens generators

```typescript
import { Lens } from "monocle-ts"

type Transaction = {
  date: Date | null
  amount: number
  participants?: {
    benificiary: string
  }
}

// first you create lens generator
const transactionProp = Lens.fromProp<Transaction>()
```

Note that `transactionProp` is a function that generates lenses. To use this generator we need to call it with one string argument: `"date"`, `"amount"` or `"participants"`. Because we provide the concrete type of `Transaction` when we create `transactionProp`,
this function will only be expecting exactly `"date"` or `"amount"` and will lead to compilation error otherwise:

```typescript
const fizzLens = transactionProp("fizz") // this will not compile
```

Let's move further and actually use our lens generator to create a couple of useful lenses:

```typescript
const transactionDateLens = transactionProp("date")
const transactionAmountLens = transactionProp("amount")
```

There are more lens generators available:

```typescript
const transactionNullableProp = Lens.fromNullableProp<Transaction>()
const transactionPath = Lens.fromPath<Transaction>()
```

`transactionNullableProp` is similar to `transactionProp` as it expects a string name of **nullable** property (a propery that can be of type `undefined` or `null`) as the first argument. But it also requires you to provide the second argument which will be used as a fall-back value:

```typescript
const endOf2018Fallback = new Date(2018, 11, 31)
const transactionDateOrEnd2018Lens = transactionNullableProp(
  "date",
  endOfYearFallback
)
// const transactionDateOrFoo = transactionNullableProp("date", "foo") // won't compile as "foo" is not of type Date
```

`transactionPath` expects an array of strings representing a **path** within Transaction structure:

```typescript
const benificiaryLens = transactionPath(["participants", "benificiary"])
// const someOtherParticipantLens = transactionPath(["participants", "somebody"]); // this won't compile,
// because there is no such path as ["participants", "somebody"] but the TS error message will be a bit obscure
```

We finally got some real objects:

- `transactionDateLens` is an instance of `Lens` class, and it's type is `Lens<Transaction, Date | null>`
- `transactionAmountLens` is also an instance of `Lens` class, and it's type is `Lens<Transaction, number>`
- `transactionDateOrEnd2018Lens` is an instance of `Lens` with type `Lens<Transaction, Date>`, note the difference from `transactionDateLens`'s type
- `benificiaryLens` is an instance of `Lens` with type `Lens<Transaction, string>`

Note that type signatures also captures information about `Transaction` and the type of the property value: `Date | null` for `"date"` property and `number` for `"amount"` property. Note in `benificiaryLens` how TS was able to understand that accessing path `["participants", "benificiary"]` returns a value of type `string`! It is intentional and will let us get more help from TS when using lense methods

#### Lens methods

Having instances of Lens class lets us start using its methods and that's actually why we all are here:

- `get` method is used to access the prop value on the instance:

```typescript
// first we need an instance of Transaction
const transactionInstance: Transaction = {
  date: new Date(2018, 0, 1),
  amount: 2000,
  participants: {
    benificiary: "John Doe",
  },
}

console.log(transactionDateLens.get(transactionInstance)) // Mon Jan 01 2018 00:00:00 GMT+0100
console.log(transactionAmountLens.get(transactionInstance)) // 2000
console.log(transactionDateOrEnd2018Lens.get(transactionInstance)) // Mon Jan 01 2018 00:00:00 GMT+0100
console.log(transactionDateOrEnd2018Lens.get({ amount: 200 })) // Mon Dec 31 2018 00:00:00 GMT+0100
console.log(benificiaryLens.get(transactionInstance)) // John Doe

// note that calling get method on the object that differs from transaction Shape causes compile error:
// transactionAmountLens.get({ foo: "bar" }) // will not compile

// IMPORTANT NOTE: path lenses are not 'safe' from undefined values
// for example the following object is correct in terms of Transaction type since "participants" property is optional
const withoutParticipants: Transaction = {
  date: null,
  amount: 2000,
}

// so passing it to 'get' (or any other method expecting Transaction) compiles just fine
// but it leads to a runtime error
console.log(benificiaryLens.get(withoutParticipants)) // Cannot read property "benificiary" of undefined

// this probably would be fixed in the next versions of monocle-ts, but for now it is not, so you need to be careful
// and of course there are other ways to safely handle such situations which will be shown later
```

- `set`

```typescript
const setAmountTo123 = transactionAmountLens.set(123)
const transactionWithUpdatedAmount: Transaction = setAmountTo123(
  transactionInstance
)
console.log(transactionWithUpdatedAmount.amount) // 123
console.log(transactionWithUpdatedAmount.date === transactionInstance.date) // true
console.log(transactionInstance.date === 2000) // true
console.log(transactionWithUpdatedAmount === transactionInstance) // false
```

Let's look a bit closer at what happens here:

- `transactionAmountLens.set` method expects a single `number` argument and will cause compilation error otherwise
- `transactionAmountLens.set(123)` returns a function, which expects a single argument of `Transaction` type and causes compilation error otherwise
- `setAmountTo123` when invoked returns a value of `Transaction` type
- `transactionWithUpdatedAmount.amount` equals to 123, but `transactionWithUpdatedAmount.date` is same as `transactionInstance.date`
- `transactionInstance.date === 2000` means that original `transactionInstance` was not changed
- `transactionWithUpdatedAmount === transactionInstance` means that `transactionWithUpdatedAmount` is a completely separate object

As we can see a lot of things happened, and effectively what just happened is similar to manually creating an object with destructuring:

```typescript
// plain old way of doing it
const transactionWithManuallyUpdatedAmount: Transaction = {
  ...transactionInstance,
  amount: 2000,
}

// compare to a more expressive way
const transactionWithUpdatedAmount1: Transaction = transactionAmountLens.set(
  123
)(transactionInstance)
```

I don't know about you, but to me the `transactionWithUpdatedAmount1` was created in a more exressive way and the code `transactionAmountLens.set(123)(transactionInstance)` almost reads as plain english.

- 'modify`method is used, well, to modify the value, and the difference from`set` is that it expects a function which receives the current value of lense and should return the value of the same type

```typescript
const modifyTransactionMonth = transactionDateLens.modify(
  (currentDate: Date | null) =>
    currentDate && new Date(currentDate.getFullYear(), 2, currentDate.getDate())
)
console.log(modifyTransactionMonth(transactionInstance).date) // Thu Mar 01 2018 00:00:00 GMT+0100
```

#### Lens composition

Having all these useful methods is very handy, but since it is an FP concept, there should be a notion of composition, right?
Sure! To illustrate this idea let's try to address an issue that we saw earlier - safe access to paths:

```typescript
// a reminder that path lens is not safe when accessing undefined values
const withoutParticipants: Transaction = {
  date: null,
  amount: 2000,
}
const transactionPath = Lens.fromPath<Transaction>()
const benificiaryLens = transactionPath(["participants", "benificiary"])
console.log(benificiaryLens.get(withoutParticipants)) // boom

// we already saw another lens generator `fromNullableProp`
// so it looks like we could handle absence of "participants" object using it
// but we still need to access "benificiary" property on it?
// here's how it might be achieved using lens composition

const participantsOrJohnDoeBenificiaryLens = Lens.fromNullableProp<
  Transaction
>()("participants", { benificiary: "John Doe" })
// now we need to access "beinificiary" property on participants
const participantsBenificiaryLens = Lens.fromProp<
  Transaction["participants"]
>()("benificiary")

// behold
const benificiaryOrJohnDoeLens = participantsOrJohnDoeBenificiaryLens.compose(
  participantsBenificiaryLens
)

// manipulating empty paths is safe now
console.log(benificiaryOrJohnDoeLens.get(withoutParticipants)) // John Doe
const updatedBenificiary = benificiaryOrJohnDoeLens.set("James Smith")(
  withoutParticipants
)
console.log(benificiaryOrJohnDoeLens.get(updatedBenificiary)) // James Smith
```

In fact you can `compose` as deep as you want and typescript will have your back all the way, because when you call `compose` on the instance of `Lens<A, B>` you'll have to provide an instance of `Lens<B, C>`, meaning that output type of the first lens needs to be aligned with the input type of the second lens, i.e `participantsOrJohnDoeBenificiaryLens` is `Lens<Transaction, { benificiary: string }>` and `participantsBenificiaryLens` is `Lens<{ benificiary: string }, string>`. Since their types align and after composing them we get `benificiaryOrJohnDoeLens` which is an instance of `Lens<Transaction, string>`.

Other optics like `Optional`, `Prism`, etc obey to similar composition patterns and different types of optics can even compose between each other, which we'll see in the final example

### Optional

Another type of optics is `Optional`. As you've might guessed from the name it seems to have something to do with the notion of optional (or absent) values (maybe even something similar to `Option` type from `fp-ts`???). In fact `monocole-ts` uses `fp-ts`'s `Option` type underneath. The ergonomics are similar to other optics from `monocle-ts` library

#### Optional generators

```typescript
// since we're functional programmers let's replace the god-awful `null` with the `proper` abstraction
type FunctionalTransaction = {
  date: O.Option<Date>
  amount: number
  participants?: {
    benificiary: string
  }
}

const transactionNullablePropOptional = Optional.fromNullableProp<
  FunctionalTransaction
>()

const participantsO = transactionNullablePropOptional("participants")
// as with lenses, in return we get an instance of class `Optional` (not `Option`!!!) fixed to the types of which it may operate:
// participantsO is an instance of Optional<FunctionalTransaction, { participants: string }>

const transactionOptionPropOptional = Optional.fromOptionProp<
  FunctionalTransaction
>()
// note that `Optional.fromOptionProp<FunctionalTransaction>()` is able to infer which properties in FunctionalTransaction have `Option` type
// so the returned function only accepts names of those properties and causes compilation error otherwise
const dateO = transactionOptionPropOptional("date") // dateO is an instance of Optional<FunctionalTransaction, Date>
// const wontCompile = transactionOptionPropOptional("amount") // compile error
```

#### Optional methods

- `getOption` instead of `Lens.get` since 'monocle-ts`uses`Option` underneath:

```typescript
const fTransactionWithValues: FunctionalTransaction = {
  date: O.some(new Date(2018, 0, 1)),
  amount: 200,
  participants: {
    benificiary: "John Smith",
  },
}

const fTransactionWithoutValues: FunctionalTransaction = {
  date: O.none,
  amount: 200,
  participants: undefined,
}

console.log(participantsO.getOption(fTransactionWithValues)) // Some({ benificiary: "John Smith" })
console.log(participantsO.getOption(fTransactionWithoutValues)) // None

console.log(dateO.getOption(fTransactionWithValues)) // Some(Mon Jan 01 2018 00:00:00 GMT+0100)
console.log(dateO.getOption(fTransactionWithoutValues)) // None
```

- `set` is also there and behaves exactly the same as for `Lens`

```typescript
import { pipe } from "fp-ts/lib/pipeable"

const updatedWithValues = pipe(
  fTransactionWithValues,
  dateO.set(new Date(2020, 0, 1)),
  participantsO.set({ benificiary: "Jane Doe" })
)

console.log(updatedWithValues.date) // Some(Wed Jan 01 2020 00:00:00 GMT+0100)
console.log(updatedWithValues.participants) // { benificiary: "Jane Doe" }
```

- `modify` is also similar to `Lens.modify`, except the passed function will only be called and the thus the update will only happen if the modified property is instance of `Some`:

```typescript
const marchOfTheSameYear = (currentDate: Date) =>
  new Date(currentDate.getFullYear(), 2, currentDate.getDate())

const modifiedWithValues = dateO.modify(marchOfTheSameYear)(
  fTransactionWithValues
)
console.log(modifiedWithValues.date) // Some(Thu Mar 01 2020 00:00:00 GMT+0100)

const modifiedWithoutValues = dateO.modify(marchOfTheSameYear)(
  fTransactionWithoutValues
)
console.log(modifiedWithoutValues.date) // None which is expected
console.log(modifiedWithoutValues === fTransactionWithoutValues) // true! modify function is never executed and the original object is returned
```

- `modifyOption` is an bit safer alternative of `modify`, since it explicitly denotes whether modification actually happened or not by returning an `Option` of `Optional` input type, i.e `Option<FunctionalTransaction>` for `Optional<FunctionalTransaction, A>`

```typescript
const indentity = <T>(arg: T): T => arg

console.log(dateO.modifyOption(indentity)(fTransactionWithoutValues)) // None - since modification didn't happen

const modifiedOptionWithValues = dateO.modifyOption(indentity)(
  fTransactionWithValues
)

console.log(modifiedOptionWithValues) // Some({ date: Some(Date(2018, 0, 1)), amount: 200, participants: { benificiary: "John Smith" } })
```

### Prism

Prims is an abstraction of conditional access. While `Optional` only handles condition of absence
of the value, prisms let you define your own conditions.

#### Prism generators

Since `Prism` is a bit more generic, we need to provide a bit more than just an input type and a path or property, so let's look a bit closer how we `Prism` instance is constructed directy and what functionality it provides first:

```typescript
import { Prism } from "monocle-ts"
import * as O from "fp-ts/lib/Option"

const transactionWithoutDate: Transaction = {
  date: null,
  amount: 2000,
}

const transactionDatePrism = new Prism(
  (t: Transaction): O.Option<Date> => O.fromNullable(t.date),
  (date: Date): Transaction => ({ date, amount: 0 })
)

console.log(transactionDatePrism.getOption(transactionWithoutDate)) // None
console.log(transactionDatePrism.reverseGet(new Date(2018, 0, 1))) // { date: Date(2018, 0, 1), amount: 0 }
```

`Prism` constructor is called with two arguments:

- first is a function `A => Option<B>` (in example it's `Transaction => Option<Date>`), which denotes a condition (`Some` if the condition is true, `None` otherwise);
- second is a reverse function `B => A`, a so-called `reverseGet`, (`Date => Transaction` in the example, which let's us create a transaction object with 0 amount and provided date)

Again `A` and `B` types must align for both functions and TS is able to infer them and will be expecting them when calling `getOption` and `reverseGet` respectively on the instance of `Lens<A, B>` (`Lens<Transaction, Date>` in the example)

Creating prisms this was looks a bit wordy and there is one overloaded smart constructor to simplify prism creation `fromPredicate`:

```typescript
const transactionWithDatePrismFromPredicate = Lens.fromPredicate(
  (t: Transaction) => !!t.date
)
// note that since predicate should return `true` or `false`, library is unable to infer a particular field
// meaning that `transactionWithDatePrismFromPredicate` is instance of Prism<Transaction, Transaction>

// a special TS predicate implementation is refinement
// note that A type can be refined to B, but B should be a subtype of A
interface Refinement<A, B extends A> {
  (a: A): a is B
}

// here's an example demonstrating what refinement of type is
type Loaded = {
  data: Array<Transaction>
}

type Error = {
  message: string
}

type State = Loaded | Error

const isLoaded: Refinement<State, Loaded> = (s: State): s is Loaded =>
  "data" in s

const isLoadedPrism = Prism.fromPredicate(isLoaded)

console.log(loadedStateP.getOption({ message: "Some error" })) // None
console.log(loadedStateP.getOption({ data: [] })) // Some({ data: [] })
```

Since `Loaded` is a **subtype** of `State`, you can create `Refinement<Loaded, State>` but you can not create a `Refinement<State, Transaction>` and thus you can not simply create `Prism<State, Transaction>`. Though you could create it manually using `new Prism(...)` syntax, you rarely need it, since optics shine in composition and you better rely on it, which we will see in the final example.

### Final (Boss) Example

Prepare to behold the power of optics and composition on the following example:

```typescript
type Transaction = {
  date: Date | null
  amount: number
  participants?: {
    benificiary: string
  }
}

type Loaded = {
  data: Array<Transaction>
}

type Error = {
  message: string
}

type State = Loaded | Error
```

The task is to write a function that **immutably** updates a value of type `State` (meaning it simply generates a new one and updates all the nested objects) changing the `date` and `benificiary` **if there are** `participants` of the **n-th transaction in the array** **if there was no error**

Here's the imperative implementation of such function (we made it curried, so we will be able to easier log it's results):

```typescript
function updateTransactionDateAndBenificiary(
  nth: number,
  newDate: Date,
  newBenificiary: string
): (state: State) => State {
  return state => {
    // check if State is Loaded, meaning there was no error
    if ("data" in state) {
      const secondTransaction = state.data[nth]
      // check optional value
      if (secondTransaction) {
        const { participants } = secondTransaction

        return {
          data: [
            ...state.data.slice(0, nth), // manage immutable update
            {
              ...secondTransaction, // manage immutable update
              date: newDate,
              ...(participants // check condition
                ? {
                    participants: {
                      ...participants,
                      benificiary: newBenificiary,
                    },
                  }
                : {}), // manage immutable update
            },
            ...state.data.slice(nth + 1), // manage immutable update
          ],
        }
      }
    }

    return state
  }
}
```

Let's see if the function does what we wanted and doesn't break for some edge cases:

```typescript
const startOf2018 = new Date(2018, 0, 1)
const updateSecondTransaction = updateTransactionDateAndBenificiary(
  1 /* array's indexes are 0 based */,
  startOf2018,
  "Michael Jackson"
)

console.log(
  updateSecondTransaction({
    message: "This is error",
  })
) // returns { message: "This is error" } unchanged
console.log(
  updateSecondTransaction({
    data: [],
  })
) // returns { data: [] } unchanged

const transactionsWithoutParticipantsState: Loaded = {
  data: [
    {
      amount: 200,
      date: null,
    },
    {
      amount: 500,
      date: null,
    },
  ],
}

console.log(updateSecondTransaction(transactionsWithoutParticipantsState)) // { data: [{ amount: 200, date: null }, { amount: 500, date: Mon Jan 01 2018 00:00:00 GMT+0100 }] }

const transactionsWithParticipantsState: Loaded = {
  data: [
    {
      amount: 200,
      date: null,
    },
    {
      amount: 300,
      date: null,
      participants: {
        benificiary: "John Smith",
      },
    },
  ],
}

const updated = updateSecondTransaction(transactionsWithParticipantsState)

console.log(updated) // { data: [{...}, { amount: 500, date: Mon Jan 01 2018 00:00:00 GMT+0100, participants: { benificiary: "Michael Jackson" } }] }
console.log(updated === transactionsWithParticipantsState) // false meaning new object was created
console.log(updated.data === transactionsWithParticipantsState.data) // false meaning new array of Transaction's was created
console.log(updated.data[1] === transactionsWithParticipantsState.data[1]) // false meaning new transaction object was created
console.log(
  updated.data[1].participants ===
    transactionsWithParticipantsState.data[1].participants
) // false meaning new participants object was created
```

Phew, looks like we properly implemented the logic, handled immutable updates and edge cases, but at what cost? There's a lot of logic to properly handle immutability, especially for arrays and it's not that easy to reason about it, because domain logic is mixed up with implemetation details.

Luckily Optics to the rescue, as you'll see, abstract immutable data access and can be assembled together like lego. We can define instances of optics that cover our use cases and then compose them together:

```typescript
import * as O from "fp-ts/lib/Option"
import { pipe } from "fp-ts/lib/pipeable"
import { Lens, Optional, Prism } from "monocle-ts"
// Let's first address conditions in our code

// 1) [if there was no error] means that State is Loaded, we already know that's where Prisms shine
const ifLoaded = Prism.fromPredicate((s: State): s is Loaded => "data" in s)

// 2) [n-th transaction in the array] since arrays allow access at any index and return undefined if there is no element, this seems like Optional

// Note: though numeric-indexes of the array are in fact object properties, we could use Optional.fromNullableProp<Loaded["data"]>()
// but monocle-ts is not be able to preserve array structure, so we have to manually create an Optional generator for our use case:
const ifHasTransaction = (
  nth: number
): Optional<Array<Transaction>, Transaction> =>
  new Optional(
    transactions => O.fromNullable(transactions[nth]), // here's where we handle possible absence of the nth-transaction
    t => transactions => {
      // here we provide a function to handle immutable update of the nth-transaction
      const copy = transactions.slice() // we copy the original array
      copy[nth] = t // update the nth-transaction
      return copy // and return newly created array
    }
  )

// 3) [if there are participants] again looks like Optional, so here it goes

const ifHasParticipants = Optional.fromNullableProp<Transaction>()(
  "participants"
)

// update the ["date"] and ["benificiary"] doesn't seem to deal with conditions and optionality, so we could simply use Lens'es
const date = Lens.fromProp<Transaction>()("date")
const benificiary = Lens.fromProp<Transaction["participants"]>()("benificiary")

// and we also would like to access data on Loaded state, so another Lens
const data = Lens.fromProp<Loaded>()("data")

// looks like we created our lego bricks of the logic, let's try to seemble them
function updateTransactionDateAndBenificiaryWithOptics(
  nth: number,
  newDate: Date,
  newBenificiary: string
): (state: State) => State {
  // let's start defining our logic's building blocks

  // 1) we need a function that updates a single transaction's date
  const updateTransactionDate: (t: Transaction) => Transaction = date.set(
    newDate
  )

  // 2) we need function that updates benificiary of single transaction if it has participants
  const updateBenificiaryIfParticipantsExist: (
    t: Transaction
  ) => Transaction = ifHasParticipants
    .composeLens(benificiary)
    .set(newBenificiary)

  // 3) Optional to access nth-transaction if the State is loaded
  const nthTransaction: Optional<State, Transaction> = ifLoaded
    .composeLens(data)
    .composeOptional(ifHasTransaction(nth))

  // 4) construct final function
  const updateState: (state: State) => State = nthTransaction.modify(
    transaction => {
      return pipe(
        transaction,
        updateTransactionDate,
        updateBenificiaryIfParticipantsExist
      )
    }
  )

  return updateState
}
```

I inetionally created all the intermediary functions and variables to show that we can combine all the blocks as we want and what is more crucial - TS will have our back and check if we use proper methods to compose corresponding optics and our functions return proper results. In fact, when you are more used to the syntax you can write it in much more consice way and TS will still be able to check if you are doing the right thing:

```typescript
// looks like we created our lego bricks of the logic, let's try to seemble them
function updateTransactionDateAndBenificiaryWithOptics(
  nth: number,
  newDate: Date,
  newBenificiary: string
): (state: State) => State {
  return ifLoaded
    .composeLens(data)
    .composeOptional(ifHasTransaction(nth))
    .modify(transaction =>
      pipe(
        transaction,
        date.set(newDate),
        ifHasParticipants.composeLens(benificiary).set(newBenificiary)
      )
    )
}
```

Regardless of the preferred implementation of `updateTransactionDateAndBenificiaryWithOptics` let's actually make sure it achieves the same thing as out imperatively written function:

```typescript
const beholdThePowerOfOptics = updateTransactionDateAndBenificiaryWithOptics(
  1,
  startOf2018,
  "Michael Jackson"
)

console.log(
  beholdThePowerOfOptics({
    message: "This is error",
  })
) // returns { message: "This is error" } unchanged
console.log(
  beholdThePowerOfOptics({
    data: [],
  })
) // returns { data: [] } unchanged

console.log(
  beholdThePowerOfOptics({
    data: [
      {
        amount: 200,
        date: null,
      },
      {
        amount: 500,
        date: null,
      },
    ],
  })
) // { data: [{ amount: 200, date: null }, { amount: 500, date: Mon Jan 01 2018 00:00:00 GMT+0100 }] }

const transactionsWithParticipantsState1: Loaded = {
  data: [
    {
      amount: 200,
      date: null,
    },
    {
      amount: 300,
      date: null,
      participants: {
        benificiary: "John Smith",
      },
    },
  ],
}

const updated1 = updateSecondTransaction(transactionsWithParticipantsState)

console.log(updated) // { data: [{...}, { amount: 500, date: Mon Jan 01 2018 00:00:00 GMT+0100, participants: { benificiary: "Michael Jackson" } }] }
console.log(updated === transactionsWithParticipantsState) // false meaning new object was created
console.log(updated.data === transactionsWithParticipantsState.data) // false meaning new array of Transaction's was created
console.log(updated.data[1] === transactionsWithParticipantsState.data[1]) // false meaning new transaction object was created
console.log(
  updated.data[1].participants ===
    transactionsWithParticipantsState.data[1].participants
) // false meaning new participants object was created
```

As we see - it behaves the same as imperative code and offloads immutability handling (okay, apart from immutable array updates :)) from you. Plus it seems a bit more expressive and brings you joy (hopefully)
