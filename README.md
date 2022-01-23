# Akade.IndexedSet

[![CI Build](https://github.com/akade/Akade.IndexedSet/actions/workflows/ci-build.yml/badge.svg?branch=master)](https://github.com/akade/Akade.IndexedSet/actions/workflows/ci-build.yml)
![NuGet version (Akade.IndexedSet)](https://img.shields.io/nuget/v/Akade.IndexedSet.svg)

Provides an In-Memory data structure, the IndexedSet, that allows to easily add indices to allow efficient querying. Based on seeing inifficent usage of 
`.FirstOrDefault?`, `.Where`, `.Single` etc... and implementing such a data-structure for every project I'm on.

## Features
This project aims to provide a data structure (*it's not a DB!*) that allows to easily setup fast access on different properties:
### Unqiue index
Dictionary-based, O(1), access on primary and secondary keys:

```csharp
var set = IndexedSetBuilder<Data>.Create(a => a.PrimaryKey)
        .WithUnqueIndex(x => x.SecondaryKey)
        .Build();

set.Add(new(primaryKey: 1, secondaryKey: 5));

// fast access via primary key
var data = set[1];

// fast access via secondary key
var data = set.Single(x => x.SecondaryKey, 5);
```

### Non-unqiue index
Dictionary-based, O(1), access on keys with multiple values:

```csharp
var set = IndexedSetBuilder<Data>.Create(a => a.PrimaryKey)
        .WithIndex(x => x.SecondaryKey)
        .Build();

set.Add(new(primaryKey: 1, secondaryKey: 5));
set.Add(new(primaryKey: 2, secondaryKey: 5));

// fast access via secondary key
IEnumerable<Data> data = set.Where(x => x.SecondaryKey, 42);
```

### Range index
Binary-heap based O(log(n)) access for range based queries.

```csharp
var set = IndexedSetBuilder<Data>.Create(a => a.PrimaryKey)
        .WithRangeIndex(x => x.SecondaryKey)
        .Build();

set.Add(new(primaryKey: 1, secondaryKey: 3));
set.Add(new(primaryKey: 2, secondaryKey: 4));

// fast access via secondary key
IEnumerable<Data> data = set.Range(x => x.SecondaryKey, 1, 5);
```
For more samples, checkout the unit tests.


### Computed or compound key

The data structure also allows to use computed or compund keys:

```csharp
var set = IndexedSetBuilder<Data>.Create(a => a.PrimaryKey)
        .WithIndex(x => (x.start, x.end))
        .WithIndex(x => x.end - x.start)
        .WithIndex(ComputedKey.SomeStaticMethod)
        .Build();

set.Add(new(primaryKey: 1, start: 2, end: 10));

// fast access via indices
IEnumerable<Data> data = set.Where(x => (x.start, x.end), (2, 10));
IEnumerable<Data> data = set.Where(x => x.end - x.start, 8);
IEnumerable<Data> data = set.Where(x => ComputedKey.SomeStaticMethod, 42);
```

### Reflection- & expression-free - convention-based index naming

We are using the [CallerArgumentExpression](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.compilerservices.callerargumentexpressionattribute)-Feature 
of .Net 6 to provide a convention based naming of the interfaces:
- `set.Where(x => (x.Prop1, x.Prop2), (1, 2))` results in an index named `"x => (x.Prop1, x.Prop2), (1, 2))"`
- `set.Where(ComputedKeys.NumberOfDays, 5)` results in an index named `"ComputedKeys.NumberOfDays"`
- **Hence, be careful what you pass in. The convention is to always use a lambda with x as variable name or use static methods.**

### Updating key-values
**The current implementation requires any keys of any type to never change the value while the instance is within the set**. So to update you will need to remove
the instance, update the values and add the instance again.

### Roadmap
Potential features:
- Thread-safe version
- Improved updating of values
- Events for changed values
- More index types

If you have any suggestion or found a bug / unexpected behavior, open an issue!
