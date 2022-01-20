# A MINT Tutorial

MINT is an _interactive_ programming language consisting of user defined subroutines (which are called commands in MINT), built-in operators, variables and literal data.

"Interactive" means you type things in at the keyboard and the machine responds. We will see some details of how it does this below.

Start the MINT system and you will see a command prompt `>` which you can use to enter MINT code to execute.

```
MINT v1.0

>
```

Now enter the following:

```
> 2 17  +  .

19
```

What happened? MINT is an interpreter. It reads in a sequence of characters which it then executes when the user types <ENTER>.

This sequence can be interpreted in only three ways: commands (subroutine names), numbers, or "not defined".

The interpreter looks at each character in turn and decides whether it is a user command (user commands are uppercase letters A..Z), a built-in command (any ASCII symbol or an uppercase letter or symbol that is preceded by a `\`), a variable reference (variables are lowercase letters a..z and system variables are \a..\z) or a literal number.

MINT is a stack-based language which means that all values are passed to and from commands via a data structure called the stack. If the character is a command it will be executed by using values that are already on the stack and it can return zero or more values back on the stack. If the character is a variable reference then the address of that variable will be returned on the stack. If the character is the start of a literal number, its characters will be read one by one and interpreted as digits. Once a non-numeric character is detected the value of the number is pushed onto the stack.

In the above example, MINT interpreted 2 and 17 as decimal numbers and
pushed them both onto the stack.

`+` is a built-in command meaning "add two numbers" and so is `.` which means "print the top item on the stack" so they were both executed. After execution 19 was left on the stack.

The command `.` removed 19 from the stack and printed it to the terminal.

We might also have said

```
#0A  #14  * ,

C8
```

`#` indicates that the following number will be in hexadecimal rather than decimal.
`*` is a built-in meaning multiply two numbers
`,` is similar to `.` but it prints the number on the stack in hexadecimal. Like `.` it removes the item it prints from the stack.

Finally, here is the obligatory "Hello, World!" program. MINT lets you output text using the command ` as follows:

`Hello, World!`

In MINT anything that appears between ` \`` and  `\``is printed to the terminal. You can also print newlines using`\N`or to emit any ASCII value to the terminal by using the`\E` command.

MINT is a byte code interpreted language. This that means it can interpret commands (subroutines or programs) typed in to the terminal, as well as define new user commands. To define a new user command we use two built-in ones `:` (meaning "start a new definition") and `;` (meaning "finish the definition").

Let's try this out:

```
:A * + ;
```

What happened? The action of the built-in command `:` is to start defining a new user command, it then reads the next character which must be an uppercase letter which will be the name of the command. The symbols which follow are not executed but are saved as part of the user command definition until a `;` is encountered.

Now try out A :

```
5 6 7 A .

47
```

This example illustrated two principles: adding a new command to
MINT and trying it out as soon as it was defined.

### Remarks on factoring

Factoring is the process of breaking out repeated pieces of code from subroutines and giving them a name. This not only shortens the overall program
but can make the code simpler.

For example, consider the following "sum of squares" routine. It takes two numbers, squares them and adds them together:

```
:S " * $ " * + ;
```

This code has the repeated phrase `" \*` and this can be replaced by a routine Q. Factoring like this can lead to shorter and more readable code and is an encouraged practice in MINT programming.

```
:Q " * ;

:S Q $ Q + ;
```

## Stacks and Reverse Polish Notation (RPN)

We now discuss the stack and the "reverse Polish" or "postfix" arithmetic based on it.

Virtually all modern CPU's are designed around stacks. MINT efficiently uses its CPU by reflecting this underlying stack architecture
in its syntax.

But what is a stack? As the name implies, a stack is the machine analog of a pile of cards with numbers written on them. Numbers are always added to the top of the pile, and removed from the top of the
pile. The MINT input line

2 73 5 16

leaves the stack in the state

| cell # | contents |       |
| ------ | -------- | ----- |
| 0      | 16       | (TOS) |
| 1      | 73       | (NOS) |
| 2      | 5        |       |
| 3      | 2        |       |

where `TOS` stands for "top-of-stack" and `NOS` stands for "next-on-stack".

We usually employ zero-based relative numbering in MINT data structures (such as stacks, arrays, tables, etc.) so TOS is given relative
#0, NOS #1, etc.

Suppose we followed the above input line with the line

```
2 73 5 16 + - * .
```

How did we get this result? The operations would produce the successive stacks

| cell # | initial | \+  | \-  | \*  | .   |
| ------ | ------- | --- | --- | --- | --- |
| 0      | 16      | 21  | 52  | 104 |
| 1      | 5       | 73  | 2   |
| 2      | 73      | 2   |
| 3      | 2       |     |

The `.` pops the final value 104 from the stack and prints it.

### Manipulating the parameter stack

A stack-based system must provide ways to put numbers on the stack, to remove them, and to rearrange their order. MINT includes standard commands for this purpose.

Putting numbers on the stack is easy: simply type the number (or include it in the definition of a MINT command).

- `'` is called the "drop" command and it removes the number from TOS and moves up all the other numbers.
- `"` is called "dup" and it duplicates the TOS into NOS.
- `$` is called "swap" and it exchanges the top 2 numbers.
- `\R` is called "rot" and it rotates the top 3 items on the stack.
- `%` is called "over" and it duplicates NOS and puts it above TOS

These actions are shown below (we show what each command does to the initial stack)

| cell | initial | '   | "   | $   | \R  | %   |
| ---- | ------- | --- | --- | --- | --- | --- |
| 0    | 16      | 5   | 16  | 5   | 73  | 5   |
| 1    | 5       | 73  | 16  | 16  | 16  | 16  |
| 2    | 73      | 2   | 5   | 73  | 5   | 5   |
| 3    | 2       |     | 73  | 2   | 2   | 73  |
| 4    |         |     | 2   |     |     | 2   |

### Variables

These stack commands are useful for manipulating the top 3 items on the stack but it is generally a good idea to avoid too much stack manipulation in MINT. Stack juggling can be tricky at times and can lead to less readable code so to keep things simple it is advisable for the programmer to make liberal use of the variables.

MINT has the user variables `a..z` and system variables `\a..\z`. The system variables used by MINT internally so some care needs to be taken with those. That said, many system variables are not used at all and are available for use by the programmer.

A variable reference is MINT is simply an address pushed onto the stack. For example a reference to variable `a` will push the address in memory of the variable `a`. A reference to variable `b` will push the address in memory of variable `b` etc.

So

```
a ,

0A80
```

The exact location of `a` in memory will vary from MINT system to MINT system.

will print in hexadecimal the address of variable `a`.

To store a value in variable `a` we need the data to store, the address and we need to use the "store" command `!`.

```
100 a !
```

This store the value 100 into the variable `a`. To retrieve the value stored inside variable `a` we need to use the "fetch" command `@`.

So

```
a @ .

100
```

`@` and `!` are commands to fetch and store 16-bit values. MINT also has commands for working with bytes. To store the byte value `#FF` in variable `z`

```
#FF z \!
```

To read the byte value stored in `z`

```
z \@ ,

FF
```

## Using memory

As we just saw, ordinary numbers are fetched from memory to the stack by fetching with `@` and stored with `!`.

`@` expects an address on the stack and replaces that address by its contents.

```
a @
```

`!` expects a number (NOS) and an address (TOS) to store it in, and places the number in the memory location referred to by the address, consuming both arguments in the process

```
3 a !
```

### Heap

In addition to the variables MINT can allocate items on the "heap". The top of the heap is pointer stored in the `\h` system variable. Allocations of any size can be made on the heap and the heap top is bumped to reflect this allocation.

For example, to allocate 10 bytes on the heap the heap we need to save the current state of `\h` in another variable. This will be the pointer to the start of our new allocation.

```
\h @ a !
```

Then we can bump up the value of `\h`.

```
10 \h@ + \h!
```

Now we can use the newly allocated 10 bytes pointed to by variable `a`. Let's store `#AA` in the 0th byte location.

```
#AA a@ !
```

We could also store #BB in the 1st byte location

```
#BB a@ 1 + \!
```

In this second example we push #BB on the stack for use later.

Then we push the address of `a` on the stack which we want to increase by 1 to point to the 1st byte location. We do that by pushing a 1 on the stack and then adding the top two stack items. Now we the address of `a+1` byte in TOS. The NOS is #BB.

Finally we call \! which does a byte store at address `a+1`

### Arrays

Directly manipulating the `\h` is easy and powerful but MINT also has a flexible notation for defining arrays of literal 16-bit and 8-bit data.

To define an array of bytes we could do the following

```
\[ 10 20 30 40 50 ]
```

This allocates 5 x 2 bytes of the heap and returns a pointer to the start of the array and the length of the array in bytes (in this case 5).

We could store the pointer to this array in variable `b` but we first need to discard the length value with a `'`.

```
\[ 10 20 30 40 50 ] ' b!
```

To access the 2nd location in this byte array

```
b@ 2 + \@ .

30
```

MINT also allows the declaration of 16-bit number arrays. MINT numbers are usually expressed as 16-bit and pointers are also 16-bit.

```
[ 100 200 400 800 ]
```

This will return a pointer to the array and its length in words (in this case 4).
Once again to store the pointer in a variable, say `c`, we first discard the length value with `'`.

```
[ 100 200 400 800 ] ' c!
```

To access a array we need to calculate the offset address as before however we need to make sure we deal with 16-bit quantities. The 3rd item in the array is _6 bytes_ from the beginning of the array.

```
c@ 3 2 * + @ .

800
```

Breaking this down: the address of the array which is stored in variable `c` is pushed on the stack. Then we push the index 3 on the stack (we want the 3rd item in the array). We multiply this number by 2 to get 6 to get its address in bytes.

We then add the 6 to the address of the array and fetch the 16-bit value stored there.

Note: a convenient way to multiply something by 2 is to shift its value left by one bit. MINT contains a convenient command to this `{`

```
c@ 3 { + @ .

800
```

You can use `}` to do the opposite, to divide by 2 or shift the value one bit top the right.

## Comparing and branching

MINT lets you compare two numbers on the stack, using relational
operators ">", "<", "=" . Thus, e.g., the phrase

```
2 3 > .

0
```

This prints out 0 or "false" because 2 (NOS) is not greater than 3
(TOS). Conversely, the phrase

```
2 3 < .

1
```

prints 1 or "true" because 2 really is less than 3.

A convenient way to convert a false value to a true one or _vice versa_ is to use the combination: `0=`. For example

```
2 3 < 0=

0
```

The relational commands are used for branching and conditional execution. For example,

```
:T 3 < (\N `Less that 3!`) ;
```

This is a user command which tests for less than 3. When we pass

```
5 T
```

Nothing happens. However if we pass

```
2 T
```

MINT with print to the terminal

```

Less than 3!
```

The TOS is compared with 3 and if the result is true, the code within the parentheses is executed. The command `\N` prints a newline followed by the string "Less than 3!". If the condition is false, the code in the parentheses is skipped.

So in summary, you can make code conditionally executed by putting a boolean value (0 = false, 1 = true) in front of some code in parentheses. For example:

```
:Z 0= (`This is zero`) ;
```

MINT also provides the ability to perform an IF...THEN...ELSE... constructions by using

```
:Y 100 > \( `greater than 100` )(` less than or equal to 100` ) ;
```

If the condition is true then the code between `\(` and `)` is executed else if the condition is false then the code between `(` and `)` is executed.

Another example:

```
:X \N 0= \(` false `) (`true`) ;

1 X

true

0 X

false
```

## Commenting MINT code

MINT comments are similar to comments in other programming languages such as C++ or JavaScript. When MINT encounters two backlash characters `\\` it assumes that the rest of the line is a comment.

```
1 2 + .     \\ this code adds the numbers 1 and 2 and prints the result
```

## Stack effect diagrams

MINT is an untyped language and MINT commands can take any number of arguments and (unlike many other languages) return any number of results. The mechanism it uses to do this is of course the stack.

> If a MINT command can be thought of as a function in the mathematical sense then then a command is a function of a stack of arguments and returns a stack of arguments.

To help document what a MINT command does, it is a common practice to create a _stack diagram_ for the command. Stack diagrams look like this:

```
( arg1 arg2 ... -- result1 result2 ... )
```

This scheme is completely informal in MINT and purely for informational purposes. You can put it inside a comment of your command's definition. For example:

```
:S " * " * + ;    \\ sum of squares ( n n -- n )
```

Built-in commands are also documented using stack-effect diagrams.

```
@ fetches the 16-bit bit value stored at adr ( adr -- n)
```

## Looping and structured programming

You've already encountered MINTs looping construct but may have thought it was only used for conditional code. In fact it turns out that conditional execution is just a special case of a looping construct.

To be clearer, a loop in MINT is anything between parentheses. The number of times through the loop is the value on the top-of-stack at the time the opening parenthesis is encountered.

To loop 5 times all that is needed is

```
5(`hello`)

hello
hello
hello
hello
hello
```

If the value on the stack is 0 then the code inside the parentheses is skipped entirely. If the value is 1 then the code is executed exactly once. Astute readers will recognize that is exactly what is need for executing on a condition because in MINT, false is represented by a 0 while true is represented by a 1.

## Loop variables

Every MINT loop comes with a counter variable `\i` which tells you how many times you've been through the loop. The variable starts at 0 and goes up to just before the specified limit. For example if the limit of the loop is 5 then we can print out the values 0 to 4 by

```
5( \i@ . )

0 1 2 3 4
```

\i is a system variable which takes its value from the current loop. This is different from other variables in MINT which are global in scope. You still use them like other variables however so you need to use `@` to access them and you can even write to them with `!`.

## Indefinite loops

Loops in MINT usually have a maximum number of iterations. This is normal and encouraged in MINT but if you really need them, you can make a loop indefinite by manipulating the counter variable `\i` by setting it to 0 For example

```
2( 0\i! `x` )

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx ...etc
```

On each iteration we assign zero to the counter variable `\i`. This prevents it from counting up to its limit.
NOTE: The limit value (in this case 2) has to be higher than 1 for this to work.

# Breaking loops

All loops can be broken out of by the `\B` command. `\B` takes a conditional argument which must be true before this command exits its loop.

```
10( \i@ 4 > \B \i@ . )

0 1 2 3 4
```

# Nesting loops

MINT allows loops to be nested but `\i` only refers to the inner most loop. You can access the counter variable of the parent loop by using `\j`. Deep nesting is not encouraged in MINT but it is possible to access the counter variable of a parent loop by adding 6 to the address of the current loop's counter variable.

```
3(`outer i: ` \i@. \N 3( `    inner i: `\i@. `j: ` \j@. \N) \N)

outer i:00000
    inner i: 00000 j: 00000
    inner i: 00001 j: 00000
    inner i: 00002 j: 00000

outer i:00001
    inner i: 00000 j: 00001
    inner i: 00001 j: 00001
    inner i: 00002 j: 00001

outer i:00002
    inner i: 00000 j: 00002
    inner i: 00001 j: 00002
    inner i: 00002 j: 00002
```

# MINT programming style

MINT is a "concatenative" programming language which that programming is achieved by concatenating pieces of code together. It shares this characteristic with similar languages such as Forth. Concatenating means joining together chains of commands to form longer chains of commands.
Commands are operations in MINT which optionally take the results of previous commands and optionally return one of more results of their own. Results are stored on a common data structure called the "stack". Sometimes this is called the "data stack" or the "parameter stack". Even literal numbers which appear in a MINT program can be considered a command (e.g. the literal 3 is a command to push the value of 3 on to the stack)

Programs grow through concatenation but the programmer can control the length and improve the clarity of a MINT program by "factoring" the repeated parts of a chain into its own user command. The body of a user command is like the body of a subroutine in another language. In MINT these bodies are often called "definitions".

Because the state of the stacks is not explicitly expressed in MINT it can sometimes be confusing to the casual reader. So to help manage MINT code and to maximize clarity here are a few recommendations:

- break your code down into short definitions < 40 chars in length
- each definition should only do one simple thing
- don't pass too many arguments in a definition, just 1 or 2, maximum 3.
- avoid complexity such as deep nesting of loops and control structures
- avoid excessive stack manipulation. Use variables
- use spaces liberally

## Glossary

### Maths Operators

| Symbol | Description                               | Effect   |
| ------ | ----------------------------------------- | -------- |
| -      | 16-bit integer subtraction SUB            | a b -- c |
| {      | shift the number to the left (2\*)        | a -- b   |
| }      | shift the number to the right (2/)        | a -- b   |
| /      | 16-bit by 8-bit division DIV              | a b -- c |
| \_     | 16-bit negation (2's complement) NEG      | a -- b   |
| \*     | 8-bit by 8-bit integer multiplication MUL | a b -- c |
| \>     | 16-bit comparison GT                      | a b -- c |
| +      | 16-bit integer addition ADD               | a b -- c |
| <      | 16-bit comparison LT                      | a b -- c |
| =      | 16 bit comparison EQ                      | a b -- c |

### Logical Operators

| Symbol | Description        | Effect   |
| ------ | ------------------ | -------- |
| \|     | 16-bit bitwise OR  | a b -- c |
| &      | 16-bit bitwise AND | a b -- c |
| ^      | 16-bit bitwise XOR | a b -- c |

Note: logical NOT can be achieved with 0=

### Stack Operations

| Symbol | Description                                                          | Effect         |
| ------ | -------------------------------------------------------------------- | -------------- |
| '      | drop the top member of the stack DROP                                | a a -- a       |
| "      | duplicate the top member of the stack DUP                            | a -- a a       |
| ~      | rotate the top 2 members of the stack ROT                            | a b c -- b c a |
| %      | over - take the 2nd member of the stack and copy to top of the stack | a b -- a b a   |
| $      | swap the top 2 members of the stack SWAP                             | a b -- b a     |

### Input & Output Operations

| Symbol | Description                                               | Effect      |
| ------ | --------------------------------------------------------- | ----------- |
| ?      | read a char from input                                    | -- val      |
| .      | print the top member of the stack as a decimal number DOT | a --        |
| ,      | print the number on the stack as a hexadecimal            | a --        |
| \`     | print the literal string between \` and \`                | --          |
| \\.    | print a null terminated string                            | adr --      |
| \\,    | prints a character to output                              | val --      |
| \\$    | prints a CRLF to output                                   | --          |
| \\>    | output to an I/O port                                     | val port -- |
| \\<    | input from a I/O port                                     | port -- val |
| #      | the following number is in hexadecimal                    | a --        |

### User Definitions

| Symbol        | Description                      | Effect |
| ------------- | -------------------------------- | ------ |
| ;             | end of user definition END       |        |
| :<CHAR>       | define a new command DEF         |        |
| \\:           | define an anonynous command DEF  |        |
| \\?<CHAR>     | get the address of the def       | -- adr |
| \\{<NUM>      | enter namespace NUM              | --     |
| \\}           | exit namespace                   | --     |
| \\<NUM><CHAR> | execute a command in a namespace | --     |

NOTE:
<CHAR> is an uppercase letter immediately following operation which is the name of the definition
<NUM> is the namespace number. There are currently 5 namespaces numbered 0 - 4

### Loops and conditional execution

| Symbol | Description                                       | Effect |
| ------ | ------------------------------------------------- | ------ |
| (      | BEGIN a loop or conditionally executed code block | n --   |
| )      | END a loop or conditionally executed code block   | --     |
| \\(    | beginIFTE \\(`true`)(`false`)                     | b --   |
| \\\_   | if true break out of loop                         | b --   |

### Memory and Variable Operations

| Symbol | Description                   | Effect        |
| ------ | ----------------------------- | ------------- |
| !      | STORE a value to memory       | val adr --    |
| [      | begin an array definition     | --            |
| ]      | end an array definition       | -- adr nwords |
| @      | FETCH a value from memory     | -- val        |
| \\!    | STORE a byte to memory        | val adr --    |
| \\[    | begin a byte array definition | --            |
| \\`    | begin a string definition     | -- adr        |
| \\@    | FETCH a byte from memory      | -- val        |

### System Variables

| Symbol | Description                        | Effect |
| ------ | ---------------------------------- | ------ |
| \\a    | data stack start variable          | -- adr |
| \\b    | base16 flag variable               | -- adr |
| \\c    | text input buffer pointer variable | -- adr |
| \\d    | start of user definitions          | -- adr |
| \\h    | heap pointer variable              | -- adr |
| \\i    | loop counter variable              | -- adr |
| \\j    | outer loop counter variable        | -- adr |

### Miscellaneous

| Symbol | Description                                   | Effect   |
| ------ | --------------------------------------------- | -------- |
| \\\\   | comment text, skips reading until end of line | --       |
| \\^    | execute mint code at address                  | adr -- ? |

### Utility commands

| Symbol | Description                     | Effect   |
| ------ | ------------------------------- | -------- |
| \\#0   | execute machine code at address | adr -- ? |
| \\#1   | push to return stack            | val --   |
| \\#2   | pop from return stack           | -- val   |
| \\#3   | stack depth                     | -- val   |
| \\#4   | print stack                     | --       |
| \\#5   | print prompt                    | --       |
| \\#6   | edit command                    | val --   |

### Control keys

| Symbol | Description                     |
| ------ | ------------------------------- |
| ^B     | toggle base decimal/hexadecimal |
| ^E     | edit a definition               |
| ^H     | backspace                       |
| ^J     | re-edit                         |
| ^L     | list definitions                |
| ^P     | print stack                     |
