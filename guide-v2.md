# MessageFormat for JavaScript Developers

`MessageFormat` is a new WIP functionality that would act as a Swiss Army knife for all your i18n needs, allowing you to localize interfaces with great ease.

## What is a "Message"?

Messages are user-visible strings, often with variable elements like names, numbers and dates. Message strings are typically translated into the different languages of a UI, and translators move around the variable elements according to the grammar of the target language.

All messages, including simple static ones, are enclosed in `{…}` delimiters:

TODO: Explain literals, maybe patterns?

**EXAMPLE**
```
{This is a message.}
```

## Evaluating Expressions

An expression represents a dynamic part of a message that will be determined during the message's formatting at runtime.

### Variable Replacement

The most common way to use `MessageFormat` is for simple variable replacement.

**EXAMPLE**
```
{Hello, {$userName}!}
```

### Function Calls

i18n functions such as common formatters can be called as functions within expressions using the following syntax, with the input followed by the function call and finally the options.

**EXAMPLE**
```
{Today is {$date :datetime weekday=long}.}
```

## Built-in Formatters

A number of commonly used formatters are built into all `MessageFormat` implementations for easy access. Formatters are simple in that they format a given locale-independent value (like a standardized date or a number) into a localized string ready to be presented to the user.

### Date and Time Formatting

A date and time formatter that closely mimics JavaScript Intl's [DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat) is available as the `:datetime` function in `MessageFormat`.

**EXAMPLE**
```
{The match on {$date :datetime dateStyle=long} is cancelled.}
```

**OPTIONS**
* `style`: The base display style to be used across the entire date time, possible values: `long`, `short` and `narrow`.
* `dateStyle`: The base display style to be used for the date component, possible values: `long`, `short` and `narrow`.
* `timeStyle`: The base display style to be used for the time component, possible values: `long`, `short` and `narrow`.

TODO: list down everything, assume it'll mimic NF.

### Number Formatting

A number formatter that closely mimics JavaScript Intl's [NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat) is available as the `:number` function in `MessageFormat`.

**EXAMPLE**
```
{Your current account balance is {$amount :number style=currency currency=EUR}.}
```

**OPTIONS**
TODO: list down everything, assume it'll mimic NF.

### List Formatting

A list formatter that closely mimics JavaScript Intl's [ListFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat) is available as the `:list` function in `MessageFormat`.

**EXAMPLE**
```
{Cart contains: {$items :list type=conjunction}.}
```

**OPTIONS**
TODO: list down everything, assume it'll mimic LF.

## Selectors

The second class of functions, apart from formatters are selectors. As opposed to formatters which format an input value inside a pattern, selectors allow you to "select" one of many patterns based on performing some kind of locale-sensitive operation on a value.

### Plural Selection

A plural selector that closely mimics JavaScript Intl's [PluralRules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/PluralRules) is available as the `:plural` function in `MessageFormat`.

**EXAMPLE**
```
match {$count :plural}
when one {One new message}
when other {{$count :number} new messages}
```