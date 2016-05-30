xschema
=======

  * [Official Repository: exjs/xschema](https://github.com/exjs/xschema)
  * [Official Chat (gitter)](https://gitter.im/exjs/exjs)
  * [Public Domain (https://unlicense.org)](https://unlicense.org)

The xschema library is a high performance data processing and validation library based on an extensible model/schema builder. It allows to build a schema that can be then used to process and validate any kind of JavaScript data (the root variable can be object, array, or any other primitive type). The library is designed for critical areas where the performance is important and even a minor overhead at validation side can cause service delays. The data validation and processing has been moved into extreme by using a JavaScript code-generator that generates the best possible data processing and validating functions for any user-defined schema. The most used JavaScript engines today have built-in JIT compiler so the code generated by xschema is then compiled by the VM into a machine code that will execute very fast and outperform all JavaScript data processing libraries that don't use such technique.

The performance is not the only aspect and feature offered by xschema. The library has been designed in a way that it should be very straightforward to define a schema, to reuse or inherit the existing one, and to create your own types that will extend the built-in functionality. There is no library that could satisfy all possible needs and use-cases so the possibility to extend the library is important. The library provides the most important types as full-featured built-ins.

The schema structure is always declarative and most of the schemas can be serialized back to JSON (xschema calls it a normalized JSON). The library also allows to associate a custom information called `metadata` with any field. Metadata is completely ignored by xschema library, but other tools can take advantage of it (for example you can associate a SQL table names with your entities and use them in your DB layer).

Additionally, xschema has several data processing options that help to deal with common problems like implementing data insertion, updating, deletion, and querying. Processing options can also be used to filter out objects' properties that are not defined (useful when extracting information from request's body or from more objects mixed together) and to accumulate all validation errors to have complete report of the validation.


Disclaimer
----------

The xschema library has been designed to solve common, but also very specific problems. It's very fast and the support for metadata allows to simply extend it by new features. All built-in features are used in production and you will find many of them handy when implementing web services that do CRUD operations, because a single schema can be used to validate data that is inserted, updated, queried, or deleted. The library has been designed to be very fast, but is also very complete and configurable.


Introduction
------------

the xschema library uses a declarative approach to build schemas, but it comes with its own syntax instead of relying on existing solutions like JSONSchema. The main reason for such move was to simplify the way schemas are defined by introducing shortcuts and directives that start with `$` character. Shortcuts are used to simplify declaration of the most common concepts (for example an array of integers can be written as `$type: "int[]"`) and directives are used to configure the type itself. Object's members are always defined without a `$` prefix, but it's possible to define also members that start with `$` by escaping it as `\\$` (escaping and schema normalization is explained later).

```js
var PersonSchema = xschema.schema({
  firstName  : { $type: "text", $maxLength: 64 },
  lastName   : { $type: "text", $maxLength: 64 },
  dateOfBirth: { $type: "date", $leapYear: false },

  active     : { $type: "bool" },
  score      : { $type: "int", $min: 0 },
  keywords   : { $type: "text[]" },
  bashrc     : { $type: "string", $maxLength: 4096 },

  address: {
    line1    : { $type: "text" },
    line2    : { $type: "text" },
    city     : { $type: "text" },
    zip      : { $type: "text" },
    country  : { $type: "text" }
  }
});
```

The example above defines a schema called `PersonSchema`, which is an `object` holding properties of various types specified by `$type` directive. Careful readers have noticed that the root object and nested `address` object have omitted the `$type` directive. It automatically uses `object` if no `$type` is provided, which allows to remove some verbosity in the schema declaration. Other directives like `$min`, `$max`, `$leapYear`, ..., are used to configure the type itself.

Confused by `string` vs `text` type? Well, `string` is _any_ string in JavaScript in contrast to `text`, which is a string that doesn't contain `\u0000-\u0008`, `\u000B-\u000C`, and `\u000E-\u001F` characters. These characters have special meaning and in many cases their presence in your application's data is unwanted and may be dangerous.

Confused by `[]` suffix in `keywords` member? It's a xschema shortcut that defines an array, which can also be defined by using `array` type like this:

```js
var KeywordsSchema = xschema.schema({
  $type: "array",
  $data: {
    $type: "text"
  }
});

var KeywordsSchema = xschema.schema({
  $type: "text[]"
});
```

Both schemas defined above are equivalent and internally normalized into the same structure.


Data Processing Concepts
------------------------

The library comes with two base concepts that are used to work with data.

  - **`xschema.process(...)`** is a concept used to create a new data based on existing data. It's very useful in cases that more entities are mixed together in a single object and you need to separate/extract their content into independent objects. This happens for example in a request-body object. Data processing does not just validate the input data, but it can also sanity it before creating the output. If configured, you can trim/simplify input text, remove unknown properties, or insert fields having default values if they are not present.

  - **`xschema.test(...)`** is a concept used to test whether the given data conforms to the schema, but without using sanitizers.


Data Processing Options
-----------------------

Several data processing options exist that allow to use a single schema for multiple purposes. The default `xschema.kNoOptions` specifies no options and the schema is processed in a default way (i.e. strict mode).

The additional options are used to control:

  - **Extraction Mode** - Options `xschema.kExtractTop`, `xschema.kExtractNested`, and `xschema.kExtractTop` are used to control data extraction. Data extraction means extracting only specified properties from objects that can contain more properties that are not defined in the schema. It's useful when extracting parameters from a request body or to extract data that is known by schema, but without failing in cases that there is something that is not not known.

  - **Delta Mode** - Option `xschema.kDeltaMode` can be used to force nearly all properties to be optional. This is used in cases that you allow delta updates, but you still require some properties that specify DB keys to be present.

  - **Error Accumulation** - Option `xschema.kAccumulateErrors` is used in case that you want to get all errors that happened during data processing, but just the first one.

Data extraction options:

  - **`xschema.kExtractTop`** - Extract from top-level object only.
  - **`xschema.kExtractNested`** - Extract from nested object(s) only.
  - **`xschema.kExtractAll`** - Combination of `xschema.kExtractTop` and `xschema.kExtractNested`, which results in extraction from any `"object"`.

TODO


Built-In Data Types
-------------------

The following data types are built-in:

Type and Aliases         | JS Type    | Description
:----------------------- | :--------- | :---------------------------------------
`any`                    | `any`      | Any type (variant)
`array`                  | `array`    | Array type
`map`                    | `object`   | Map type
`object`                 | `object`   | Object type (default)
`boolean`, `bool`        | `boolean`  | Boolean
`double`, `number`       | `number`   | Double precision floating point number
`numeric`                | `number`   | Double precision, used to distinguish from `number` when specifying a DB columns
`int8`                   | `number`   | 8-bit signed integer
`uint8`                  | `number`   | 8-bit unsigned integer
`int16`, `short`         | `number`   | 16-bit signed integer
`uint16`, `ushort`       | `number`   | 16-bit unsigned integer
`int24`                  | `number`   | 24-bit signed integer
`uint24`                 | `number`   | 24-bit unsigned integer
`int32`                  | `number`   | 32-bit signed integer
`uint32`                 | `number`   | 32-bit unsigned integer
`int53`                  | `number`   | 53-bit signed integer (safe integer)
`uint53`                 | `number`   | 53-bit unsigned integer (safe integer)
`int`, `integer`         | `number`   | signed integer (unsafe integer)
`uint`                   | `number`   | unsigned integer (unsafe integer)
`lat`, `latitude`        | `number`   | Latitude value (-90...90)
`lon`, `longitude`       | `number`   | Longitude value (-180...180)
`char`                   | `string`   | String containing exactly 1 character
`string`                 | `string`   | Any string
`text`                   | `string`   | Restricted multi-line string
`textline`               | `string`   | Restricted single-line string
`bigint`                 | `string`   | String that contains integer of unlimited precision
`int64`                  | `string`   | Bigint limited to 64 bits (signed)
`uint64`                 | `string`   | Bigint limited to 64-bits (unsigned)
`time`                   | `string`   | Time without milliseconds
`time-ms`                | `string`   | Time with milliseconds (ms)
`time-us`                | `string`   | Time with microseconds (μs)
`date`                   | `string`   | Date
`datetime`               | `string`   | Date and time without milliseconds
`datetime-ms`            | `string`   | Date and time with milliseconds (ms)
`datetime-us`            | `string`   | Date and time with microseconds (μs)
`color`                  | `string`   | Color values specified by `"#RGB"`, `"#RRGGBB"`, or a CSS name
`creditcard`             | `string`   | Credit-card number
`mac`                    | `string`   | MAC address
`ip`                     | `string`   | IPV4 or IPV6 address
`ipv4`                   | `string`   | IPV4 address
`ipv6`                   | `string`   | IPV6 address
`isbn`                   | `string`   | ISBN identifier (either ISBN-10 or ISBN-13)
`isbn10`                 | `string`   | ISBN-10 identifier
`isbn13`                 | `string`   | ISBN-13 identifier
`uuid`                   | `string`   | UUID or GUID


Any
---

Any `$type` is specified as `any`.

Any type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$allowed`               | `any[]`    | `null`  | Array of values that are allowed. Any type allows to put anything into the `$allowed` array. If an array or object is put in there a deep comparison will be performed to verify if the input data conforms to it


Boolean
-------

Boolean `$type` is specified as `bool` or `boolean`.

Boolean type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$allowed`               | `bool[]`   | `null`  | Array of boolean values that are allowed. This is useful to restrict the value to be always `true` or `false`, but it does nothing if the array is empty or both `true` and `false` values are specified.


Number and Integer
------------------

Number type `$type` is specified by the following type names and properties:

Type and Aliases         | Minimum Value     | Maximum Value    | Description
:----------------------- | :---------------- | :--------------- | :-------------
`double`, `number`       | None              | None             | Double precision floating point
`numeric`                | None              | None             | Numeric value (alias to double, but can be used to distinguish between double and numeric in case of describing DB schema)
`int8`                   | -128              | 127              | 8-bit signed integer
`uint8`                  | 0                 | 255              | 8-bit unsigned integer
`int16`, `short`         | -32768            | 32767            | 16-bit signed integer
`uint16`, `ushort`       | 0                 | 65535            | 16-bit unsigned integer
`int32`                  | -2147483648       | 2147483647       | 32-bit signed integer
`uint32`                 | 0                 | 4294967295       | 32-bit unsigned integer
`int`, `integer`         | -9007199254740991 | 9007199254740991 | 53-bit signed integer, matches `Number.isSafeInteger()` behavior
`uint`                   | 0                 | 9007199254740991 | 53-bit unsigned integer, matches `Number.isSafeInteger()` behavior
`lat`, `latitude`        | -90               | 90               | Latitude (double precision)
`lon`, `longitude`       | -180              | 180              | Longitude (double precision)

Number type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$allowed`               | `number[]` | `null`  | Array of numbers that are allowed. If this directive is used it cancels all directives that specify minimum, maximum, or any other number related constraints
`$min`                   | `number`   | `null`  | Minimum value (the number has to be greater or equal than `$min`)
`$max`                   | `number`   | `null`  | Maximum value (the number has to be lesser or equal than `$max`)
`$minExclusive`          | `number`   | `null`  | Minimum value is exclusive
`$maxExclusive`          | `number`   | `null`  | Maximum value is exclusive
`$divisibleBy`           | `number`   | `null`  | The number has to be divisible by this value (without a remainder)


Character
---------

Character `$type` is specified as `char` and it's a string that has length equal to one (that is, one character long string).

Character type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the char can be an empty string
`$allowed`               | `char[]`   | `null`  | Array of characters that are allowed


String and Text
---------------

String `$type` is specified as `string`, `text`, or `textline`. If type `string` is specified any JavaScript string passes, however, if `text` type is specified the validator only passes if the string doesn't contain `\u0000-\u0008`, `\u000B-\u000C`, and `\u000E-\u001F` characters. Use `text` to disallow these characters that have special meaning and are in most cases unwanted (especially the `\u0000` character). The `textline` type restricts text from using line and paragraph delimiter characters.

String/Text type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `true`  | Specifies if the string can be an empty
`$allowed`               | `string[]` | `null`  | Array of strings that are allowed. If this directive is used it cancels all directives related to string length validation, except `$empty` directive, which always applies, regardless of other constraints
`$length`                | `number`   | `null`  | Exact string length
`$minLength`             | `number`   | `null`  | Minimum string length
`$maxLength`             | `number`   | `null`  | Maximum string length
`$re`                    | `RegExp`   | `null`  | Regular expression


BigInt
------

BigInt `$type` is specified as `bigint`. BigInt is a string that contains only ASCII digits (characters from `0` to `9`) and an optional minus sign at the beginning. It allows to validate whether the number represented as a string doesn't overflow 64 bits and also allows to set a possible minimum and maximum value (also as string). BigInt can also be configured to allow more than 64-bits by using `$min` and `$max` directives, described below.

BigInt type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$allowed`               | `string[]` | `null`  | Array of strings that are allowed. If this directive is used it cancels `$min` and `$max` constraints
`$min`                   | `string`   | `null`  | Minimum value (as string)
`$max`                   | `string`   | `null`  | Maximum value (as string)

Optionally, you can use `xschema.misc.isBigInt(s, min, max)` function to check whether a string value matches BigInt with optional `min` and `max` constraints. This function doesn't require a schema instance.


Color
-----

Color `$type` is specified as `color`. Color is a string that matches `#RGB`, `#RRGGBB` or `color-name` format. It supports all color names that are defined by CSS specification and allows to include a dictionary having extra color names that you need to allow. Color names are case-insensitive by default.

Color type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$cssNames`              | `bool`     | `true`  | Specifies if CSS color names are allowed
`$extraNames`            | `set`      | `null`  | A set (dictionary having `key: true`) that contains extra color names that are allowed


Credit Card
-----------

Credit card `$type` is specified as `creditcard`. It checks whether the string is a valid credit card number by using a LUHN algorithm. The validator doesn't accept dashes or any other characters used as separators.

Credit card type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty


ISBN
----

ISBN `$type` is specified as `isbn`. It checks whether the string is a valid ISBN number.

ISBN type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$format`                | `string`   | `""`    | Specifies the ISBN format to accept. The default value `null` (or alternatively `""`) is used to accept any valid ISBN number. To restrict to a particular format use `"10"` or `"13"`.


MAC Address
-----------

MAC address `$type` is specified as `mac`. MAC address is a string in form `XX:XX:XX:XX:XX:XX` that specifies a network MAC address.

MAC address type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$separator`             | `char`     | `:`     | Specifies separator used between MAC address components


IP Address
----------

IP address `$type` is specified as `ip`, `ipv4`, or `ipv6`. IP address is a string specifying a network IP address. The `ip` type matches both IPV4 and IPV6 addresses, in contrast to `ipv4` and `ipv6` types, that match IPV4 and IPV6 addresses, respectively.

IP address type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$port`                  | `bool`     | `false` | Specifies if the IP address can contain a port number


UUID
----

UUID `$type` is specified as `uuid`. UUID validator is used to check whether the string contains a valid UUID number, that can be optionally surrounded by curly braces.

UUID type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$format`                | `string`   | `"rfc"` | Specifies the UUID format. It can be `"rfc"` to accept UUIDs in a `"XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"` format, `"windows"` to accept UUIDs surrounded by curly brackets, or `"any"` to accept either RFC or WINDOWS format. If not specified or `null` `"rfc"` format is used.
`$version`               | `string`   | `null`  | Specifies the version of UUID to accept. Version is a string from `"1"` to `"5"`. It can contain an optional `"+"` sign (like `"3+"`) to accept UUIDs of a particular version and all newer.


DateTime
--------

DateTime `$type` is specified as `date`, `datetime`, `datetime-ms`, and `datetime-us`. It's a formatted string that contains date, time, or date+time components. The validator extracts these components and validates whether they are correct. Leap years and leap seconds support is built-in and can be configured through directives.

DateTime defaults:

Date Type                | Format                       | Description
:----------------------- | :--------------------------- | :---------------------
`date`                   | `YYYY-MM-DD`                 | Date only
`datetime`               | `YYYY-MM-DD HH:mm:ss`        | Date+time
`datetime-ms`            | `YYYY-MM-DD HH:mm:ss.SSS`    | Date+time+ms
`datetime-us`            | `YYYY-MM-DD HH:mm:ss.SSSSSS` | Date+time+μs

DateTime format options:

Format Option            |Fixed Length| Range           | Description
:----------------------- | :----------| :-------------- | :---------------------
`Y`                      | `false`    | `1-9999`        | Year (1-4 digits)
`YY`                     | `true`     | `00-99`         | Year (2 digits)
`YYYY`                   | `true`     | `0001-9999`     | Year (4 digits)
`M`                      | `false`    | `1-12`          | Month (1-2 digits)
`MM`                     | `true`     | `01-12`         | Month (2 digits)
`D`                      | `false`    | `1-31`          | Day (1-2 digits)
`DD`                     | `true`     | `01-31`         | Day (2 digits)
`H`                      | `false`    | `0-23`          | Hour (1-2 digits)
`HH`                     | `true`     | `00-23`         | Hour (2 digits)
`m`                      | `false`    | `0-59`          | Minute (1-2 digits)
`mm`                     | `true`     | `00-59`         | Minute (2 digits)
`s`                      | `false`    | `0-60`          | Second (1-2 digits)
`ss`                     | `true`     | `00-60`         | Second (2 digits)
`SSS`                    | `true`     | `000-999`       | Millisecond (3 digits)
`SSSSSS`                 | `true`     | `000000-999999` | Microsecond (6 digits)
`?`                      | `true`     |                 | Any other character requires exact match of that character, for example `-`, `/`, `.`, `,`, etc...

DateTime type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$empty`                 | `bool`     | `false` | Specifies if the string can be an empty
`$format`                | `string`   | `null`  | Specifies date+time format, see format options above
`$leapYear`              | `bool`     | `true`  | Specifies whether to allow leap year date
`$leapSecond`            | `bool`     | `false` | Specifies whether to allow leap second date+time


Map
---


Map `$type` is specified as `map`. It's an object where all keys are strings and all values have the same type specified by `$data` directive.

Map type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the map can be `null`
`$data`                  | `object`   | `null`  | Specifies the schema of all map values.

Object
------

Object `$type` is specified as `object` or can be omitted completely. Object is a special type that allows to specify its members by using unprefixed keys (keys that don't start with `"$"`). For example the following specifies an object that has mandatory members (keys) `"a"` and `"b"`:

```js
var Schema = xschema({
  a: { $type: "int" },
  b: { $type: "int", $optional: true }
});

// These will pass.
xschema.test({ a: 1       }, Schema);
xschema.test({ a: 1, b: 2 }, Schema);
```

Object's directives and members can be mixed in the same definition, for example the following schema defines an object that has a nested object, which is optional:

```js
var Schema = xschema({
  nested: {
    $optional: true,

    a: { $type: "int" },
    b: { $type: "int", $optional: true }
  }
});

// These will pass.
xschema.test({                        }, Schema);
xschema.test({ nested: { a: 1       } }, Schema);
xschema.test({ nested: { a: 1, b: 2 } }, Schema);
```

TODO

Object type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`


Array
-----

Array `$type` is specified as `array` or by `[]` suffix (shortcut).

Array type directives:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the array can be `null`
`$length`                | `int`      | `null`  | Specifies an exact length of the array.
`$minLength`             | `int`      | `null`  | Specifies the minimum length of the array.
`$maxLength`             | `int`      | `null`  | Specifies the maximum length of the array.
`$data`                  | `object`   | `null`  | Specifies the schema of all array items.


Custom Types
------------

TODO


Global Directives
-----------------

Global directives can be applied to any `$type`:

Directive Name           | Value      | Default | Description
:----------------------- | :--------- | :------ | :-----------------------------
`$null`                  | `bool`     | `false` | Specifies if the value can be `null`
`$fn`                    | `function` | `null`  | A user-defined validation function. It should return `true` or `""` on success and false or `"ErrorCode"` on failure. The function is always called on a processed object, if posible. For example, if an object is transformed to object containing less members, the user-function will be called with that object, not the input one.
