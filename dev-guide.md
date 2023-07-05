# MessageFormat Developer Guide

This guide aims to provide an introduction to ICU MessageFormat syntax, as supported by the MessageFormat v2 project.

## String Lookup

<!-- An expression always uses {…} delimiters. An expression can appear as a local variable value, as a selector, and within a pattern. -->
The simplest case of MessageFormat involves no formatting, just a string passthrough.
All messages, including simple static ones, are enclosed in `{…}` delimiters:

```
{This is a message.}
```

## Expressions

An expression represents a dynamic part of a message that will be determined during the message's formatting at runtime.

The most common way to use MessageFormat is for simple variable replacement.
Whitespace (including newlines) is ignored when not in literal mode.

A simple expression is a bare variable name:

```
{Hello, {$userName}!}
```

## Functions

Functions are the building block of MessageFormat. A function is named functionality, possibly with options, that format, process, or operate on a variable.

MessageFormat functions can be invoked in two contexts:

* inside placeholders, to produce a part of the message's formatted output; for example, a raw value of `|1.5|` may be formatted to `1,5` in a language which uses commas as decimal separators,
* inside selectors, to contribute to selecting the appropriate variant among all given variants.

Furthermore, functions can behave differently based on context despite having a single name and many functions can be used, for instance, for both formatting and selecting values.

For example, a message with an interpolated $date variable formatted with the :datetime function:

```
{Today is {$date :datetime weekday=long}.}
```

### Prefix sigils

TODO: Include a decent non-markup example to demonstrate their utility. 

Functions use one of the following prefix sigils:

- `:` for standalone content
- `+` for starting or opening elements
- `-` for ending or closing elements

A message with two markup-like _functions_, `button` and `link`, which the runtime can use to construct a document tree structure for a UI framework:

    {{+button}Submit{-button} or {+link}cancel{-link}.}

An opening element MAY be present in a message without a corresponding closing element, and vice versa.

### Built-in Functions

TODO: There is still some uncertainity whether Intl-style builtins are prefereable to Skeleton-style or vice versa. It's important to drive to some consensus here and update the document accordingly. Currently, we're documenting the Intl-style variants.

There are some standard functions that are widely applicable and are therefore included by MessageFormat implementations.

#### `platform`

It's a simple selector function that matches the current OS, which might be important for an application. Common values include `windows`, `linux`, `macos`, `android` and `ios`.

An example invocation would look like:

```
match {:platform} when windows {Settings} when * {Preferences}
```

#### `number`

NOTE: similar to `NumberFormat`

It can be used to:
* format a number for a locale
* match a numerical value against CLDR plural categories or a number

In either case, the following options are always accepted by the function:
* minimumIntegerDigits
* minimumFractionDigits
* maximumFractionDigits
* minimumSignificantDigits
* maximumSignificantDigits

All of the above options accept a positive integer value. Furthermore, the formatting use-case accepts two more options:
* style: should be one of `decimal`, `currency`, `percent` or `unit`
* currency: should be a well-formed currency code

Example usage that demonstrates both variants of the function:

```
match {$count :number}
when 1 {One new message}
when * {{$count :number} new messages}
```

#### `plural`

NOTE: similar to `PluralRules`

TODO

#### `list`

NOTE: similar to `ListFormat`

TODO

#### `datetime`

NOTE: similar to `DateTimeFormat`

TODO

#### `duration`

NOTE: similar to `DurationFormat`

TODO

#### Others

Placeholder section, other common builtins include: `noun`, `adjective`, `term`

| Builtin     | ICU4J | ICU4C | Polyfill | message2 |
| ----------- | ----- | ----- | -------- | -------- |
| `literal`   | ✅    |       | ✅       | ✅       |
| `markup`    |       |       | ✅       |          |
| `number`    | ✅    |       | ✅       | ✅       |
| `datetime`  | ✅    |       | ✅       |          |
| `plural`    | ✅    |       |          | ✅       |
| `list`      |       |       |          | ✅       |
| `noun`      |       |       |          | ✅       |
| `adjective` |       |       |          | ✅       |
| `person`    |       |       |          | ✅       |


### Custom Functions

MessageFormat allows you to specify extension points via custom functions in a portable manner through the "Function Registry".

The registry provides a machine-readable description of MessageFormat extensions (custom functions). It contains descriptions of function signatures as defined in the [data model syntax](https://github.com/unicode-org/message-format-wg/blob/main/spec/registry.dtd) and [documentation](https://github.com/unicode-org/message-format-wg/blob/main/spec/registry.md).

An example of a user-defined `customRegistry.xml` could look like the following:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE registry SYSTEM "./registry.dtd">

<registry>
    <function name="noun">
        <description>Handle the grammar of a noun.</description>
        <formatSignature locales="en">
            <input/>
            <option name="article" values="definite indefinite"/>
            <option name="plural" values="one other"/>
            <option name="case" values="nominative genitive" default="nominative"/>
        </formatSignature>
    </function>

    <function name="adjective">
        <description>Handle the grammar of an adjective.</description>
        <formatSignature locales="en">
            <input/>
            <option name="article" values="definite indefinite"/>
            <option name="plural" values="one other"/>
            <option name="case" values="nominative genitive" default="nominative"/>
        </formatSignature>
        <formatSignature locales="en">
            <input/>
            <option name="article" values="definite indefinite"/>
            <option name="accord"/>
        </formatSignature>
    </function>
</registry>
```

Messages can now use the `:noun` and the `:adjective` functions. The following message references the first signature of `:adjective`, which expects the plural and case options:

```
{You see {$color :adjective article=indefinite plural=one case=nominative} {$object :noun case=nominative}!}
```

## Declarations

A _message_ can define local variables for transforming input or providing additional data to an _expression_. Local variables appear in a _declaration_, which defines the value of a named local variable.

A _message_ containing a _declaration_ defining a local variable `$whom` which is then used twice inside the pattern:

    let $whom = {$monster :noun case=accusative}
    {You see {$quality :adjective article=indefinite accord=$whom} {$whom}!}

A message defining two local variables: `$itemAcc` and `$countInt`, and using `$countInt` as a selector:

    let $countInt = {$count :number maximumFractionDigits=0}
    let $itemAcc = {$item :noun count=$count case=accusative}
    match {$countInt}
    when one {You bought {$color :adjective article=indefinite accord=$itemAcc} {$itemAcc}.}
    when * {You bought {$countInt} {$color :adjective accord=$itemAcc} {$itemAcc}.}

## Selectors

A selector selects a specific pattern from a list of available patterns in a message based on the value of its expression. A message can have multiple selectors.

Think of selectors like a `switch` statement for your messages.
A common use-case is used to select gender in a string.

A message with a single _selector_:

    match {$count :number}
    when 1 {You have one notification.}
    when * {You have {$count} notifications.}

A message with a single _selector_ which is an invocation of
a custom function `:platform`, formatted on a single line:

    match {:platform} when windows {Settings} when * {Preferences}

A message with a single _selector_ and a custom `:hasCase` function
which allows the message to query for presence of grammatical cases required for each variant:

    match {$userName :hasCase}
    when vocative {Hello, {$userName :person case=vocative}!}
    when accusative {Please welcome {$userName :person case=accusative}!}
    when * {Hello!}

A message with 2 _selectors_:

    match {$photoCount :number} {$userGender :equals}
    when 1 masculine {{$userName} added a new photo to his album.}
    when 1 feminine {{$userName} added a new photo to her album.}
    when 1 * {{$userName} added a new photo to their album.}
    when * masculine {{$userName} added {$photoCount} photos to his album.}
    when * feminine {{$userName} added {$photoCount} photos to her album.}
    when * * {{$userName} added {$photoCount} photos to their album.}

<!-- 
## PluralFormat

PluralFormat is a similar mechanism to SelectFormat, but specific to numerical values.
The key that is chosen is generated from the specified input variable by a locale-specific _plural function_.

The numeric input is mapped to a plural category, some subset of `zero`, `one`, `two`, `few`, `many`, and `other` depending on the locale and the type of plural.
English, for instance, uses `one` and `other` for cardinal plurals (e.g. "one result", "many results") and `one`, `two`, `few`, and `other` for ordinal plurals (e.g. "1st result", "2nd result", etc).
For information on which keys are used by your locale, please refer to the [CLDR table of plural rules](https://unicode-org.github.io/cldr-staging/charts/latest/supplemental/language_plural_rules.html).

For some languages, the number of printed digits is significant (e.g. "1 meter", "1.0 meters"); to account for that you may pass in the number using its stringified representation of the number (as produced by `String(n)`).

Matches for exact values are available with the `=` prefix, e.g. `=0` and `=1`.

The keyword for cardinal plurals is `plural`, and for ordinal plurals is `selectordinal`.

Within a plural statement, `#` will be replaced by the variable value, formatted as a number in the current locale.

```
{ COUNT, plural,
    =0 {There are no results.}
    one {There is one result.}
    other {There are # results.}
}
```

```js
msg({ COUNT: 0 }); // 'There are no results.'
msg({ COUNT: 1 }); // 'There is one result.'
msg({ COUNT: 100 }); // 'There are 100 results.'
```

```
You are { POS, selectordinal,
          one {#st}
          two {#nd}
          few {#rd}
          other {#th}
        } in the queue.
```

```js
msg({ POS: 1 }); // 'You are 1st in the queue.'
msg({ POS: 33 }); // 'You are 33rd in the queue.'
``` -->