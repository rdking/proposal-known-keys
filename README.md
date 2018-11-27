# proposal-known-keys
An ES proposal to add a new set of helper methods to Object for dealing with objects having prototypes.

## Motivation
`Object` currently has several functions designed to manipulate properties in batch. These have proven very useful over the years, but with the introduction of `class` and the increasing use of inheritance, these functions are beginning to feel dated. So much so that developers have taken to making all data properties "own properties" of the instance just to make them visible to these functions. This is further goaded by the known foot-gun of having an object data property on a prototype. However, the lengths that developers are willing to go to in order to accomodate these outdated functions are beginning show signs of trading away valuable functionality.

## Solution
To avoid this, new functions need to be crafted that perform precisely the same task as the tried and true existing functions, but this time accounting for the prototype chain. Below is a list of the existing functions and their proposed updated version. The existing functions will not be replaced.

```js
Object.assign(target, ...sources)          -> Object.assignKnown(target, ...sources)
Object.entries(obj)                        -> Object.knownEntries(obj)
Object.getOwnPropertyDescriptor(obj, prop) -> Object.getKnownPropertyDescriptor(obj, prop)
Object.getOwnPropertyDescriptors(obj)      -> Object.getKnownPropertyDescriptors(obj)
Object.getOwnPropertyNames(obj)            -> Object.getKnownPropertyNames(obj)
Object.getOwnPropertySymbols(obj)          -> Object.getKnownPropertySymbols(obj)
Object.keys(obj)                           -> Object.knownKeys(obj)
```

## Issues
1 big issue with this proposal is that accessor properties added via the `class` keyword are always defined as non-enumerable. This means that accessor properties will still not be seen by the new functions that only access enumerable properties. This proposal has no intention of remedying this behavior. Decorators may provide a solution for this.

## Behavior
The behavior of each of the 7 new functions should parallel the existing functions while diving through the prototype chain. In general, each of the new functions can be implemented by repeated application of their predecessors. A potential reference implementation of each function is provided below.


#### Object.assignKnown(target, ...sources)

```js
Object.assignKnown(target, ...sources) {
    let allSources = [];

    for (let source of sources) {
        let list = [];
        while (source) {
            list.unshift(source);
            source = Object.getPrototypeOf(source);
        }
        allSources = allSources.concat(list);
    }

    return Object.assign(target, ...allSources);
}
```

#### Object.knownEntries(obj)

```js
Object.knownEntries(obj) {
    let keys = new Set();
    let retval = [];

    while (obj) {
        let entries = Object.entries(obj);

        for (let entry of entries) {
            if (!keys.has(entry[0])) {
                keys.add(entry[0]);
                retval.push(entry);
            }
        }
        obj = Object.getPrototypeOf(obj);
    }

    return retval;
}
```

#### Object.getKnownPropertyDescriptor(obj, prop)

```js
Object.getKnownPropertyDescriptor(obj, prop) {
    let retval;

    if (prop in obj) {
        while (obj && !retval) {
            retval = Object.getOwnPropertyDescriptor(obj, prop);
            if (!retval) {
                obj = Object.getPrototypeOf(obj);
            }
        }
    }

    return retval;
}
```

#### Object.getKnownPropertyDescriptors(obj)

```js
Object.getKnownPropertyDescriptors(obj) {
    let retval = {};

    while (obj) {
        let descs = Object.getOwnPropertyDescriptors(obj);
        for (let key in descs) {
            let desc = descs[key];
            if (!(key in retval)) {
                retval[key] = desc;
            }
        }
        obj = Object.getPrototypeOf(obj);
    }

    return retval;
}
```

#### Object.getKnownPropertyNames(obj)

```js
Object.getKnownPropertyNames(obj) {
    let allNames = new Set();
    let retval = [];

    while (obj) {
        let names = Object.getOwnPropertyNames(obj);
        for (let name of names) {
            if (!allNames.has(name)) {
                allNames.add(name);
                retval.push(name)
            }
        }
        obj = Object.getPrototypeOf(obj);
    }

    return retval;
}
```

#### Object.getKnownPropertySymbols(obj)

```js
Object.getKnownPropertySymbols(obj) {
    let allSymbols = new Set();
    let retval = [];

    while (obj) {
        let symbols = Object.getOwnPropertySymbols(obj);
        for (let symbol of symbols) {
            if (!allSymbols.has(symbol)) {
                allSymbols.add(symbol);
                retval.push(symbol)
            }
        }
        obj = Object.getPrototypeOf(obj);
    }

    return retval;
}
```

#### Object.knownKeys(obj)

```js
Object.knownKeys(obj) {
    let allKeys = new Set();
    let retval = [];

    while (obj) {
        let keys = Object.keys(obj);
        for (let key of keys) {
            if (!allKeys.has(key)) {
                allKeys.add(key);
                retval.push(key)
            }
        }
        obj = Object.getPrototypeOf(obj);
    }

    return retval;
}
```

