# CindyScript language specification

This document describes the CindyScript language at a low level.

## Lexical structure

The lexical structure describes how the input string is turned
into a sequence of individual units, called tokens, and similar to words.

### Comments

A comment is introduced by `//` and spans to the end of the line,
i.e. the next newline character or the end of the input text.

    > 6 * 7 // this is a comment so it will be ignored
    < 42

There are no block comments in CindyScript.
However, one can often achieve a similar effect by turning the text in question
into a multi-line string literal,
as long as it does not contain any double quotation marks of its own.

    > "This is a multi-line string literal,
    >  pointing out that the value computed here
    >  has some connection to the
    >  Hitchhiker's Guide to the Galaxy";
    > 6 * 7
    < 42

### Whitespace

CindyScript considers the following Unicode characters to be whitespace:

* U+0020 (space)
* U+0009 (horizontal tab)
* U+000A (line feed)
* U+000D (carriage return)

Whitespace is conceptually removed before tokenization,
so identifiers may contain space characters and even line breaks.

    > arc sin ( 1 )
    < 90°
    > abc = 1 2 3  .  45; a b c
    < 123.45
    > a
    > 	b
    > 		c
    < 123.45
    > re ver se ([1,2,3])
    < [3, 2, 1]

Using characters other than a single space within a token is discouraged,
since it is highly confusing and may be unsupported in a future version.

### String literals

A string literal starts with a double quotation mark,
and ends at the next quotation mark.
There are no escape sequences;
all the text between the quotation marks is included literally.
Use the [`unicode`](Texts_and_Tables.md#unicode$1) function
to include literal double quotes and other special characters.

    > " Text with
    > newline, // comment and
    > some	tab character "
    < " Text with\nnewline, // comment and\nsome\ttab character "
    > "She said " + unicode("22") + "Hello, world!" + unicode("22")
    < "She said \"Hello, world!\""

Note that string results are printed in JSON format in this documentation,
but that representation would actually not be suitable as input.

### Numeric literals

A numeric literal is any non-empty sequence of any number of ASCII digits
and at least one dot (U+002E).

In particular, this includes a sole dot as a valid representation
for the number zero.
This is a rather obscure quirk, and portable code should not rely
on this behavior in future releases.

    > [1.23, 2., .34, ., 5]
    < [1.23, 2, 0.34, 0, 5]

There are no negative literals, but unary sign operators may be combined with
numeric literals to build negative numbers.
There also are no literals for complex numbers,
but the variable `i` is pre-set to the complex unit.

    > [- 2.3, 4.5 + 6.7 * i]
    < [-2.3, 4.5 + i*6.7]

There is no implicit way to write literals in scientific notation.
What looks like a numeric literal is in fact considered a variable name.
One can use the [power operator `^`](Arithmetic_Operators.md#$5eu)
to write in scientific notation, though.

    > 12e3
    * Warning: Accessing undefined variable: 12e3
    < ___
    > 12e3 = 17; 12e3
    < 17
    > 12 * 10^3
    < 12000

There are no literals representing infinite values or NaN (Not a Number) values.

Numeric literals may contain an arbitrary number of digits,
although adding more digits doesn't add precision once the internal
floating-point precision is exhausted.

    > 3.141592653589793234567890123456789012345 == pi // last digits are WRONG!
    < true

### Identifier names

Identifiers may contain one or more of the following symbols:
* ASCII letters `a`-`z` and `A`-`Z`
* ASCII digits `0`-`9`
* The ASCII apostrophe U+0027 `'`
* Codepoints from the [Unicode 6.2.0](http://www.unicode.org/Public/6.2.0/)
Basic Multilingual [Plane](http://www.unicode.org/glossary/#plane) (BMP) with 
[General category](http://www.unicode.org/reports/tr44/tr44-16.html#General_Category_Values) `L` (i.e. Letters).

In addition to these, [`#`](General_Concepts.md#local-variables-the-variable)
and `#1` through `#9` are also valid variable names.
Multiple digits are not permissible, though.

    > #9 = 12; #9
    < 12
    > #12 = 17; #12
    < ___
    > foo#1 = 19; foo#1
    < ___

Note that some letters may have right-to-left as the default text orientation.
Depending on the environment, this may lead to unexpected display order
for input or output.

    > ערשטער = 1;
    > רגע = 2;
    > דריט = 3;
    > [ערשטער, רגע, דריט]
    < [1, 2, 3]

Contrary to many other programming languages,
the [underscore `_` is an operator](Lists_and_Linear_Algebra.md#$5fu)
and may not be used as part of an identifier name.
Spaces may be used instead to separate words, as described [above](#whitespace).

These identifiers may be used both for
[variables](Variables_and_Functions.md#defining-variables) and for
[user-defined functions](Variables_and_Functions.md#defining-functions),
as well as for arguments to such functions and to name
[modifiers](General_Concepts.md#modifiers).

### Operators and Brackets

The following section will describe a number of operators and bracket symbols
which too form lexical tokens.
They won't overlap with the tokens described above with one exception:
the `.` may be used both for field access and for numeric literals.

Internally, the dot is always parsed as a field access operator at first,
but if its arguments are only digits,
then it is interpreted as a numeric literal instead.
So the precedence of the dot operator controls this distinction.
Since this precedence is pretty low, one can in general assume that
a given piece of code containing a dot is a numeric literal
unless there is an identifier before or after the dot.
One notable exception is the `:` operator which has even lower precedence.
It is not implemented in CindyJS yet, though.

    > x = [];
    > x:12.3 = 4.56;
    > x:"12.3"
    * Left hand side of assignment is not a recognized lvalue
    * Operator : is not supported yet.
    < ___

## Grammar

The grammar describes how lexical tokens are combined to form expressions.

### Expressions

An expression can be one of the following

1. a literal number or string
1. an identifier to be used as the name of a variable
1. a sequence of zero or more expressions,
   separated by commas and enclosed in brackets
1. an expression followed by a binary operator followed by an expression
1. an expression followed by a postfix operator
1. a prefix operator followed by an expression

### Operator precedence

CindyJS knows about the following operators, in order of precedence:

1. [`:`](Variables_and_Functions.md#user-defined-data)
1. [`.`](Accessing_Geometric_Elements.md#properties-of-geometric-objects),
   [`°`](Arithmetic_Operators.md#_$b0u)
1. [`_`](Lists_and_Linear_Algebra.md#$5fu),
   [`^`](Arithmetic_Operators.md#$5eu)
1. [`*`](Arithmetic_Operators.md#$2au),
   [`/`](Arithmetic_Operators.md#$2fu)
1. [`+`](Arithmetic_Operators.md#$2bu),
   [`-`](Arithmetic_Operators.md#$2du),
   [`!`](Boolean_Operators.md#$21u_)
1. [`==`](Boolean_Operators.md#$3du$3du),
   [`~=`](Boolean_Operators.md#$7eu$3du),
   [`~<`](Boolean_Operators.md#$7eu$3cu),
   [`~>`](Boolean_Operators.md#$7eu$3eu),
   `=:=` *(unofficial)*,
   [`>=`](Boolean_Operators.md#$3eu$3du),
   [`<=`](Boolean_Operators.md#$3cu$3du),
   [`~>=`](Boolean_Operators.md#$7eu$3eu$3du),
   [`~<=`](Boolean_Operators.md#$7eu$3cu$3du),
   [`>`](Boolean_Operators.md#$3eu),
   [`<`](Boolean_Operators.md#$3cu),
   [`<>`](Boolean_Operators.md#$21u$3du)
1. [`&`](Boolean_Operators.md#$26u),
   [`%`](Boolean_Operators.md#$25u),
   [`!=`](Boolean_Operators.md#$21u$3du),
   [`~!=`](Boolean_Operators.md#$7eu$21u$3du),
   [`..`](Elementary_List_Operations.md#$2eu$2eu)
1. [`++`](Elementary_List_Operations.md#$2bu$2bu),
   [`--`](Elementary_List_Operations.md#$2du$2du),
   [`~~`](Elementary_List_Operations.md#$7eu$7eu),
   [`:>`](Elementary_List_Operations.md#$3au$3eu),
   [`<:`](Elementary_List_Operations.md#$3cu$3au)
1. [`=`](Variables_and_Functions.md#defining-variables),
   [`:=`](Variables_and_Functions.md#defining-functions),
   [`::=`](Variables_and_Functions.md#binding-variables-to-functions),
   `:=_` *(unofficial)*,
   [`->`](General_Concepts.md#modifiers)
1. [`;`](General_Concepts.md#control-flow)

Operators of equal precedence are left-associative.

    > 3^2^4
    < 6561
    > 3^(2^4)
    < 43046721
    > x = y = 0
    * Can't use infix expression as lvalue
    < 0
    > x = (y = 1)
    < 1
    > x
    < 1

Sometimes it makes sense to view the comma `,` as another operator,
with even lower precedence than `;`.
But since `,` may only be used inside brackets, and not at the top level,
it is not included in the above list.

    > 1, 2, 3
    < ___

### Prefix and postfix operators

The `+` and `-` operators may be used both in infix and in prefix notation.

    > x = 17;
    > -x
    < -17
    > +x
    < 17
    > -[1, 2, 3]
    < [-1, -2, -3]
    > +[1, 2, 3]
    < [1, 2, 3]

The `!` operator may only be used in prefix notation.

    > !(7 == 7)
    < false

The `°` operator may only be used in postfix notation.

    > 90° + 0
    < 1.5708

All other operators are valid only in infix notation.

### Brackets

A bracket expression consists of an opening bracket,
followed by zero or more expressions, delimited by commas,
followed by a matching closing bracket.

It is incorrect to append a comma after the last element.

    > [1, 2, ]
    < [1, 2, ___]

CindyScript knows four types of brackets:

#### Round parentheses `(…)`

As `(‹expr›)` it can be used to control order of evaluation for an expression.
As `(‹expr1›, ‹expr2›, …)` it defines a vector with the given elements.
The form `()` can be used to denote the empty list.

    > 7 * (1 + 2)
    < 21
    > 7 * (1, 2)
    < [7, 14]
    > 7 * ()
    < []

#### Square brackets `[…]`

These always denote a list, even if they contain exactly one expression.

    > 7 * [1 + 2]
    < [21]
    > 7 * [1, 2]
    < [7, 14]
    > 7 * []
    < []

#### Curly braces `{…}`

The single-expression form `{‹expr›}` is just like `(‹expr›)`.
This form is deprecated, and likely will get removed in the near future.
Use `(…)` instead.
The use with zero elements, or more than one, is reserved for future extensions.

    > 7 * {1 + 2}
    < 21
    > 7 * {1, 2}
    < ___

#### Vertical bars `|…|`

With a single argument, [`|‹expr›|`](Arithmetic_Operators.md#$7cu_$7cu)
denotes the absolute value of a complex number,
or the norm of a vector.

    > |3 + 4*i|
    < 5
    > v = [2, 2, 3, 2, 2]; |v|
    < 5

With two arguments, [`|‹vec1›,‹vec2›|`](Arithmetic_Operators.md#$7cu_$2cu_$7cu)
computes the distance between these points or vectors.

    > x = [3, 7];
    > y = [7, 10];
    > |x, y|
    < 5

It is illegal to nest expressions using vertical bars.

    > |[3, |4*i|]|
    < ___
