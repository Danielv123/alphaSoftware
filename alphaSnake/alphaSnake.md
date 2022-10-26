# AlphaSnake

Snake for the mitsubishi alpha 2. Runs without any external hardware, just the integrated buttons and display.

There are 3 programs in this project:

- snakeDisplay
- modularSnake
- alphaSnake

each is a different implementation attempt.

## snakeDisplay

The first issue when creating a game on the alpha is the screen. The alpha has a small 4x12 character display, but the API to write to the screen is rather limited. For game purposes you need one `display` block per pixel, in addition to the controlling circuitry. It didn't take long to notice that this took too much space.

## modularSnake

To deal with the issue of the screen taking too much time for compute I tried to put the game logic inside the screen instead. This way the screen would only need to display the current state of the game, and not compute it. This would also allow for a larger screen.

It didn't pan out due to the amount of logic requried for the inactive parts of the screen.

## arduSnake

This is the final implementation. It is based on a pixel scanning snake program I wrote for an arduino and ported as directly as possible to the alpha. The program is designed to allow a snake length of 7 with 7 memory cells for the X and Y position of each segment.

Again, a whole lot of blocks is taken up by the screen. Each pixel has the following:

- 1 display block for output
- 1 SR block to hold the pixel
- 2 compare blocks for X and Y
- 1 AND block

for a 4x4 screen that gives us 80 blocks, almost half the available memory.

The memory is based on calculator blocks for storage and information retrieval. It contains the following:

- 1 calculator for segment value store
- 1 calculator, 1 NOT, 1 compare and 1 countUpDown for selecting the segment value
- 1 pulse block to shift the snake

This gives a total of 6 * 7 * 2 = 84 blocks for memory.

After the memory and screen this leaves us 36 blocks for the game logic. We use 33 of them.

In the end the snake is only of a very limited length - some of the memory cells are malfunctioning for some unknown reason. I'll leave it as an excercise for the reader to figure out why.

## History

The project was written spring 2019 while in VG2.
