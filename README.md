# project-Ai
Implementing a game using min-max algorithm using Ai

import tkinter as tk
from tkinter import messagebox

def create_board(): # this function is used to create a board for the interface
    board = []
    for _ in range(8): #8 rows
        board.append([' '] * 8) # in 8 colums
    return board

def is_board_full(board): # this function is used for checking if the board is full with brikes
    for row in board:
        for col in row:
            if col == ' ':  # if there is an empty square then return false
                return False
    return True

def print_board(board):
    for row in board:
        print(' '.join(row))
    print('\n')


def is_valid_move(board, row, col): #check if the movement is valid and within the rules that was discussed in the project
    if board[row][col] != ' ':
        return False

    if col == 0 or col == 7:
        return True

    if board[row][col - 1] != ' ' or board[row][col + 1] != ' ':
        return True

    return False


def make_move(board, row, col, player):
    if is_valid_move(board, row, col):
        board[row][col] = player
        return True
    return False

# this function is implemented in a way that it has to check
# if a player wins by checking 4 consecutive columns or rows or diagonal
#it checks if the variable col is less than 4 and  checks if all the elements in a sequence meet a certain condition.
# The condition being checked is board[row][c] == player, which compares each element of the sequence
# (referred to as c) to the variable player.
# If all the elements in the sequence satisfy the condition (i.e., board[row][c] is equal to player for all values of c),
# the function returns True. Otherwise, it returns False.
def check_win(board, player):
    for row in range(8):
        for col in range(8):
            if board[row][col] == player: #checking for 64 square
                if col < 4 and all(board[row][c] == player for c in range(col, col+5)):
                    return True
                if row < 4 and all(board[r][col] == player for r in range(row, row+5)):
                    return True
                if row < 4 and col < 4 and all(board[row+r][col+r] == player for r in range(5)):
                    return True
                if row < 4 and col >= 4 and all(board[row+r][col-r] == player for r in range(5)):
                    return True
    return False



def minimax(board, depth, maximizing_player):
    if depth == 0 or check_win(board, '■') or check_win(board, '□'):
        if check_win(board, '■'): #it calls the function that we have implement it previously
            return 1
        elif check_win(board, '□'):
            return -1
        else:
            return 0

    if maximizing_player:
        max_eval = float('-inf') # we define the variable max_eval with positive infinivty just like in the alph in alph-beta algorithm
                                # and starting with positive infinity ensures that the first encountered score will always be greater.and
        for row in range(8):
            for col in range(8):
                if is_valid_move(board, row, col):
                    board[row][col] = '■'
                    eval_score = minimax(board, depth - 1, False)
                    board[row][col] = ' '
                    max_eval = max(max_eval, eval_score)
        return max_eval
    else:
        min_eval = float('inf') # we define the oponent player with a value of +infinity just like in the alplha in alpha -beta algorithm
                                # and starting with negative infinity ensures that the first encountered score will always be greater.
        for row in range(8):
            for col in range(8):
                if is_valid_move(board, row, col):
                    board[row][col] = '□'
                    eval_score = minimax(board, depth - 1, True)
                    board[row][col] = ' '
                    min_eval = min(min_eval, eval_score)
        return min_eval


def get_best_move(board): # to determine the best move we have to compare between
    best_score = float('-inf') # we define it with -infinity as it will never decrease , it will always getting higher as a value
    best_move = None
    for row in range(8):
        for col in range(8):
            if is_valid_move(board, row, col):
                board[row][col] = '■'
                score = minimax(board, 3, False)  # The depth of 3 make computer move in 3 second approximately
                board[row][col] = ' '
                if score > best_score: #This condition checks if the current move's score is greater than the previous best score. If so, it updates
                    best_score = score
                    best_move = (row, col)
    return best_move

class GameApp:

    def __init__(self):
        self.window = tk.Tk()
        self.window.title('Magnetic Cave')
        self.board = create_board()

        self.game_mode = 0
        self.current_player = '■'

        self.buttons = [[None for _ in range(8)] for _ in range(8)]

        self.game_mode_window()

    def game_mode_window(self):
        mode_window = tk.Toplevel(self.window)
        mode_window.title("Select Game Mode")

        mode_text = "Please select a game mode:\n" \
                    "1. Manual entry for both '■' and '□' moves.\n" \
                    "2. Manual entry for '■' moves & automatic minimax moves for '□'.\n" \
                    "3. Manual entry for '□' moves & automatic minimax moves for '■'."

        mode_label = tk.Label(mode_window, text=mode_text)
        mode_label.pack()

        mode_entry = tk.Entry(mode_window)
        mode_entry.pack()

        def submit_mode():
            try:
                self.game_mode = int(mode_entry.get())
                if self.game_mode in [1, 2, 3]:
                    mode_window.destroy()
                    self.init_board()
                else:
                    messagebox.showerror("Error", "Invalid selection. Please select a valid game mode!!.")
            except ValueError:
                messagebox.showerror("Error", "Invalid input. Please enter a valid integer.")

        submit_button = tk.Button(mode_window, text='Submit', command=submit_mode)
        submit_button.pack()

    def init_board(self):
        colors = ['white', 'grey']
        for i in range(8):
            for j in range(8):
                color_index = (i + j) % 2  # Determine the color index based on row and column indices
                color = colors[color_index]
                self.buttons[i][j] = tk.Button(self.window, height=2, width=4, bg=color,
                                               command=lambda row=i, col=j: self.move(row, col))
                self.buttons[i][j].grid(row=i, column=j)

    def update_board(self):
        for i in range(8):
            for j in range(8):
                self.buttons[i][j]['text'] = self.board[i][j]

    def move(self, row, col):
        # Manual player's move
        if self.game_mode == 1 or (self.game_mode == 2 and self.current_player == '■') or (
                self.game_mode == 3 and self.current_player == '□'):
            if make_move(self.board, row, col, self.current_player):
                self.current_player = '□' if self.current_player == '■' else '■'
                self.update_board()

                if check_win(self.board, '□' if self.current_player == '■' else '■'):
                    messagebox.showinfo("Info", f"Player {'□' if self.current_player == '■' else '■'} wins!")
                    self.window.quit()
                elif is_board_full(self.board):
                    messagebox.showinfo("Info", "DRAW!!")
                    self.window.quit()
            else:
                messagebox.showerror("Error", "Invalid move, please try again.")

        # Computer's move
        if self.game_mode == 2 and self.current_player == '□' or self.game_mode == 3 and self.current_player == '■':
            row, col = get_best_move(self.board)
            if make_move(self.board, row, col, self.current_player):
                self.current_player = '□' if self.current_player == '■' else '■'
                self.update_board()

                if check_win(self.board, '□' if self.current_player == '■' else '■'):
                    messagebox.showinfo("Info", f"Computer ({'□' if self.current_player == '■' else '■'}) wins!")
                    self.window.quit()
                elif is_board_full(self.board):
                    messagebox.showinfo("Info", "DRAW!!")
                    self.window.quit()



    def run(self): # this will run a window
        self.window.mainloop()



if __name__ == '__main__':
    app = GameApp()
    app.run()

    #checks if the current module is being run as the main module
    # and if the condition is true, it creates an instance of the GameApp
    # class and calls its run() method. This suggests that the program likely
    # contains a class named GameApp with a run() method that performs the main logic or execution of the application.
