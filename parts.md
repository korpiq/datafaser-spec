# Parts of datafaser language and implementations

## Input and output sources

### File

### Network

### Database

### Interactive

### Other source options

## Input and output formats

### JSON

### YAML

### XML

### CSV

### Text

### Other format options

## Validation

### JSON Schema

### Scalar types based on schema

#### Numeric type selection based on range and precision

#### Choosing string or byte array type

### Other validation options

## Internal datatypes

### Nodes

#### Numbers

##### BigNum

##### Limited size numbers

#### Strings

##### Byte arrays

##### Unicode strings

#### Lists

#### Name-value records

#### Null

#### Error

### Selection results

    (path, value)

## Data selection

### Path

A path is a list of keys each referring to the name of a record field or index of list item in the associated collection:

    [ key, ... ]

For instance, the path to 12 below would be `[0, "age"]`:

    [ { age: 12 } ]

#### Empty path refers to whole

    []

### Selectors

Selector paths are paths where some entries may be selector records instead of static key values.

Individual selectors are records that specify which parts of associated collection will be considered for selection.

Output of each selector and selector path is a list of matching `(path, key value)` entries.

#### Selector target types

1. Value selectors inspect the value part of an entry.

    value: [Selection Criteria]

2. Key selectors inspect the current key, or latest path element, of an entry.

    key: [Selection Criteria]

3. Path selectors inspect the full path leading up and including the current key in an entry.

    path: [Selection Criteria]

#### Comparing to selection candidate

Selectors can have a `with` clause. It can specify names to use for the `path`, `key`, and `value` field of each entry, so they can be used for comparison in their selection criteria.

Example: select only entries whose keys are listed elsewhere:

    path: [ "list of keys to select", { value: consider_key } ]
    with:
        consider_key: key

Example: select only entries whose value is a data structure that contains the path of current entry.

    value: { path: [ current_path ] }
    with:
        current_path: path

### Selector criteria

#### Equals

Match any single value or data structure exactly.

    equals: [Value to match exactly]

#### Range

Number above or at least a given minimum and/or below or at most a given maximum value. A range without any limits accepts any Number.

    range:
        [above: Number | min: Number |]
        [below: Number | max: Number |]

Example: select numbers above zero:

    range: { above: 0 }

#### Modulo

Matches values having specified `remainder` when divided by given `divider`. Default `remainder` is zero, meaning that the value is a multiple of given `divider`.

    modulo:
        divider: Number
        [remainder: Number]

Example: pick only even keys of a list:

    path: [ { modulo: { divider: 2 } } ]

#### Contains

Matches any text or byte array with exactly given value appearing anywhere within it.

    contains: "string"

#### More selector criteria

There are plenty more options to consider as potential selection criteria.

#### Collections of selectors

Multiple selectors may be collected together.

##### All

##### Some

##### None

## Composing results

### Composing field values

### Composing data structures

### Collecting results into records or single values instead of lists

### Handling errors
