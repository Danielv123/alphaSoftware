# AlphaTetris

AlphaTetris is a project to explore the viability of implementing Tetris on a mitsubishi alpha 2 PLC. The primary obstacles is the limited amount of memory and logic available on the PLC, as well as restrictions in terms of output for the display.

## Usage

To start the program, enter monitor mode and click the "Enable CPU" button halfway down on the left side. Next, click the "Start" button on the top left to activate the program clock. Control the dropping piece with the left and right arrow keys, and rotate the piece with the up arrow key.

## Hardware limitations

The alpha 2 has 5kb memory, but also a hard limit of 200 logic blocks. It has an integrated 4x12 character display, but it cannot be used for this project due to the large amount of blocks required to write arbitrary pixels to the display.

## Goals

To cut down the required logic to implement the game it had to be simplified a bit. These are the goals for the implementation:

- 10 high 8 wide display
- 1 different tetromino
- Score counter
- Game over detection
- Dropping tetrominos
- Moving tetrominos side to side

## Implementation

The implementation is split into 3 parts:

- Screen memory
- The game logic
- Display output

### Screen memory

This is the main memory of the game. It is an array of 10 bytes, each representing a row of the screen. The bits in the byte represent the pixels in the row. The screen is 8 pixels wide, so the 8 least significant bits are used.

The outer game clock counts 1 row at a time, from the bottom of the screen. The current row is read out into the game logic section of the program using `calculator` blocks.

### Game logic

The game logic is the main part of the program. It decodes the current row, checks collisions against the position of the tetromino and writes back the new row to the screen memory after merging the old row and the tetromino.

### Display output

Due to the lack of function blocks in the alpha we use a scanning display. The 8 first outputs are used for pixel 1 - 8 on the row. The 9th bit is the display clock - it toggles each time the display changes state.

This means that to have a useable display you need an external display controller. This can be implemented in any PLC, but to keep it strictly alpha we can use 3 alpha PLCs to implement a display controller.

Each alpha will have their inputs connected to the 8 first outputs of the snake controller. The 9th bit will be connected to the display clock

By counting the active row the current row state can be written to the display. Each PLC will handle 4 rows of the display.

And alternative and cooler solution is to have the main controller drive the X scan of a relay holding circuit display and a secondary controller drive the Y scan by counting the rows sent from the primary controller. This would allow for an external 8x10 display with relays/transistors and diodes that would probably sound great.
