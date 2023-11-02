# MessageFormat for JavaScript Developers

`MessageFormat` is a new WIP functionality that would act as a Swiss Army knife for all your i18n needs, allowing you to localize interfaces with great ease.

## What is a "Message"?

Messages are user-visible strings, often with variable elements like names, numbers and dates. Message strings are typically translated into the different languages of a UI, and translators rearrange the variable elements according to the grammar of the target language.

Simple (static) messages can be written without special syntax, since the default mode is "text mode".

**EXAMPLE**
```
This is a message.
```

For more complex messages, you need to switch into "code mode" by using a set of double braces (`{{...}}`). This is syntax you might be familiar with from templating languages.

**EXAMPLE**
```
{{
    match {$userType}
    when guest {{Welcome Guest!}}
    when registered {{Welcome {$username}!}}
}}
```

<!-- TODO: explain literals? -->

## Expressions

An expression represents a dynamic part of a message that will be determined during the message's formatting at runtime. Expressions are enclosed within a single set of braces (`{...}`).

### Variable Replacement

The most common way to use `MessageFormat` is for simple variable replacement.

**EXAMPLE**
```
Hello, {$userName}!
```

### Annotations

An annotation is part of an expression containing either a function together with its associated options, or a private-use or reserved sequence.

An annotation can appear in an expression by itself or following a single operand. If an operand is present, that operand is the input to the annotation.

#### Function calls

i18n functions such as common formatters can be called as functions within expressions using the following syntax, with the operand followed by the function name and finally the options.

**EXAMPLE**
```
Today is {$date :datetime weekday=long}.
```

#### Private-use

A private-use annotation is an annotation whose syntax is reserved for use by a specific implementation or by private agreement between multiple implementations. Implementations define their own meaning and semantics for private-use annotations.

A private-use annotation starts with either `&` or `^`.

#### Reserved

A reserved annotation is an annotation whose syntax is reserved for future standardization.

A reserved annotation starts with a reserved character, which would be one of: `!`, `@`, `#`, `%`, `*`, `<`, `>`, `/`, `?` or `~`.

## Declarations

A declaration binds a variable identifier to a value within the scope of a message. This variable can then be used in other expressions within the same message. Declarations are optional: many messages will not contain any declarations.

An **input** declaration binds a variable to an external input value.

A **local** declaration binds a variable to the resolved value of an expression.

Unlike in JavaScript, declared variables cannot be used before their declaration, and their values mustn't be self-referential; otherwise, a message is not considered valid. Declaring the same variable multiple times results in an invalid message as well.

**EXAMPLE**
```
input {$x :function option=value}
local $y = {|This is an expression|}
```

## Patterns

A pattern is a sequence of text and placeholders (also called expressions) to be formatted as a unit. Unless there is an error, the result of formatting a message is always the result of formatting a single pattern: either the single pattern that makes up a simple message, or the pattern of the matching variant in a complex message.

### Quoted Patterns

A quoted pattern is a pattern that is enclosed in double braces (`{{...}}`). Quoting may be necessary because a pattern may contain characters that have a special meaning in the MessageFormat syntax, and the quotes make it clear that these characters should be interpreted literally. The current design of the proposal requires *all* patterns in complex messages to be quoted.

### Text

Text is the translatable content of a pattern. Any Unicode code point is allowed, except for surrogate code points U+D800 through U+DFFF inclusive. The characters `\`, `{`, and `}` must be escaped.

Note that whitespace in text, including tabs, spaces, and newlines is significant and will be preserved during formatting.

**EXAMPLE**
```
{{
    input {$num :number}
    {{   This is the {$num} pattern   }}
}}
```

## Built-in Formatters

A number of commonly used formatters are built into all `MessageFormat` implementations for easy access. Formatters are simple in that they format a given locale-independent value (like a standardized date or a number) into a localized string ready to be presented to the user.

### Date and Time Formatting

A date and time formatter that closely mimics JavaScript Intl's [DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat) is available as the `:datetime` function in `MessageFormat`.

**EXAMPLE**
```
The match on {$date :datetime dateStyle=long} is cancelled.
```

**OPTIONS**
* `style`: The base display style to be used across the entire date time, possible values: `long`, `short` and `narrow`.
* `dateStyle`: The base display style to be used for the date component, possible values: `long`, `short` and `narrow`.
* `timeStyle`: The base display style to be used for the time component, possible values: `long`, `short` and `narrow`.

<!-- TODO: list down everything, assume it'll mimic DTF. -->

### Number Formatting

A number formatter that closely mimics JavaScript Intl's [NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat) is available as the `:number` function in `MessageFormat`.

**EXAMPLE**
```
Your current account balance is {$amount :number style=currency currency=EUR}.
```

**OPTIONS**
<!-- TODO: list down everything, assume it'll mimic NF. -->

### List Formatting

A list formatter that closely mimics JavaScript Intl's [ListFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat) is available as the `:list` function in `MessageFormat`.

**EXAMPLE**
```
Cart contains: {$items :list type=conjunction}.
```

**OPTIONS**
<!-- TODO: list down everything, assume it'll mimic LF. -->

## Selectors

The second class of functions, apart from formatters are selectors. As opposed to formatters which format an input value inside a pattern, selectors allow you to "select" one of many patterns based on performing some kind of locale-sensitive operation on a value.

### Plural Selection

A plural selector that closely mimics JavaScript Intl's [PluralRules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules) is available as the `:plural` function in `MessageFormat`.

**EXAMPLE**
```
{{
    match {$count :plural}
    when one {{One new message}}
    when other {{{$count :number} new messages}}
}}
```
