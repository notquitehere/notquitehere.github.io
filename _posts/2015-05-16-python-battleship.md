---
layout:     post
title:      "Python battleship"
date:       2015-05-16 21:22:43
modified:   2015-05-16 21:22:43
categories: code
tags:       python
published:  true
---

For the lovely ladies on my short Introduction to Programming course here is **an** answer to the code challenges at the end of the course, this is not the only answer, nor is it necessarily the best answer but it does work and has been thoroughly tested. The challenges were to add the following functionality to your program:

* use `len(x)` to check if the user has input something, if they haven't ask them to re-enter their guess
* use `x.isnumeric()` to check if the user has input a valid number
* ability to check if that location has been guessed before (change the location to a "G", print the board once you've done so)
* limited number of guesses
* would you like to play again?

I decided to use `try`/`except` rather than `isnumeric()` to check whether the input is valid or not because it gives a neater solution.

```python
from random import randint

guess_row = 0
guess_col = 0

def print_board(board):
    for x in board:
        print " ".join(x)

def rand_loc(board):
    return randint(0,len(board)-1)

def guesses():
    global guess_row, guess_col
    # this allows us to modify variables within a function (look up scope in python for further explanation)

    guess_row = raw_input("Guess row: ")
    guess_col = raw_input("Guess column: ")
    # check if the user has input something, make them try again if they haven't

    while len(guess_row) == 0 or len(guess_col) == 0:
        print "Oops you didn't enter anything in one of the co-ordinates, please try again."
        guesses()
        #it's fine to call a function from within itself

    # check if the user has input a valid number or not by attempting to convert a string to a number
    try:
        int(guess_row)
        int(guess_col)
    except ValueError:
        print "I'm sorry at least one of those wasn't a valid number, please try again."
        guesses()

# check to see if the user would like to play again
def play_again():
    play_again = raw_input("Would you like to play again?")

    if play_again.lower() == "yes" or play_again.lower() == "y":
        battleship()
    elif play_again.lower() == "no" or play_again.lower() == "n":
        print "Goodbye"
    else:
        print "Please answer yes or no:"
        play_again()

def battleship():
    global guess_row, guess_col
    board = []

    # initialises an empty list for i in range(0,5):
    board.append(['O'] * 5)

    ship_row = rand_loc(board)
    ship_col = rand_loc(board)

    no_of_guesses = 10

    guesses()

    while no_of_guesses != 0:
    # limits the number of guesses
        guess_row = int(guess_row)
        guess_col = int(guess_col)
        no_of_guesses -= 1

        if guess_row == ship_row and guess_col == ship_col:
            print "You win!"
            board[ship_row][ship_col] = "X"
            print_board(board)
            break

        # Check to see if the guess location is actually on the board
        elif guess_row < 0 or guess_row >= len(board) or guess_col < 0 or guess_col >= len(board):
            print "Sorry, that's not even in the ocean \n You have %s guesses remaining." % no_of_guesses guesses()

        # Check to see if that location has already been guessed
        elif board[guess_row][guess_col] == "G":
            no_of_guesses += 1
            # counteracts the removal of a guess

            print "You've already tried there! \n You have %s guesses remaining." % no_of_guesses
            print_board(board)
            guesses()

        else:
            print "You missed my battleship. \n You have %s guesses remaining." % no_of_guesses

            board[guess_row][guess_col] = "G"
            print_board(board)
            guesses()

    if no_of_guesses == 0:
        print "Sorry, you lose. \n my battleship was at: \n row: %s \n column: %s" % (ship_row, ship_col)
        board[ship_row][ship_col] == "X"
        print_board(board)
        play_again()
    else:
        play_again()
        battleship()
```

Remember there are plenty more things you can do with this. You could:

* ask the user how many goes they'd like
* make the board bigger and have a second battleship
* make a two player version
* make your battleship bigger
* &c
