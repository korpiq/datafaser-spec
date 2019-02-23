# Parts of datafaser language and setup

This document turned out to go into too many details at once. I'll try another approach in another file.

1. `Datafaser language` only describes how to transform data.
2. `Datafaser setup` connects datafaser run to its environment.
3. Both parts are required to be implemented by a datafaser runner.

## Setup

Setup of a datafaser program can be specified as a data structure. That data structure can be provided by any combination of

- command-line parameters for a datafaser runner

        datafaser --input-file=source.yaml

- environment variables for the same

        DATAFASER_INPUT_FILE=source.yaml

- files or other input sources indicated by any of above

        DATAFASER_SETUP_FILE=setup.yaml

- default configuration file `datafaser.*` (if no other given)

        datafaser.yaml

        input:
            file: "source.yaml"
        plan:
            file: "datafaser-plan.yaml"

- parameters for datafaser library setup functions

        datafaser.setup({ "input": { "file": "source.yaml" } })

Former ones overwrite settings in latter ones in above list.

### Parameter name conversions

A setting with name parts `["subject", "detail"]` will be set from

- setup data structure part at `subject: detail:`
- command-line parameter `--subject-detail`
- environment variable `DATAFASER_SUBJECT_DETAIL`.

### References to external sources

External resources such as files or network connections can only be specified within the setup configuration, not in the actual datafaser plan. This is in order to separate the concerns of data processing model on one hand and connections with outside environment on the other.

#### File

    file: "filename.extension" | *number of open file descriptor*
    (type: *json|yaml|xml|...*)

Type can be deduced from filename extension or even contents of the beginning of the stream.

#### Web location

    link: "url"
    (method: *get|put|post|...*)
    (headers: {headers})
    (body: *text or data structure to serialize*)
    (type: *json|yaml|xml|...*)
    (do: *code to run to handle the response*)

- For input, request (GET by default) is made exactly as specified the response is expected to contain a document of specified type.
- For output, default is to POST a body with serialized result.

If `do` is specified, it will be run with

    request:
        *web location data structure*
    response:
        status: *integer HTTP status code*
        headers: {headers}
        body: *text*

### Datafaser plan

Specifying the datafaser plan to run is one part of its setup.

    plan:
    - *step*

Example: read plan from a file:

    plan:
        file: "plan.json"

### Future dreams

These features would provide flexibility for setups to be utilised in a wide variety of environments, but would make the configuration harder to understand and reason about.

#### Setup generation with datafaser

If setup configuration contains `do`, it would be executed as datafaser plan with values set in an optional `with` clause, that can refer to outside resources, and the resulting data structure will be used as the setup configuration.

#### User defined settings for setup

Keyword `settings` in setup configuration could be used to define names and ranges to accept as values in configuration data structure, command line parameters, and environment variables.

#### Listening service

    listen:
        network:
            (address: *ip address or "all"; default is localhost*)
            port: *network port number or service name*
            (protocol: *http; default is plain data*)
        socket: "filename"
        (type: *json|yaml|xml|...*)
        (pidfile: "filename")

Either or both of network and socket may be specified.

Listening on a network address or file system socket will be started during datafaser setup phase, before the plan starts running. Access to any data thereof will block and wait until it can be read from a connecting client. Such server processes are solely the responsibility of datafaser setup; they can not be directly affected by the data processing language.

Specifying same socket file or network endpoint for both input and output will cause datafaser setup to send results for requests from clients as responses for them. In such server mode results for clients detaching before response is sent will be discarded.

#### Buffering

Specifying a buffer size will cause datafaser setup to try to keep up to that much data buffered.

    (buffer:
        bytes: *number of bytes to buffer* |
        items: *number of values to buffer* |
        entries: *number of top-level entries to buffer*

An input with a buffer will try to read pre-emptively to fill it to

- help datafaser run have ready the data it needs to process an item
- help datafaser setup know how wide to scale the run pipeline.

An output with a buffer will try to pull results pre-emptively from the datafaser run behind it, so they can be served as soon as a client comes to get them.

#### Database resource access

In addition to files and streams, databases could be used as sources and targets for data.

    database:
        type: *postgres|mariadb|mongodb...*
        connection_string: ""
        with: {additional named parameters for db client}
        (statement: *select for input, insert for output*)

Without a statement, the mapping might be like

    schema name:
        table name:
        - { field name: value, ...} (list of records)

but then there's the questions

- append or truncate?
- do we really want a full export of all tables every time?

### Input and output formats

#### JSON

JSON collections are a good minimum set to build data transformations around.

Scalar types in JSON are too unspecific, so we need to try to deduce more accurate types somehow.

#### YAML

Only accept data structures and scalars that match what we get out of JSON. Others should cause errors in order to avoid implicit impedance mismatch.

#### XML

Map each node to

    node name:
        attributes: {record of attribute names to values}
        contents: [list of subelements]

#### CSV

List of lists. Provide functions to convert to records based on field names on first n rows and/or m columns. Support different field and record separators, value enclosing and escaping, to support a wide variety of file formats.

#### Text

As is.

#### Other format options

Markdown, TOML, INI files, etc.

## Validation

### JSON Schema

    validate:
        json-schema:

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

    (path, key, value)

## Fases

A program, or `plan`, in datafaser language consists of one or more `fases`, or steps combining data selection and processing that produces some new data based on the selection.

    each:
        path: *selector path producing 0..n selections for which to run rest of this fase*
        as: *name*
    with:
        *name*: *value producer to access other parts of data*
        ...
    do: *value producer*
    else: *value producer for zero selection results*
    failing: *value producer run for each error*

Either or both of `each` and `with` can be specified to form input for the `fase`. A `fase` without either will run once and have no input.

Any of `do`, `else` and `failing` can be omitted. Missing `do` or `else` means no result is produced on selected or skipped entries accordingly. A missing `failing` statement causes each failing turn of `fase` to have the associated `error` as its sole result.

Each value producer can be a single composer command or a complete `fase`.

### Plan

Datafaser run plan is a list of `fases` that each consume the results of previous one to transform it further. The first `fase` gets to select from full input for the program, and the results of the last `fase` form the output of the program.

### Functions

A Datafaser setup can provide a map of function names to `fases` that can then be called as a value producer:

    do: *function name*: *function input*

## Data selection

### Path

A path is a list of keys each referring to the name of a record field or index of list item in the associated collection:

    { path: [ key, ... ] }

For instance, the path to 12 below would be `[0, "age"]`:

    [ { age: 12 } ]

In places where it's unambiguous, you can replace the `path:` record with just a list – it'll be understood by context to be the path.

#### Empty path refers to whole

To handle the whole input in one fell swoop:

    []

#### Selecting each entry separately

Any selector selects each entry it touches separately. Without criteria, default is to accept all input. Thus, to handle every entry of input separately, you can say

    [{}]

And to handle separately each second level entry, you say

    [{}, {}]

General form would be to use a range specifying acceptable depth to make one selector handle multiple levels:

    [{ range: { min: 2, max: 6 } }]

### Selectors

Selector paths are paths where some entries may be selector records instead of static key values.

Individual selectors are records that specify which parts of associated collection will be considered for selection.

Output of each selector and selector path is a list of matching `(path, key, value)` entries.

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

#### Match

Matches text values that consist solely of parts matching tests in a list:

    match:
    - "Once upon a midnight dreary"
    - some: [" while", " I", " pondered"]
      range: { min: 3, max: 3 }
      name: "words"

- `range` allows you to specify how many times a matching step should appear.
- `name` allows you to refer to matched parts in the `do` section associated with a selection, so you can insert them into new texts.

#### More selector criteria

There are plenty more options to consider as potential selection criteria.

#### Combining selectors

Multiple selectors may be considered together for same entry.

##### All

Each of the listed criteria must be fulfilled.

    all: [list of separate criteria]

##### Some

At least one of the listed criteria must be fulfilled.

    some: [list of separate criteria]

##### None

None of the listed criteria must be fulfilled.

    none: [list of separate criteria]

## Composing results

### Composing field values

#### value

Value creates a value as is.

#### Error

creates an error.

    error:
        type: *error type name*
        message: *explanation of this particular error*

Other fields of error are filled in automatically when created.

#### Fill a text or structure

    fill: "template with {{selector}} entries"

    fill: [a list with selectors for values]

    fill: [a map with templates for keys and selectors for values]

#### Combine a value from parts

    combine: [a list of selectors whose results are combined as text]

### Composing data structures

#### list

List combines the results of given value producers to a list.

    list:
    - *value producer*

#### Record

Creates a record.

    record: *name: value producer, ...*

#### merge

Combines given inputs as a common resulting data structure.

    merge:
    - *list of value producers whose results will be merged*

- Each new entry for a record is added to that record.
- Each new entry for a list is appended to that list in producer order.
- Records or lists as record entries with same names are merged together.
- Merging entries with same names but different types is an error.

### Collecting results into records or single values instead of lists

    each:
        path: []
        as: []
    with:
        parallel_handlers:
            - each: [ "long list", { key: { modulo: { divider: 2 } ]
              do: { [parallel task] }
            - each: [ "long list", { key: { modulo: { divider: 2, remainder: 1 } ]
              do: { [parallel task] }
        separate_subtask:
            with:
                separate_data: [ "some subdata" ]
            do: { [first separate subtask] }
        another_separate_subtask:
            with:
                separate_data: [ "other subdata" ]
            do: { [second separate subtask] }
    do:
    - fill:
        all_data_combined:
            some:
                path:
                - "parallel_handlers"
                - key: { range: {} }
                - key: { range: {} }
                path:
                - "separate_subtask"
                path:
                - "another_separate_subtask"

TODO:

- explain
- figure out a way to scale dynamically

### Handling errors

    failing:
        do: *value producer*

Input for an error handler is an error record:

    error:
        type: *error type name*
        message: *text explaining this error and how to fix it*
        plan:
        - *call stack of the datafaser run*
        path: *path to the value associated with error, if any*
        key: *last element of above path, if any*
        value: *value of the erroneous element, if any*
