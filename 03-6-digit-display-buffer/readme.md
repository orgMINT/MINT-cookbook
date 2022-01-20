# Recipe 3: 6 digit display buffer

## Project

Set up a region of memory to contain segment on/off information for the TEC-1's 6 7-segment displays. Write a routine to send this information to the TEC-1's displays by rapidly scanning the displays one at a time, activating each display for a short time before moving on to the next display. To create the illusion that all 6 displays are being lit simultaneously, this scan process needs to happen faster than the human eye can perceive.

## Hardware specifics

The TEC-1 controls its display using latches on two ports: the lower 6 bits (bits 0 - 5) of port 1 controls which digit of the 6 7-segment displays is illuminated. A 1 in bit 0 and all zeros in the other 5 illuminates the rightmost digit.

| Bit | digit         | selector |
| --- | ------------- | -------- |
| 0   | digit 0       | #01      |
| 1   | digit 1       | #02      |
| 2   | digit 2       | #04      |
| 3   | digit 3       | #08      |
| 4   | digit 4       | #10      |
| 5   | digit 5       | #20      |
| 6   | serial        | #40      |
| 7   | speaker / LED | #80      |

One complication is that the bit-banged serial used by MINT on some hardware configurations need bit 6 (#40) to be kept high during the scanning so that random noise isn't transmitted to the serial terminal.

The eight bits of port 2 control the segments which are illuminated according to the scheme below:

| Bit | segment | desc         |
| --- | ------- | ------------ |
| 0   | a       | top          |
| 1   | f       | top left     |
| 2   | g       | middle       |
| 3   | b       | top right    |
| 4   | dp      | point        |
| 5   | c       | bottom right |
| 6   | e       | bottom left  |
| 7   | d       | bottom       |

## Solution

### Display buffer

The first step is to declare a buffer of 6 bytes and initialise them all to zero (off). The easiest way to that is to do that is to declare a 6 item byte array full of zeroes.

```
\[0 0 0 0 0 0] ' b!
```

- `\[` indicates that the numbers following are byte values which will be stored in a byte array allocated on the heap.
- `0 0 0 0 0 0` is 6 literal zeros to initialise the array
- `]` indicates the end of the array. This pushes the address of the array on to the stack followed by its length in bytes.
- `'` we don't actually need this length so we can drop it.
- `b!` we store the address of the array in the variable `b` so we can access it later.

### Command A: output segments to a digit

Write a command which takes the segment data for a digit and a byte which selects the digit.
Update the hardware display ports with this data.

```
\\ selector segments --

:A 2\> #40| 1\> 10() #40 1\> ;
```

Where:

- `:A` declare a command called `A`
- `2\>` output `segments` data to port 2
- `#40|` ensure that bit 6 of the `selector` byte is high
- `1\>` output selector info to port 1
- `10()` delay for about half a millisecond
- `#40 1\>` turn off all segments, leave bit 6 high
- `;` end of command

### Command B: scan digits to display

Write a command which scans the buffer to the display

```
:B #20 b@ 6( %% \@ A 1+ $}$ ) '' ;
```

Where:

- `:B` declare a command called `B`
- `#20` initial selector value, the hex value #20 selects the leftmost digit
- `b@` get the address of the start of the display buffer
- `6(` loop 6 times, once for each of the 6 digits
- `%%` duplicate the top two items `selector` and `address`
- `\@` read segments data from `address`
- `A` call A with `selector` and segments data
- `1+` increase `address` to point to the next digit in buffer
- `$}$` swap top two items so that `selector` is on top, shift right 1 bit and swap back
- `)` end of loop
- `''` drop the top two items
- `;` end of command

## Exercise 1: fill buffer with all "8."'s and display for 10 seconds

```
:C b@ 6( #FF % \! 1+ ) ' 1000(B) ;
```

Where:

- `:C` declare a command called `C`

- `b@` get the address of the start of the display buffer

- `6(` loop 6 times, once for each of the 6 digits

- `#FF` segment data to display `8.`

- `%` copy address to top of stack. Top two items are `segments` and `address`

- `\!` write `segments` data to `address`, consumes both

- `1+` increase `address` to point to the next digit in buffer

- `)` end of loop

- `'` drop the top item

- `1000(B)` scan the display for about 10 seconds (on a 4MHz Z80)

- `;` end of command

  ### Code for Exercise 1

  ```
  \[0 0 0 0 0 0] ' b!
  :A 2\> #40| 1\> 10() #40 1\> ;
  :B #20 b@ 6( %% \@ A 1+ $}$ ) '' ;
  :C b@ 6( #FF % \! 1+ ) ' 1000(B) ;
  ```

  Run code with command C

## Exercise 2: display the numbers 0 to 5 for 10 seconds

To write out the numbers we need to look up their segment data in a table.

### Command E: convert a number ranging from 0 to F into segments

```
\\ number -- segments

\[#EB #28 #CD #AD #2E #A7 #E7 #29 #EF #2F #6F #E6 #C3 #EC #C7 #47] ' c!
:E c@ + \@ ;
```

- `\[` indicates that the numbers following are byte values which will be stored in a byte array allocated on the heap.
- `#EB #28 #CD`... hexadecimal segment data `#EB` is the segment data for `0`.
- `]` indicates the end of the array. This pushes the address of the array on the stack followed by its length.
- `'` we don't need the length so we drop it.
- `c!` we store the address of the array in the variable `c` so we can access it later.
- `:E` declare a command called `E`
- `c@` get the address of the start of the segments table
- `+` add `number` to address of segments table
- `\@` get segments data corresponding to number
- `;` end of command

### Main program

```
:F b@ 6( \i@E % \! 1+ ) ' 1000(B) ;
```

- `:F` declare a command called `F`

- `b@` get the address of the start of the display buffer

- `6(` loop 6 times, once for each of the 6 digits

- `\i@E` get loop counter variable and get segment data

- `%` copy buffer address to top of stack. Top two items are `segments` and `address`

- `\!` write segments data to buffer address, consume both

- `1+` increment address

- `)` end of loop

- `'` drop the top item

- `1000(B)` scan the display for about 10 seconds (on a 4MHz Z80)

- `;` end of command

  ### Code for Exercise 2

  ```

  \[0 0 0 0 0 0] ' b!
  :A 2\> #40| 1\> 10() #40 1\> ;
  :B #20 b@ 6( %% \@ A 1+ $}$ ) '' ;
  \[#EB #28 #CD #AD #2E #A7 #E7 #29 #EF #2F #6F #E6 #C3 #EC #C7 #47] ' c!
  :E c@ + \@ ;
  :F b@ 6( \i@E % \! 1+ ) ' 1000(B) ;

  ```

  Run code with command F

## Exercise 3: count up from 0 in hex incrementing once a second

### Command G: convert the lower 4 bits of a number into segments data and store them at address

```
\\ number address --

:G $ #0F& E $ \! ;
```

- `:G` declare a command called `G`
- `$` swap so that `number` is on top
- `#0F&` mask bottom 4 bits of `number`
- `E` get segments for nibble
- `$` swap so that `address` is on top
- `\!` write segments data to buffer address
- `;` end of command

### Command H: convert a number into segments data and store them in buffer

```
\\ number --

:H b@ 3+ 4( %% G 1- $ }}}} $ ) '' ;
```

- `:H` declare a command called `H`
- `b@` get the address of the start of the display buffer
- `3+` get address of 3rd digit (we will write segment data for digits 3,4,5 and 6)
- `4(` loop 4 times, once for each of the 4 digits
- `%%` duplicate the top two items of stack: `number` and `address`
- `G` convert the lower 4 bits of `number` into segments and store at `address`, consume both
- `1-` decrement address
- `$` swap so that `number` is on top
- `}}}}` shift `number` right by 4 bits
- `$` swap so that `address` is on top
- `)` end of loop
- `''` drop the top two items
- `;` end of command

### Main program

```
:I #FFFF( \i@ H 100(B) ) ;
```

- `:I` declare a command called `I`
- `#FFFF(` count up from 0 to #FFFF
- `\i@` get loop counter variable
- `H` convert to segments in buffer
- `100(B)` scan the display for about 1 second (on a 4MHz Z80)
- `)` end of loop
- `;` end of command

### Code for Exercise 3

```
\[0 0 0 0 0 0] ' b!
:A 2\> #40| 1\> 10() #40 1\> ;
:B #20 b@ 6( %% \@ A 1+ $}$ ) '' ;
\[#EB #28 #CD #AD #2E #A7 #E7 #29 #EF #2F #6F #E6 #C3 #EC #C7 #47] ' c!
:E c@ + \@ ;
:G $ #0F& E $ \! ;
:H b@ 3+ 4( %% G 1- $ }}}} $ ) '' ;
:I #FFFF( \i@ H 100(B) ) ;

```

Run code with command I
