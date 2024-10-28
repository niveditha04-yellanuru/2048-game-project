# 2048-game-project
import tkinter as tk
import random

class Game2048:
    def __init__(self, root):
        self.root = root
        self.root.title("2048 Game")
        self.game_over = False
        self.grid_size = 4
        self.board = [[0] * self.grid_size for _ in range(self.grid_size)]
        
        self.colors = {
            0: "#CCC0B4", 2: "#EEE4DA", 4: "#EDE0C8", 8: "#F2B179",
            16: "#F59563", 32: "#F67C5F", 64: "#F65E3B", 128: "#EDCF72",
            256: "#EDCC61", 512: "#EDC850", 1024: "#EDC53F", 2048: "#EDC22E"
        }
        
        self.create_widgets()
        self.place_random_tile()
        self.place_random_tile()
        self.update_grid()

        self.root.bind("<Key>", self.handle_keypress)
    
    def create_widgets(self):
        self.frame = tk.Frame(self.root, bg="#BBADA0", bd=5, width=400, height=400)
        self.frame.grid(padx=10, pady=10)

        self.tiles = []
        for i in range(self.grid_size):
            row = []
            for j in range(self.grid_size):
                tile = tk.Label(self.frame, text="", bg=self.colors[0], font=("Helvetica", 24), width=4, height=2)
                tile.grid(row=i, column=j, padx=5, pady=5)
                row.append(tile)
            self.tiles.append(row)

    def place_random_tile(self):
        empty_tiles = [(i, j) for i in range(self.grid_size) for j in range(self.grid_size) if self.board[i][j] == 0]
        if empty_tiles:
            x, y = random.choice(empty_tiles)
            self.board[x][y] = random.choice([2, 4])

    def compress_board(self):
        new_board = [[0] * self.grid_size for _ in range(self.grid_size)]
        for i in range(self.grid_size):
            pos = 0
            for j in range(self.grid_size):
                if self.board[i][j] != 0:
                    new_board[i][pos] = self.board[i][j]
                    pos += 1
        self.board = new_board

    def merge_board(self):
        for i in range(self.grid_size):
            for j in range(self.grid_size - 1):
                if self.board[i][j] == self.board[i][j + 1] and self.board[i][j] != 0:
                    self.board[i][j] *= 2
                    self.board[i][j + 1] = 0

    def reverse_board(self):
        for i in range(self.grid_size):
            self.board[i] = self.board[i][::-1]

    def transpose_board(self):
        self.board = [list(row) for row in zip(*self.board)]

    def move_left(self):
        self.compress_board()
        self.merge_board()
        self.compress_board()

    def move_right(self):
        self.reverse_board()
        self.move_left()
        self.reverse_board()

    def move_up(self):
        self.transpose_board()
        self.move_left()
        self.transpose_board()

    def move_down(self):
        self.transpose_board()
        self.move_right()
        self.transpose_board()

    def update_grid(self):
        for i in range(self.grid_size):
            for j in range(self.grid_size):
                tile_value = self.board[i][j]
                self.tiles[i][j].config(text=str(tile_value) if tile_value != 0 else "",
                                        bg=self.colors.get(tile_value, "#CDC1B4"))
        self.root.update_idletasks()

    def check_game_over(self):
        for i in range(self.grid_size):
            for j in range(self.grid_size):
                if self.board[i][j] == 0:
                    return False
                if j < self.grid_size - 1 and self.board[i][j] == self.board[i][j + 1]:
                    return False
                if i < self.grid_size - 1 and self.board[i][j] == self.board[i + 1][j]:
                    return False
        return True

    def handle_keypress(self, event):
        if self.game_over:
            return

        moved = False
        if event.keysym == "Left":
            self.move_left()
            moved = True
        elif event.keysym == "Right":
            self.move_right()
            moved = True
        elif event.keysym == "Up":
            self.move_up()
            moved = True
        elif event.keysym == "Down":
            self.move_down()
            moved = True

        if moved:
            self.place_random_tile()
            self.update_grid()

            if self.check_game_over():
                self.game_over = True
                self.show_game_over()

    def show_game_over(self):
        game_over_label = tk.Label(self.root, text="Game Over!", font=("Helvetica", 36), fg="red")
        game_over_label.grid(row=1, column=0, columnspan=4)

root = tk.Tk()
game = Game2048(root)
root.mainloop()
