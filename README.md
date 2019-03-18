# Datafaser language

## License

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.

Full license: see `LICENSE` file
Explanation: https://creativecommons.org/licenses/by-sa/4.0/

You are free to:

- Share — copy and redistribute the material in any medium or format

- Adapt — remix, transform, and build upon the material for any purpose, even commercially.

Under the following terms:

- Attribution — You must give appropriate credit, provide a link to the license, and indicate if changes were made. You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.

- ShareAlike — If you remix, transform, or build upon the material, you must distribute your contributions under the same license as the original.

- No additional restrictions — You may not apply legal terms or technological measures that legally restrict others from doing anything the license permits.

## Purpose

Datafaser is a programming language limited to producing new data from existing data. Purpose of the language and its restriction is to have a tool as clear and concise as possible, so that programs written in it could be well understood and reasoned about. Hopefully it would allow many problems to be solved more easily and with less mistakes than can usually be achieved with general purpose programming languages.

## Overview

### Example

Run:

    datafaser --from-file source.yaml --run-file greet.yaml --output-format=yaml

Data source file `source.yaml` contents:

    greeting: Hello
    greet:
       - World
       - Life

Datafaser plan file `greet.yaml` contents:

    - each:
          name: { path: [ "greet" ] }
      with:
          greeting: { path: [ "greeting" ] }
      do:
          fill: "{{greeting}}, {{name}}!"

Result:

    - "Hello, World!"
    - "Hello, Life!"

### Source or input data

Datafaser works on textual and numeric data in records and lists, like this:

    [
        { "name": "Universe", "age": 13.8e9 },
        { "name": "Earth", "age": 4.5e7 }
    ]

This and many other examples use [JSON](https://www.json.org/) format to represent the data and its structure.

Many different data formats can be converted to `JSON` or equivalent formats. Datafaser language specification includes some ways to convert some of them, such as `YAML`, `XML`, and `CSV`. Datafaser runners (implementations: interpreters, compilers, and like) may provide their own specifications that support even more formats. For example, companies with their own, proprietary data formats may provide their own Datafaser specifications and runners that support them. This may enable interoperability better than providing only a format specification such as `OOXML`.

Datafaser runners may be written to operate on large data sets by

- operating on streams of input
- launching remote instances (in cloud) to process different parts of the input data.

### Result or output data

The result of a datafaser run will be whatever the last step in its plan produces:

1. It can be a single number or text, that can be written into a file or shown on screen.
2. It can as well be a large data structure, that can then be represented in some format such as `JSON` or written as records in a database.
3. Nested records can also be stored as directories where each field with a value other than a record will be written into a file of same name.
4. A list of results may be continuously streamed from streaming sources through a service running datafaser into some consumer, such as trading activities or patient health telemetry for associated response systems.

### Data transformation plan

A plan (or "program written in datafaser language") consists of one or more steps (or "fases"). Each step produces some set of data by

1. selecting parts of the data it gets as input
2. producing as its output some new data based on the selected data.

- A step that does not select any part operates on all data it gets.
- A step that does not alter the data it gets or selects passes it on as is.
- A step that does not produce any data stops any further steps from running.
- A step without a selection nor production is valid, and all data just passes through it. (It may serve as, say, placeholder or comment.)

Some methods will be specified for

- using same data as input for multiple separate steps.
- combining the results of multiple separate steps.

### Datafaser program runner

Datafaser can be implemented as a set of libraries, server, or standalone utility program in general-purpose languages (or perhaps one day in itself) as an interpreter, transpiler, or compiler. Interactive or visual interfaces may be created on top of same libraries as well.

Consider

- streaming with possibility to optimize memory usage by tracking what data will no longer be used
- ease and clarity of implementation to help adapting (each Datafaser release shall have a comprehensive test set)
- efficiency classification of implementations to clarify tradeoffs

### Errors

Errors that happen when a datafaser plan runs are represented inside it as a separate data type, that can then be handled as necessary within the program, and represented separately from the results.

## Etymology

Name `Datafaser` tries to emphasise

1. discrete steps, or phases, in processing data;
2. discrete phases of data between processing steps;
3. facing data over staring at code.
