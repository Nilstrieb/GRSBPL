# GRSBPL - Generic Random Stack Based Programming Language


A language that works on the stack (and has lots of other cool non-stack-based features)
```
1 5 * 5 +
> 10
```

There is a stack and variables. Operations are done on the stack, and you can store results in variables (a bit like in
the JVM). The stack contains integer values. Floating point numbers are not supported.

When the program finishes (run to the end of the program), the last value on the stack is returned. If the stack is
clear, 0 is always returned. If there is an error during execution, -1 is returned along with an error message to
stderr.

## Operators and keywords:

### Values

* any number `<number>` -> push the numeric value of n
* any character `'<character>'` -> push c as its escaped ascii value
* a string `"string"` -> only valid in combination with an `out` afterwards
* `&<ident>` -> pop and store it in a variable
* `@<ident>` -> load variable and push it, does not consume the variable

### Binary Operators

* `+` -> add two values on the stack, pops both and pushes the result
* `-` -> subtract two values on the stack, pops both and pushes the result
* `*` -> multiply two values on the stack, pops both and pushes the result
* `/` -> divide, pops both and pushes the result
* `%` -> mod, pops both and pushes the result
* `bnot` bitwise not on stack value
* `and` bitwise and
* `or` bitwise or
* `xor` bitwise xor

### Other operators

* `not` -> invert stack value (!=0 -> 0, 0 -> 1)
* `dup` -> duplicate the value on the stack
* `swap` -> swaps the 2 top stack values
* `pop`-> pop a value and discard it

### IO

* `out` -> pop and output it to the console as ascii
* `nout` -> pop and output as a number to the console
* `in` -> push input char as ascii to the stack
* `"<text>" out` -> prints the string

### Control flow

* `:<ident>` -> define a label
* `goto <ident>` -> goto a label if the value on the stack is !=0, peek
* `function <ident> <digit>` -> define a function with the arg count, always return a single number
* `<ident>` -> call a function, args have to be in the stack
* `return` -> return from a function

### Other

* `# comment #` text between # is ignored
* `# comment\n` text after # is ignored

`<ident>`: \w+, not a keyword  
`<number>`: decimal | hexadecimal with 0x prefix integer | binary with 0b prefix | octal with o prefix, underscores are
allowed anywhere in number 
`<digit>`: \d  
`<character>`: single character

Character escape sequences:  
\n, \r, \\, \0, \', \b, \f  
Same meaning as in Java

## Functions

When a function is called, a new stack frame for that function is created. The amount of args that function expects is
then popped of the stack and pushed onto that new stack frame, keeping the same order. After a function is finished, it
can return, deleting the stack frame and all local variables, and returning the top value on the stack. This value is
then pushed onto the parent stack. Execution now continues.

Since there are no blocks, a function missing a return keyword will simply continue to execute until it reaches the end
or jumps due to gotos.

Function 'bodies' (there are no bodies) are not ignored by the main flow, and have to be explicitly skipped by gotos.
It's recommended to place all functions at the end, and have one goto that ends the program by jumping to a label at the
end

## Examples:

FizzBuzz

```grsbpl
1 &i                # init loop counter
:start              # set start label
@i 100 - not goto exit              # if i is 100, exit
@i 15 % not goto print_fizz_buzz        # fizzbuzz
@i 5 % not goto print_buzz              # buzz
@i 3 % not goto print_fizz              # fizz
@i nout '\n' out                         # normal number
:end                # go back here after printing
@i 1 + &i           # increment i
1 goto start        # go back to the start

:print_fizz_buzz
   'F' out 'i' out 'z' out 'z' out 'B' out 'u' out 'z' out 'z' out '\n' out
   goto end
:print_fizz
   'F' out 'i' out 'z' out 'z' out '\n' out
   goto end
:print_buzz
   'B' out 'u' out 'z' out 'z' out '\n' out
   goto end

:exit 0
```

Add function

```grsbpl
1 2 add

function add 2
+ return
```

Recursive Factorial

```grsbpl
10 factorial 1 goto exit

function factorial 1
dup not goto isZero
&del dup 1 - factorial * return
:isZero
1 return

:exit swap
```

## Some Tips

* Increment a variable:
  `@i 1 + &i`
* Goto if equal
  `@i 100 - not goto finished`
* Goto not equal
  `@i 100 - goto finished`
* Exit the program
  `... goto exit ... :exit 0`
* Exit with exit code depending on the branch
  ```grsbpl
  ... 
  69 swap goto exit # push 69 to the 2nd stack position
  ... 
  5 swap goto exit # push 5 to the 2nd stack position
  ... 
  :exit &del # pop the top stack value to expose the pushed value
  ```
