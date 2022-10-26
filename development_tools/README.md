# Mitsubishi Alpha 2 developer tools

The mitsubishi alpha 2 is an old industrial PLC platform for simple control applications. It has an integrated screen with a few buttons, a few inputs and a few relay outputs. Most importantly though, it has a very simple and easy to use programming environment which makes it a great introduction to PLC programming.

To program in alpha you need the following:

- Alpha 2 PLC
- Programming cable
- RS232 to USB adapter
- Mitsubishi Alpha 2 software

The programming software is AL-PCS-WIN v2.70, uploaded here for simplicity.

## Installation instructions

To install the software, simply extract the zip folder and run the setup.exe file. The software will install to C:\Program Files (x86)\ALVLS

## Simulation

One of the most useful features of the alpha 2 platform is the fully featured simulation mode. This allows you to test your program without having to connect to a PLC. To use the simulation mode, simply press the purple "S" in the GUI.

![Simulation mode](https://i.imgur.com/8Q0czeT.png)

When in simulation mode you can click inputs to toggle them, and the outputs will be displayed in the GUI.

Lines are colored after their activation status. To see the screen in simulation mode, press `CTRL + TAB` to switch to the screen view.
