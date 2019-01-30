# Othello
Ian Magnusson
CS5001
Homework 6
November 18th, 2018

----Othello design----

CLASSES:

GameState
	attributes:
	- game_board, a Board object, the main game space
	- comp_turn, a bool, True if computer's turn, False if human's turn

	methods:
	- __init__(int), initialize with board size; note requires even n value 
	- human_move(int, int), used with onscreen click. it attempts human move
	  until valid click, then invokes computer move. Human skipped if no 
	  valid moves and if neither have moves left invokes end condition and
	  score file i/o
	- computer_move(), uses AI module to select and play a move and then invoke
	  human_move (within onscreen click), skips comp turn if they have no valid
	  moves left. and if neither have moves left invokes end condition and
	  score file i/o

Board
	attributes:
	- stones, a list of lists of strings, stone colors or '' for empty
	- human_moves, a matrix of Move objects for human
	- comp_moves, a matrix of Move objects for the computer
	
	methods:
	- __init__(list of list of strings), initialize w/ color matrix,
	  this will first build a matrix of stone color strings and then
	  build Moves dictionary with colors as keys and matrices of
	  Moves as values. Moves will be computed using the check_moves
	  method on each row/column/diag containing a non-empty tile.
	- __getitem__(self, a pair of ints), return stone color at coords

	- process_move(string, pair of ints), execute move for color
	  designated by string and at coordinates given by pair of ints.
	  Checks if valid (returns True if yes, else False). Adds stone at
	  given coordinates and flips stones designated by corresponding Move
	  object. Tracks rows/columns/diags with new or flipped stones and
	  calls check_moves on each of these to update moves dictionary.

	- check_moves(list of stones, list of moves), updates Moves for a
	  given row, column, diagonal, or skew diagonal based on its sequence
	  of stones and empty tiles (see check_moves pseudocode below)

	# return lists containing a given sequence of Moves or stone colors
	- get_move_row(int, color)
	- get_move_column(int, color)
	- get_move_diagonal(int, color)
	- get_move_skew_diagonal(int, color)
	- get_stone_row(int)
	- get_stone_column(int)
	- get_stone_diagonal(int)
	- get_stone_skew_diagonal(int)

	- deep_copy(), returns Board object with copied values (for AI)

Move
	attributes:
	- legal, a bool (True if any coordinates in will_flip)
	- coordinates, a pair of ints, row and column of possible move
	- num_flip, an int, number of disks this move will flip
	- will_flip, a dictionary with "row", "column", "diagonal", and
	  "skew diagonal" as keys and values for each as a pair of indexes
	  for the range of stones that will change color 
	
	methods:
	- __eq__(self, pair of ints), compares with coordinate pair
	- set_flip(string, (int, int)), set will_flip[string] to given range,
	  if set to no flips in this direction, checks if any flips left, if
	  none left then set illegal.

AI module
	functions:
	- pick_move(Board, string), given a Board object and a color name
	  evaluates moves in Board by invoking helper functions, selects
	  highest rank move.
	- evaluate_positions(Board, string), returns a list of ranks
	  corresponding to each move of the given color, weighted to 
	  prioritize corners, then edges and avoid tiles adjacent to corners,
	  then tiles adjacent to edges.
	- evaluate_outcomes(Board, string), returns a list of ranks
	  corresponding to each move of the given color, weighted to 
	  minimum disks flipped, until board fill threshold is reached and
	  to maximize the number of moves the comp has over the player,
	  this is achieved by deep copying the Board and recording the
	  outcomes after testing each move. (If time allows I will implement
	  this to go to an arbitrary depth of future moves).	

Graphics module
	functions:
	- display(Board), renders the current board state in turtle graphics
	- draw_board(int), code from instructor
	- draw_lines(turtle, int), code from instructor
	- draw_circle(string, pair of ints), draws a stone of given color 
	  at coords
	- coords_to_tile(int, int, int), maps cartesian coordinates
	  onto row/column coordinates given num rows/columns
	- tile_to_coords(int, int, int), expands row and column coords to
	  cartesian coordinates in the middle of the specified tile, given
	  the number of rows/columns
	
PSEUDOCODE FOR Board.check_moves:
# note that this algorithm works for an arbitrary number of colors

check that sequence has more than one color in it and at least one empty tile

for color in colors:
	iterate thru tiles until empty or color found
	if empty:
		enter state1
	if color:
		enter state2

state1:
	iterate until color position, saving last empty position
	if end of sequence before color:
		no new move, proceed to next color in colors
	
	if both positions exist and are not adjacent:
		record last empty as move and pos b/t as tiles flipped
		enter state2

	if positions are adjacent:
		no new move, just enter state2

state2:
	iterate until empty position, saving last color position
	if end of sequence before empty:
		no new move, proceed to next color in colors
	
	if both positions exist and are not adjacent:
		record empty pos as move and pos b/t as tiles flipped
		enter state1

	if positions are adjacent:
		no new move, just enter state1

DRIVER:
	# note: because of onscreenclick driver is very short and I have
	#	also included driver like method gamestate.attempt_move

main():
	- initialize GameState object with board size, colors, first player
	 is human and second player is comp
	- display starting board
	- call onscreen click with gamestate.attempt_move and prevent
	 window from closing with turtle.done method.

gamestate.attempt_move(x, y):
	- convert cartesian coords to row and column pair
	- attempt move, if illegal do not advance to next player
	- if no moves left, announce scores and winner, invoke score i/o
	- if move legal, invoke board.process_move, render board, and then
	 turn off click and invoke comp move (comp move will render again
 	 after it finishes and turn human player click back on. If no valid
	 human moves after comp move, human turn is skipped)

WISHLIST:
- board state saves and loads
- AI war gaming to arbitrary depth of future moves (tree data structure?)
- investigate possibility of storing move rank in move and only recomputing
  move ranks when they are changed
