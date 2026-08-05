## Overview

The 49 puzzles of *These Walls Move* (TWM) require the player to move a pawn through a regular grid of sliding tiles. These grids range in size from 3-by-3 to 5-by-5, though the vast majority are 4-by-4. The goal of the game is to complete all puzzles in order by including the pawn’s current tile in a contiguous path from all starting arrows to the finish arrow. 

The player can move the pawn to any adjacent, available tile, shift the pawn’s current column vertically without moving the pawn’s current tile, or shift the pawn’s current row horizontally without moving the pawn’s current tile. Thus, any tile that would occupy the pawn’s space skips to the next available position, and any tile at either end of a vector migrates to the opposite side of the grid. 

Consider a shift in the positive direction. If the pawn occupies a vector’s last tile, this vector’s second-to-last tile shifts to the beginning. If the pawn occupies a vector’s first tile, the vector’s last tile shifts to the second-from-first position.

## Puzzle Mechanics

As the game progresses, puzzles introduce two obstacles, the first of which is a minotaur. A minotaur never leaves its tile. If the player shifts a minotaur’s vector, this minotaur and its tile move together. A minotaur must never occupy the same path as the pawn. If the player shifts a vector in such a way as to compromise the pawn, the game undoes the move. Thus, a solution cannot contain a minotaur, up to two of which can appear in a puzzle.

Next, puzzles introduce rotating tiles, which override the player’s ability to shift vectors. As long as the pawn occupies any of these special tiles, the player can rotate all of them by 90 degrees either clockwise or counter-clockwise without affecting any other tile. The player cannot rotate them individually. Note that, if the pawn occupies a regular tile, the player can move rotating tiles by shifting vectors. If the player rotates these special tiles in such a way as to allow any minotaur to reach the pawn, the game undoes the move. Again, up to two rotating tiles can appear in a puzzle.

More peripherally, the player can resolve previous puzzles by selecting them from a lobby. Above the lobby, a menu offers up to 3 save files to account for multiple users.

## System Architecture

The most fundamental game object is a grid that stores tiles in a grid data structure, renders the board, and optimizes the board creation process for any grid size. Beyond storing the data structure, the grid object arranges all tiles and pieces on the board.

### Tiles

The grid object starts to create a puzzle by generating tiles. Each tile stores its “type” (i.e, dead end, straightaway, elbow, T, or cross), its angle, and four booleans to indicate cardinal directions with clear paths. For example, an elbow tile open to the bottom and the right would have its “bottom” and “right” booleans resolve to true. Using the top-left position of the grid as a reference point, the object places all tiles relative to this position upon their creation.

### Movement

The pawn houses the player’s capabilities, interpreting most input into concrete effects on the board. The grid object creates and places the pawn on an edge tile, while the pawn receives input from the player to move itself or shift tiles. The pawn can move in any cardinal direction as long as the path is clear. Thus, the pawn checks the direction away from the current tile without neglecting to check the opposite direction away from the target tile. The pawn cannot move outside the puzzle.

### Shifting Tiles

Another fundamental mechanic is shifting tiles relative to the pawn. In a 3-by-3 puzzle, such behavior is simple because there are only two tiles to move, and moving in opposite directions produces the same effect. Whatever tile from which the player shifts, the game performs a simple swap between the other two tiles in the vector.

For any puzzle greater than 3-by-3, there are 4 distinct cases for a single direction. Consider a shift in the positive direction (i.e, down or right): 

* If the **pawn is at the start of a vector**, this vector’s last tile becomes second-from-first and all tiles beside the pawn’s tile move to the next space.   
    
* If the **pawn is at the end of a vector**, this vector’s second-to-last tile becomes first and all tiles beside the pawn’s tile move to the next space.   
    
* If the **pawn’s position is second-from-first**, this vector’s first tile becomes third-from-first, the last tile moves to the beginning, and all tiles beside the pawn’s tile move to the next space.   
    
* Finally, if the **pawn is anywhere else**, all tiles beside the pawn’s tile move to the next space only, skipping the player’s position. 

With regard to the negative direction, not only must tiles move in the opposite way, but a case for the pawn being second-to-last supplants the case for the pawn being second-from-first. The game performs a genuine swap for every position the pawn does not occupy, looping the tile at the end around to the other side and shifting all other tiles past the pawn’s tile to the newly open space.

### Minotaurs

Minotaurs are the primary, dynamic obstacle. Not only do the minotaurs follow their tiles, but they also run a script to detect the pawn. More precisely, minotaurs use Breadth-First Search (BFS) to find the pawn whenever the player shifts tiles, since only such movement could yield a compromising path. Again, the game undoes the last shift or rotation if a minotaur finds the pawn.

### Rotating Tiles

Rotating tiles appear later in the game, sometimes alongside minotaurs. Instead of a behavior living inside tiles, a “pivot” object references a tile to follow and runs a script on that tile to override the player script. When the player is on the tile and rotates it, the pivot achieves two changes, turning all rotatable tiles by 90 degrees and revising each rotating tile’s four directional booleans to synchronize with the rotation.

### Arrows

Start and finish arrows govern completion conditions, ultimately linking one puzzle to the next. Start arrows work similarly to minotaurs whenever the player shifts tiles, using BFS to find the pawn, resetting the search by unflagging all reachable tiles, and finding the finish arrow from the pawn. 

The pawn always begins from a start arrow, but many puzzles feature multiple start arrows. In such cases, the pawn starts at one of the tiles adjacent to these arrows. To evaluate the player’s solution accurately, each start arrow must verify that it has a path through the pawn’s current tile to the finish arrow. Accordingly, each arrow reports its verification to a “master” object for all arrows, and if all start arrows report success, the arrow master allows the player to proceed.

Finally, each arrow stores the requisite direction of its adjacent tile so that a contiguous path genuinely connects the arrows rather than the first and last tiles. For example, a start arrow that points to the right from the left side of the grid requires the first tile in the path to have an opening on the left. A finish arrow that points to the right from the right side of the grid requires the last tile in the path to have an opening on the right.

### The Lobby

Similar to the puzzles themselves, the lobby from which to select puzzles employs a 7-by-7 grid data structure to reference each level. The lobby object uses a global integer to determine which levels the player has completed, graying out any levels above the integer and keeping the player’s cursor from accessing them, as the player has not yet unlocked these levels. The game increments this global variable whenever the player completes the latest level.

### Saving

As the singular indicator of the player’s progress, the number of unlocked levels in the lobby requires a saving mechanic. When the player completes a puzzle, the grid object overwrites the save file in the current slot, updating and encrypting the number of available puzzles as well as the amount of time the player has spent in that file. The player can store up to 3 save files on a device.

### Main Menu

The main menu’s four options appear after a brief introductory animation. The player can select any of the 3 files or exit the game. One may delete files from this menu, as well. The menu also shows information about each file so that the player knows what to expect from them.

## Input Handling

*These Walls Move* accommodates three media of input: a keyboard, a conventional gamepad, and an Android. The game features neither tutorials nor customizable keybindings because it seeks to evoke the experience of finding and tinkering with an unfamiliar device. Thus, the controls are so intuitive as to tether each function to multiple keys. 

To return from a puzzle to the lobby or from the lobby to the main menu, the player can press Q, backspace, delete, or escape on the keyboard, “face 2” on the gamepad (often recognized by laymen as “B” or circle), or the back button on the Android. Though the game relays no specific information, the player has multiple right answers to guess on the device with the most buttons. A subject who is new to games would likely select backspace to return to an earlier dialogue, but someone who has played games for a long time might press Q.

## Performance Considerations

Far from a graphically intensive game, *These Walls Move* can run at 60 frames per second on Windows devices and Android phones. Of course, more recent technology will have the best results.

## Known Limitations

*These Walls Move* runs only in landscape on Androids. This is because GameMaker’s ability to change from landscape to portrait is slightly inconsistent, and the game works much more predictably in one set of dimensions. Landscape was preferable to portrait not only because the grids are bigger, but also because landscape has been a successful option for many popular games like *Plague Incorporated* and *Angry Birds*.

At 4 to 6 hours, *These Walls Move* is not very long in comparison to other games. However, with concrete mechanics and a simple layout, the game has enough flexibility to include further levels or procedurally generate puzzles in later versions.