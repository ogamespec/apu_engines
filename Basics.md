# APU Engine Basics

Here is some preliminary information about the architecture of the engines.

In time, as the engines will be studied by me, this description will be expanded.

## MUSIC vs SFX

There are usually two types of sound entities in games: MUSIC and SFX

MUSIC is a long melody playing in the background.

SFX is a short sound effect (steps, shots, etc.).

## FSM

The game engine is essentially an FSM. Switching between states is done by a main `Play` procedure.

Usually `Play` is called periodically in the NMI handler (VBlank).

## DPCM

The APU is capable of playing DPCM audio in the background. Usually a large array of digitized sound is stored in the ROM for this purpose, and it is triggered to play when required.

## APU Engine vs Mappers

If the game uses a bank-switching mapper, you need to make sure that nothing is broken when switching banks.
