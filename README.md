import random

class Ship:
    def __init__(self, positions):
        self.positions = positions
        self.hits = set()

    def hit(self, position):
        self.hits.add(position)
        if len(self.hits) == len(self.positions):
            return True
        return False


class Board:
    def __init__(self, size):
        self.size = size
        self.ships = []
        self.board = [['О' for _ in range(size)] for _ in range(size)]

    def add_ship(self, ship):
        self.ships.append(ship)
        for x, y in ship.positions:
            self.board[x][y] = '■'

    def is_valid_position(self, positions):
        for x, y in positions:
            if not (0 <= x < self.size and 0 <= y < self.size):
                return False
            if self.board[x][y] == '■':
                return False
            for dx, dy in [(0, 1), (0, -1), (1, 0), (-1, 0), (1, 1), (1, -1), (-1, 1), (-1, -1)]:
                nx, ny = x + dx, y + dy
                if 0 <= nx < self.size and 0 <= ny < self.size:
                    if self.board[nx][ny] == '■':
                        return False
        return True

    def is_hit(self, x, y):
        if self.board[x][y] == 'X' or self.board[x][y] == 'T':
            return True
        return False

    def attack(self, x, y):
        if self.is_hit(x, y):
            raise Exception("You have already attacked this cell.")
        for ship in self.ships:
            if (x, y) in ship.positions:
                if ship.hit((x, y)):
                    for px, py in ship.positions:
                        self.board[px][py] = 'X'
                else:
                    self.board[x][y] = 'X'
                return True
        self.board[x][y] = 'T'
        return False

    def print_board(self):
        print("   ", end="")
        for i in range(self.size):
            print(f"| {i + 1} ", end="")
        print("|")
        print("---------------------------------")
        for i in range(self.size):
            print(f" {chr(i + 65)} ", end="")
            for j in range(self.size):
                print(f"| {self.board[i][j]} ", end="")
            print("|")
            print("---------------------------------")


class Game:
    def __init__(self, size):
        self.size = size
        self.player_board = Board(size)
        self.computer_board = Board(size)
        self.player_turn = True

    def setup(self):
        ship_sizes = [3, 2, 2, 1, 1, 1, 1]

        for size in ship_sizes:
            while True:
                positions = self.get_ship_positions(size)
                if self.player_turn:
                    board = self.player_board
                else:
                    board = self.computer_board
                if board.is_valid_position(positions):
                    ship = Ship(positions)
                    board.add_ship(ship)
                    break

        self.player_turn = True

    def get_ship_positions(self, size):
        positions = []
        direction = random.choice(['horizontal', 'vertical'])
        if direction == 'horizontal':
            x = random.randint(0, self.size - size)
            y = random.randint(0, self.size - 1)
            for i in range(size):
                positions.append((x + i, y))
        else:
            x = random.randint(0, self.size - 1)
            y = random.randint(0, self.size - size)
            for i in range(size):
                positions.append((x, y + i))
        return positions

    def player_move(self):
        self.player_board.print_board()
        x = self.get_coordinate("Enter X coordinate (A-F): ")
        y = self.get_coordinate("Enter Y coordinate (1-6): ")
        while not self.computer_board.is_hit(x, y):
            if self.computer_board.attack(x, y):
                print("Hit!")
            else:
                print("Miss!")
            if self.computer_board.is_game_over():
                print("Congratulations! You won!")
                return
            self.player_board.print_board()
            x = self.get_coordinate("Enter X coordinate (A-F): ")
            y = self.get_coordinate("Enter Y coordinate (1-6): ")

    def computer_move(self):
        x = random.randint(0, self.size - 1)
        y = random.randint(0, self.size - 1)
        while not self.player_board.is_hit(x, y):
            if self.player_board.attack(x, y):
                print("Computer hit your ship!")
            else:
                print("Computer missed!")
            if self.player_board.is_game_over():
                print("Game Over! You lost!")
                return
            x = random.randint(0, self.size - 1)
            y = random.randint(0, self.size - 1)

    def get_coordinate(self, message):
        while True:
            coordinate = input(message).upper()
            if len(coordinate) == 2 and 'A' <= coordinate[0] <= chr(65 + self.size - 1) and '1' <= coordinate[1] <= str(self.size):
                x = ord(coordinate[0]) - 65
                y = int(coordinate[1]) - 1
                return x, y
            print("Invalid coordinate. Please try again.")

    def play(self):
        self.setup()
        while True:
            if self.player_turn:
                self.player_move()
            else:
                self.computer_move()
            self.player_turn = not self.player_turn


# Игра начинается здесь
game = Game(6)
game.play()
