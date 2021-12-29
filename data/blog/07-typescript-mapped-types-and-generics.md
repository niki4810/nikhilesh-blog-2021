---
title: 'My learning with Typescript Mapped Types and generics'
summary: "I've been working with Typescript for past couple of years. Through this blog post I'd like to share some of my learnings, specifically with Mapped Types and Generics."
date: 2021-12-28 14:14
tags: ['react', 'TypeScript', 'hooks', 'carousel', 'css']
---

## Summary

I've been working with Typescript for past couple of years. Through this blog post I'd like to share some of my learnings, specifically with Mapped Types and Generics.


## My Usecase

I wanted to build a function that would return an object with a few requirements, The returned object:
- the type for each value for a given key should be customizable by the caller of the function
- should always contain a key named `defaultMock`
- can contain more than one key, apart from the above `defaultMock` key
- And finally when I try to lookup the object keys, typescript should hint/suggest the available keys for this object.

Let's solve each requirement step by step.

As a starting point we will have the following code. You can use [Typescript playground](https://typescript-play.js.org/) to follow along.

```
function getUserApiVariations(): Record<string, unknown> {
    return {
        "defaultMock": {"name": "Foo Baz", "age": 50},
        "withMiddleName": {"name": "Foo Bar Baz", "age": 30},
        "withNoAge": {"name": "Foo"},
        "errorMock": {}
    }
};

const userMockVariations = getUserApiVariations(); 
```

## Step 1: Making the object value customizable

The first issue we have with the above code is that it returns an object of type `Record<string, unknown>`.

<img src="/static/images/ts-mapped-types-and-generics/01.png" alt="unknown return type" />

We want the `value` of each key in the returned object to be typed. To achieve this we will use Typescript Generics. The below code shows how to achieve this:

```
function getUserApiVariations<MockType>(): Record<string, MockType> {
    return {
        "defaultMock": {"name": "Foo Baz", "age": 50},
        "withMiddleName": {"name": "Foo Bar Baz", "age": 30},
        "withNoAge": {"name": "Foo"},
        "errorMock": {}
    }
};

type User = {name?: string, age?:string};
const userMockVariations = getUserApiVariations<User>(); 
```

We define a new `User` type and use generics to pass that type to our `getUserApiVariations` function as a type variable called `MockType`. Our `getUserApiVariations` function will use this type variable and pass it along to the `Record` type return value. By doing this our function now shows the correct type for the return value instead of saying `unknown`.

<img src="/static/images/ts-mapped-types-and-generics/02.png" alt="correct return type for value" />


## Step 2: Making the keys customizable

In the above step we were able to fix the type of the value, but the key's with in each object is still too generic, and displays as `string` (see below).

<img src="/static/images/ts-mapped-types-and-generics/03.png" alt="any string key" />

There are two issues here:
1. The return type of `Record<string, User>` means the key can be `any` string
2. We do not get any typescript hint (see below) when we try to inspect the object.

<img src="/static/images/ts-mapped-types-and-generics/04.gif" alt="no ts hinting" />

To fix this we will use a combination of typescript generics and Mapped types. The below code shows how to achieve this:

```
type GenericMockRecord<MockVariations, MockType> = {
    [Property in MockVariations]: MockType
}
function getUserApiVariations<MockVariations, MockType>()
    : GenericMockRecord<MockVariations, MockType> {
    return {
        "defaultMock": { "name": "Foo Baz", "age": 50 },
        "withMiddleName": { "name": "Foo Bar Baz", "age": 30 },
        "withNoAge": { "name": "Foo" },
        "errorMock": {}
    }
};


type User = { name?: string, age?: string };
type UserMocksTypes = "defaultMock" | "withMiddleName" | "withNoAge" | "errorMock";
const userMockVariations = getUserApiVariations<UserMocksTypes, User>();
```

Similar to above step, we create a String Literal Types `UserMocksType` and pass it to `getUserApiVariations` function as a generic variable. In our `getUserApiVariations` function signature we define an additional generic type variable `MockVariations` which is passed along to a new mapped type `GenericMockRecord`.  the `GenericMockRecord` uses the Typescript type mapping, i.e. `[Property in MockVariations]` to derive the keys for the return object. The end result of this is that when we try to lookup the object, typescript provides hinting on all available keys for the object, and we can be certian that the returned object contains only keys that we intend to have.

<img src="/static/images/ts-mapped-types-and-generics/05.gif" alt="with ts hinting" />

## Step 3: Making the `defaultMock` key required.

Our above code almost perfectly matches our requirements with one exception. The key's that our function will suggest will always be equal to the `UserMocksTypes` we pass. If we always want the function to always have a `defaultMock` as a key, we need to adjust our code a bit:

```

type GenericMockRecord<MockVariations, MockType> = {
    [Property in MockVariations]: MockType
} & {"defaultMock": MockType};

function getUserApiVariations<MockVariations, MockType>()
    : GenericMockRecord<MockVariations, MockType> {
    return {
        "defaultMock": { "name": "Foo Baz", "age": 50 },
        "withMiddleName": { "name": "Foo Bar Baz", "age": 30 },
        "withNoAge": { "name": "Foo" },
        "errorMock": {}
    }
};


type User = { name?: string, age?: string };
type UserMocksTypes = "withMiddleName" | "withNoAge" | "errorMock";
const userMockVariations = getUserApiVariations<UserMocksTypes, User>();
```

Using the `&` Typescript intersection we are telling that the `GenericMockRecord` type will should always contain a key named `defaultMock` which is of the type `MockType` which is set as a generic variable at the time of calling the `getUserApiVariations`. Using this we still achieve the same functionality as above step, with an added benefit that our returned object should now always contain a `defaultMock` property. This satisfies our requirements correctly.

<img src="/static/images/ts-mapped-types-and-generics/06.png" alt="with ts hinting" />


## Using this pattern for other function


Our type `GenericMockRecord` can now be used for other functions. For e.g. if we want to create another function `getPetsApiVariations`, we could do this:

```
type GenericMockRecord<MockVariations, MockType> = {
    [Property in MockVariations]: MockType
} & {"defaultMock": MockType};


function getUserApiVariations<MockVariations, MockType>()
    : GenericMockRecord<MockVariations, MockType> {
    return {
        "defaultMock": { "name": "Foo Baz", "age": 50 },
        "withMiddleName": { "name": "Foo Bar Baz", "age": 30 },
        "withNoAge": { "name": "Foo" },
        "errorMock": {}
    }
};


type User = { name?: string, age?: string };
type UserMocksTypes = "withMiddleName" | "withNoAge" | "errorMock";
const userMockVariations = getUserApiVariations<UserMocksTypes, User>();


type Pet = {name?: sting, breed?: string};
type PetMocksTypes = "withLongName" | "withNoBreedInformation";

function getPetApiVariations<MockVariations, MockType>()
    : GenericMockRecord<MockVariations, MockType> {
    return {
        "defaultMock": { "name": "Foo", "breed": "pug" },
        "withLongName": { "name": "Foooooooooooooooo", "breed": "pug" },
        "withNoBreedInformation": { "name": "Foo" }
    }
};


const petMockVariations = getPetApiVariations<PetMocksTypes, Pet>();

```

Yielding in the same result:

<img src="/static/images/ts-mapped-types-and-generics/07.png" alt="with ts hinting for different function" />

## Conclusion

In conclusion, using typescript generics and mapped types we were able to create a reusable generic type that can be used by any function that needs to return an object where one key is mandatory while other keys can be specified by the calling function along with the return type for each value.

There maybe other ways to achieve this in typescript, but this is one way I figured out and wanted to share. Hope this helps 🙌