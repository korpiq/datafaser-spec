# Fase

Datafaser is a language for filtering and reconstructing data.
Each datafaser step, called `fase`, consists of data selection criteria and operations on selected data or other outcomes of the selection.
All parts of a `fase` are optional. An empty `fase` would pass its input as its output unmodified.

    "with":
        name: {selector},
        ...
    "each":
        name: {selector},
    "without":
      - {selector on the selected data}

    "do":
        {operation on selected data}
    "else":
        {operation to run for each part of input data that "each" selection excludes}
    "none":
        {operation to run when "each" selects no items}
    "failing":
        {operation taken for errors that happen during this fase}

## Selections in a fase

Keyword `with` gives access to selected parts of input data under specified names.
Keyword `each` causes the `do` operation to run once for each item in a selected part of input data.

If neither `with` nor `each` is defined, the `fase` will act on its whole input data as is.
If either is specified, only the parts specified in them will be visible in the `fase`.

### With picks parts of input for use in the fase

Whole input renamed as "all":

    "with":
        "all": []

### Each repeats the fase for a list of selected items

Output of an `each fase` is a list of the results of operations therein.
It may refer to names defined by `with`.

Run `fase` once for the whole input (if any):

    "each":
        "all": []

Run `fase` for each top level item separately:

    "each":
        "it": [{}]

### Without excludes parts of selected data

After `with` and `each` have selected some data, `without` can be used to exclude parts of that selection.

Exclude all fields on second level of selection named as "private":

    "without":
      - [{}, "private"]

Excluding all data blocks the whole run:

    "without":
      - []

## Actions in a fase

Keyword `else` specifies an operation to run for each part of input data that the selection in `each` decides to skip.
Keyword `none` specifies an operation to run once when `each` selects no data.
Keyword `failing` specifies an operation to run once for each error that occurs during the selections or actions in the `fase`.

### Do runs an operation

Keyword `do` specifies how to produce output from the input data. That operation can combine many smaller operations into one.

    "with":
        "all": []
    "do":
        "quote":
            "value": ["all"]

### Else runs an operation for data not selected by each

### None runs an operation when each selects no data

### Failing runs on operation for each failure happening during the fase

    "failing":
      - "in": ["with","each","without","do","else","none","failing"]
        "at": [selector]
        "type": errortype
        "do": action
        "then":
            "fail" to exit current list of fases,
            "last" to end this fase, or
            "next" to continue with next iteration of this fase

## Referencing data

### Value

Operation `value` returns the value of a name set by `each` or `with` in a fase.

    "with": "all": []
    "do": "value": ["all"]

### Path

Operation `path` returns the path leading to a value selected for a name by `each` or `with`.

This would return a list of lists with some key from the top level structure, another second level key, and "address":

    "each": "address of a person in an organisation": [{}, {}, "address"]
    "do": "value": ["address of a person in an organisation"]

### Key

Operation `key` returns the last element of a path of a value selected for a name by `each` or `with`.

- for records it is the field name.
- for lists it is the index.

### Name

Operation `name` returns the last record field name of a path of a value selected for a name by `each` or `with`.

### Index

Operation `index` returns the last list entry number of a path of a value selected for a name by `each` or `with`.

### Iteration

Operation `iteration` returns the count of iterations of this fase run so far for its `each`.

## Operations

### Fail produces an error

    "fail": { "user error": "something went awry" }

### Quote produces value as given

Use `quote` to produce a value of any type as is:

    "quote": "this text will be produced as seen here."

    "quote":
        "record field name": "record field value"

#### Unquote allows operations inside a quote

Use `unquote` to run operations inside quotes:

    "quote":
      - "Value of index is now: "
      - { "unquote": { "value": "index" } }

#### Unquote literal can be produced as output key with record operation

To include a record with "unquote" key, use "record" to build it:

    "quote":
      - "Next comes a record { unquote: true }: "
      - { "unquote": { "record": [{ "key": "unquote", "value": true }] } }

### Join combines items of a list into a text

    "join":
      - "Value of index is now: "
      - { "value": "index" }
      - { "value": "all" }

- Sublists are combined recursively just like top level.
- Records produce an error.

Alternate form that lets you specify a nonempty separator:

    "join":
        "parts": [...]
        "by": separator
        "range": range

### Split produces a list from text

    "split":
        "text": [...]
        "by": separator
        "range": range

### Keys produces a list of keys from a record

    "keys": [path]

### Values produces a list of values from a record or list

    "values": [path]

### Fields produces a list of key-value pairs from a collection

    "fields": [path]

For each field in a record or list `fields` produces a record:

    "key": key
    "value": value

### Record produces a record from a list of key-value pairs

    "record":
      - { "key": key, "value": value }
      ...

#### Insert adds new fields inside a record

Inside a `record` operation, `insert` operation inserts fields of given record into it.

    "record":
      - { "insert": { "quote": { "key1": "value1", "key2": "value2" } } }
      - { "key": key3, "value": value3 }
      - { "insert": { "record: [{ "key": key4, "value": value4 }, { "key": key5, "value": value5 }] } }

#### Replace replaces existing fields inside a record

#### Merge adds or replaces fields inside a record

### Range ensures limits on item producers

Specify a range to limit how many items an operation should generate.
An upper bound (specified by `max` or `below`) stops generation after that many items have been produced.
A lower bound (specified by `min` or `above`) causes an error if the operation produced too few items.

    "range":
        "min": lower_bound_included | "above": lower_bound_excluded,
        "max": upper_bound_included | "below": upper_bound_excluded
        "do": {operation}

## Example fases

### Collect output records of previous fase into one record

    "do":
        "record":
          - "insert": { "value": [] }

### Collect output records of previous fase into one text

    "do":
        "join":
            "value": []

### Calculate sum of numbers produced by previous fase

    "do":
        "number":
            "sum": []
