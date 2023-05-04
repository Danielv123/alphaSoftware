# ShiftySnake

My previous attempt at making snake for the alpha was very much rooted in attempting to do things the way I would have done them in C by almost line for line replicating an arduino program. This however meant I had to deal with the memory limitations of the alpha, especially the severly limited display.

The main issue of the display is the 200 block limit. There are 2 ways to write to the display using the display block:

1. Write a fixed string, up to 4x12 characters long
2. Write a number, up to 4 digits long

Becasue the string is fixed its not very useful for making games, so we will ignore that. 4 digits however means we only need 16 display blocks for an 8x8 display, same as what I used for the 4x4 display in arduSnake.

This gives us 4 realistic options for display:

* Integrated 4x12 display driven per pixel by 12 display blocks
* Integrated 4x10 display driven per pixel by 8 display blocks
* External display with up to 32 pixels per row
* Integrated 5x8 display using stacked calculator monitoring output in a base10 graphcs memory
  - No way to do leading zeroes. Add fixed offset of 10000 maybe? Could use 2s or any other number to represent the snake as well

The only major difference between them code wise is how the pixels are updated. When using the integrated display we would use powers of 10 while with an external display we would use powers of 2 in our GPU.

The game itself will still operate on a simplified representation - directly updating and maintaining state of the game in the display buffers is very complicated and error prone due to its scanning nature.

## Game logic

The game has 2 shifting array buffers built using calculator blocks, one for the X coordinate and one for the Y coordinate. Whether each block is rendered depends on an external length counter. Collisions is checked by comparing the head of the snake to the rest of the snake while scanning through for rendering.

## GPU logic

The GPU consists of 8 calculator memory cells. It is updated one cell at a time.

### Memory cell update

1. Clear the memory cell to 0
2. Loop through the snake array for each row we want to draw. `x = a | b`
3. Use a shortcut. Since we know that we will never get a double write to the same memory cell, we can add 2^x to the memory cell instead of doing a bitwise OR. This is because `x = a + 2^b` is the same as `x = a | (1 << b)`, which is the same as setting the xth bit to 1, as long as we know that the bit is not already set. We can use the same shortcut with an internal base 10 display as well.

The difficult part of this is the bitwise OR. Alpha does not have an operator for this, so we have to roll out own. There are 2 ways:

1. Loop through bits dividing by 2 each time, comparing the remainder and adding the (result of the comparison)^(loop counter) to the result
2. Use static calculator blocks to extract each bit and add them together

### Code

We operate on 1 set of X,Y coordinates at a time. Using that we need to selectively update a single display memory cell.

Before loop:

1. Clear the memory cells to 0
  - 1 compare block, 1 counter, 1 not, 8x OR = 11 blocks

Loop:

1. Amount to add = 10 ^ (X % 5)
  - Exponents aren't possible, so we handle this by using a lookup table with 5 comparators, counters, NOT blocks and calculators = 20 blocks + 2 to combine them
2. Figure out X cell to write to: IF X < 5 THEN X cell = 1 ELSE X cell = 2
  - 1 AND per cell, 1 compare = 9 blocks
3. Figure out Y cell to write to: 4 compare blocks feeding into the same AND as step 2
  - 4 compare blocks = 4 blocks
4. Send write for a single cycle, feeding into same AND as step 2
  - 1 pulse block, 1 compare block = 2 blocks
5. Loop to draw next pixel

Renderer budget:

| Function                | Amount | Used |
| ----------------------- | ------ | ---- |
| Clear memory cells      | 11     | 11   |
| Calculate amount to add | 22     | 21   |
| Calculate X cell        | 9      | 10   |
| Calculate Y cell        | 4      | 4    |
| Send write              | 2      | 1    |
| Video memory            | 8      | 8    |
| Display buffer          | 8      | 9    |
| Display blocks          | 8      | 8    |
| Total                   | 72     | 72   |

### Video memory cells

Because the wiring is making it quite difficult to read the program, I have decided to document the wiring here.

The video memory is divided in 2 banks of 4 cells. Each cell stores 5 pixels.

![Video memory wiring](./images/video%20memory%20cell.png)

Each cell consists of 3 blocks - AND, OR and a calculator.

- First input on the AND block is the bank select signal
- Second input on the AND block is the Y coordinate select signal
- Third input on the AND block is the write signal configured as `shift clock` falling trigger

The OR block is used to combine the cell level write with a block level write used to clear the entire memory before rendering (connected to input 2 on the OR block)

The calculator is used to store the current value of the cell. The output loops back to input A. The formula is `Y = (A+B)*C`, where:

- A is the current value of the cell
- B is the write value from the `10^X` section
- C is set to 0 to clear the cell

### Display output

We output 1 row at a time as a binary number. This is done by looping through the bits in the memory cell and dividing by 2, outputting the remainder each time and saving it in an SR block.

## Performance

For an 8x8 display we need the following loops:

* 20 - 60ms per snake segment for processing and updating the memory cell
* 20 * 8 + 100 = 260ms for outputting the display per row, 2s for the whole display - definitely the bottleneck
