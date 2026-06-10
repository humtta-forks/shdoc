# shdoc — Bash Script Documentation Format Specification

**Version:** 1.0.0  
**Status:** Final  
**Published:** 10 June 2026

---

## Overview

`shdoc` is a structured inline comment documentation format for Bash scripts, inspired by JavaDoc. It defines a consistent way to annotate functions, variables, and constants directly in source code, enabling both human readability and automated documentation generation.

The format is designed to be minimally intrusive: documentation blocks are plain `#`-prefixed comments that require no special tooling to read in source form.

## Design Principles

- **Plain `#` comments only** — no special comment delimiters (`##`, `#!`, `#|`)
- **Structured, not freeform** — every block has a defined set of named fields
- **Concise** — each field fits on one line; multi-line descriptions use stacked `#` lines
- **Machine-parseable** — field names follow a fixed keyword syntax (`# Key: Value`)
- **Separation of public and private** — a dash marker (`# -`) signals internal/private entities

## Comment Block Structure

A shdoc block is a contiguous sequence of `#`-prefixed lines placed **immediately above** the function or variable declaration it documents. The block ends at the first non-comment line.

```bash
# Short one-line description of what this does
#
# 1: First argument name (Type)
# 2: Second argument name (Type) [Optional]
# *: Variadic arguments (Type)
#
# Code: Yes
# Echo: No
#
# Example:
# someFunction "arg1" "arg2"
someFunction() {
  ...
}
```

## Block Sections

A documentation block consists of the following sections, in this order:

### 1. Description

A plain-text summary of the entity's purpose. May span multiple lines using stacked `#` comment lines. Separated from the rest of the block by a blank comment line (`#`).

```bash
# Short description on one line.
#
# Optional extended description paragraph.
# It may span multiple lines.
```

### 2. Parameters

Documents positional arguments passed to the function. Each parameter occupies one line:

```
# N: Name (Type) [Optional]
```

- **N** — positional index (`1`, `2`, `3`, …), or `*` for variadic (all remaining arguments)
- **Name** — human-readable parameter name
- **Type** — one of the [standard types](#type-system) in parentheses
- **[Optional]** — literal suffix when the argument may be omitted

```bash
# 1: Username (String)
# 2: Port (Number) [Optional]
# *: Additional flags (String)
```

If the function takes no parameters, this section is omitted entirely.

### 3. Return Value Fields

Two mandatory fields describe what the function communicates back to the caller:

| Field   | Allowed values     | Meaning                                              |
|---------|--------------------|------------------------------------------------------|
| `Code`  | `Yes` / `No`       | Whether the function returns a meaningful exit code  |
| `Echo`  | Description or `No`| What is printed to stdout, or `No` if nothing        |

```bash
# Code: Yes
# Echo: No
```

```bash
# Code: No
# Echo: PID (Number)
```

When a function both sets an exit code **and** echoes output, both fields are populated:

```bash
# Code: Yes
# Echo: Matched line content (String)
```

### 4. Example (Optional)

An optional section showing one or more usage examples. Introduced by the `Example:` keyword on its own line; example code lines follow as `#`-prefixed lines.

```bash
# Example:
# local pid=$(GetPID)
# SendSignal $pid
```

Multiple examples may be stacked without a separator:

```bash
# Example:
# AddCommand "check" "Check config" "myCheckFunc" true
# AddCommand "kill" "Kill connections" "myKillFunc"
```

## Private / Internal Marker

Entities that are **internal implementation details** and not part of the public API are marked with a lone dash on the first line of their doc block:

```bash
# -
#
# Description of the internal function
#
# Code: No
# Echo: No
internalHelper() {
  ...
}
```

The `# -` marker serves the same role as `@private` in JavaDoc. It signals that the entity should be excluded from generated public documentation.

When both a dash marker and a description are present, the dash occupies a line by itself followed by a blank comment separator before the description.

## Variable and Constant Documentation

Scalar variables and constants are documented with a single-line comment placed **immediately above** the declaration:

```bash
# Status code for working service
STATUS_RUNNING=1

# Time in seconds for service start (Number)
kv[delay_start]=15
```

The type annotation in parentheses is optional but recommended for non-obvious types.

For **private or implementation-detail** variables that should be excluded from generated docs, use the dash prefix on the comment:

```bash
# -
# Template for temporary data
TMP_TEMPLATE="/tmp/kaosv.XXXXXXXX"
```

### Associative Array / Config Key Documentation

Keys of associative arrays used as configuration objects follow the same single-line pattern:

```bash
# Enable/disable colored output (Boolean)
kv[use_colors]=true

# Path to log file (String)
kv[log]=""
```

## Type System

shdoc uses a small set of standard type names. Types are always written in **Title Case** inside parentheses.

| Type      | Description                                                                          |
|-----------|--------------------------------------------------------------------------------------|
| `String`  | A text value                                                                         |
| `Number`  | An integer value                                                                     |
| `Boolean` | A true/false value (`true`/`false` for variables; exit code semantics for `Code:`)   |
| `List`    | A space-separated list of values                                                     |

For `Echo` return values, the type follows the description in parentheses:

```bash
# Echo: Owner name (String)
# Echo: Process ID (Number)
```

## Complete Annotated Example

```bash
# Find service PID
#
# Code: No
# Echo: PID (Number)
#
# Example:
# local pid=$(FindPID)
FindPID() {
  ...
}

# Send a signal to service
#
# 1: Signal (String)
# 2: PID (Number) [Optional]
#
# Code: Yes
# Echo: No
#
# Example:
# if ! SendSignal $SIGNAL_QUIT ; then
#   Error "Can't send signal"
# fi
SendSignal() {
  ...
}

# Read property from file
#
# 1: File (String)
# 2: Property name (String)
# 3: Delimiter (String) [Optional]
#
# Code: No
# Echo: Property value (String)
#
# Example:
# local port=$(ReadProperty "/path/to/my.conf" "port")
# local host=$(ReadProperty "/path/to/my.conf" "host" ":")
ReadProperty() {
  ...
}

# -
#
# Internal method that builds ulimit command string
#
# Code: No
# Echo: Command (String)
getLimitsCmd() {
  ...
}
```

## File-Level Documentation

File-level documentation uses a comment block at the top of the script, before any code. It is a freeform description block without structured fields:

```bash
#!/usr/bin/env bash

# Short one-line summary of the script.
#
# Extended description of the script's purpose,
# intended audience, and usage context.
```

## Section Separators

Visual separator lines of repeated `#` characters may be used to divide logical sections of a file. These are purely cosmetic and carry no semantic meaning:

```bash
################################################################################
```

## Field Reference Summary

| Field       | Required        | Applies To      | Format                                  |
|-------------|-----------------|-----------------|-----------------------------------------|
| Description | Yes             | Functions, Vars | Plain text, one or more `#` lines       |
| `N:`        | When args exist | Functions       | `# N: Name (Type) [Optional]`           |
| `*:`        | When variadic   | Functions       | `# *: Name (Type)`                      |
| `Code:`     | Yes             | Functions       | `# Code: Yes` or `# Code: No`           |
| `Echo:`     | Yes             | Functions       | `# Echo: Description (Type)` or `No`   |
| `Example:`  | No              | Functions       | `# Example:` followed by code lines    |
| `# -`       | No              | Any entity      | First line of block to mark as private |

## Alias / Compatibility Stub Documentation

When a function is a pure alias or thin wrapper for another function (for backwards compatibility), it should be marked with `# -` and only a brief one-line description or no description at all, deferring full documentation to the canonical function:

```bash
# -
getPid() {
  findPID "$@"
}
```

---

## Grammar (EBNF)

```ebnf
doc_block        = private_marker? description_block? param_section? return_section example_section?
private_marker   = "# -" newline blank_comment
description_block = description_line+ blank_comment
description_line  = "# " text newline
blank_comment    = "#" newline
param_section    = param_line+ blank_comment
param_line       = "# " index ": " name " (" type ")" optional_marker? newline
index            = digit+ | "*"
optional_marker  = " [Optional]"
return_section   = code_line echo_line blank_comment?
code_line        = "# Code: " ("Yes" | "No") newline
echo_line        = "# Echo: " (return_desc | "No") newline
return_desc      = text " (" type ")"
example_section  = "# Example:" newline example_line+
example_line     = "# " code newline
type             = "String" | "Number" | "Boolean" | "List"
```