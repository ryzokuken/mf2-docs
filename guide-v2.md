# MessageFormat for JavaScript Developers

`MessageFormat` is a new WIP functionality that would act as a Swiss Army knife for all your i18n needs, allowing you to localize interfaces with ease.

## What is a "Message"?

Messages are user-visible strings, often with variable elements like names, numbers and dates. Message strings are typically translated into the different languages of a UI, and translators move around the variable elements according to the grammar of the target language.

All messages, including simple static ones, are enclosed in `{…}` delimiters:

**EXAMPLE**
```
{This is a message.}
```

## Evaluating Expressions

An expression represents a dynamic part of a message that will be determined during the message's formatting at runtime.

### Variable Replacement

The most common way to use MessageFormat is for simple variable replacement.

**EXAMPLE**
```
{Hello, {$userName}!}
```

### Function Calls

i18n functions such as common formatters can be called within expressions.

**EXAMPLE**
```
{Today is {$date :datetime weekday=long}.}
```

## Built-in Functions

A number of commonly used functions are built into all `MessageFormat` implementations for easy access.

### Date Formatting

