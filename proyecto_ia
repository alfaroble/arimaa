import pygame
import sys

# Dimensiones del tablero y de las celdas
CELL_SIZE = 70
BOARD_SIZE = 8
WIDTH, HEIGHT = CELL_SIZE * BOARD_SIZE, CELL_SIZE * BOARD_SIZE + 50  # Espacio adicional para el título

# Colores
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GRAY = (200, 200, 200)
GOLD_COLOR = (255, 215, 0)
SILVER_COLOR = (192, 192, 192)
GREEN = (0, 255, 0)


class Piece:

    def __init__(self, name, strength, player):
        self.name = name
        self.strength = strength
        self.player = player


class Board:

    def __init__(self):
        self.size = 8
        self.grid = [[None for _ in range(self.size)]
                     for _ in range(self.size)]
        self.traps = {(2, 2), (2, 5), (5, 2), (5, 5)}

    def place_piece(self, piece, x, y):
        if self.is_within_bounds(x, y) and not self.grid[x][y]:
            self.grid[x][y] = piece

    def move_piece(self, x1, y1, x2, y2):
        if self.is_within_bounds(x2, y2) and not self.grid[x2][y2]:
            piece = self.grid[x1][y1]
            if piece:
                if abs(x2 - x1) + abs(y2 - y1) != 1:
                    return False

                if piece.name == "Conejo":
                    if piece.player == "gold" and x2 > x1:
                        return False
                    if piece.player == "silver" and x2 < x1:
                        return False

                self.grid[x1][y1] = None
                self.grid[x2][y2] = piece
                self.update_traps()

                if piece.name == "Conejo" and piece.player == "silver" and x2 == 7:
                    return "silver"

                return True
        return False

    def update_traps(self):
        for x, y in self.traps:
            piece = self.grid[x][y]
            if piece and not self.is_protected(x, y):
                self.grid[x][y] = None

    def is_within_bounds(self, x, y):
        return 0 <= x < self.size and 0 <= y < self.size

    def is_protected(self, x, y):
        piece = self.grid[x][y]
        if not piece:
            return False
        for dx, dy in [(-1, 0), (1, 0), (0, -1), (0, 1)]:
            nx, ny = x + dx, y + dy
            if self.is_within_bounds(nx, ny):
                neighbor = self.grid[nx][ny]
                if neighbor and neighbor.player == piece.player:
                    return True
        return False

    def check_victory(self):
        gold_has_rabbits = any(
            piece for row in self.grid for piece in row
            if piece and piece.name == "Conejo" and piece.player == "gold")
        silver_has_rabbits = any(
            piece for row in self.grid for piece in row
            if piece and piece.name == "Conejo" and piece.player == "silver")

        if not gold_has_rabbits:
            return "silver"
        if not silver_has_rabbits:
            return "gold"

        return None


class Game:

    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption("Arimaa")
        self.clock = pygame.time.Clock()
        self.board = Board()
        self.turn = "gold"
        self.steps_remaining = 4
        self.pieces = {"gold": [], "silver": []}
        self.selected_piece = None
        self.move_history = []
        self.font = pygame.font.SysFont(None, 40)
        self.coord_font = pygame.font.SysFont(None, 20)
        self.title_font = pygame.font.SysFont(None, 50)
        self.winner = None

    def place_initial_pieces(self, player, positions):
        piece_order = ["Elefante", "Camello", "Caballo", "Perro", "Gato"]
        strengths = {
            "Conejo": 1,
            "Gato": 2,
            "Perro": 3,
            "Caballo": 4,
            "Camello": 5,
            "Elefante": 6
        }
        positions += [(7 if player == "gold" else 0, i) for i in range(8)]

        for i, pos in enumerate(positions):
            piece_type = piece_order[min(i,
                                         len(piece_order) -
                                         1)] if i < 8 else "Conejo"
            strength = strengths[piece_type]
            piece = Piece(piece_type, strength, player)
            self.board.place_piece(piece, pos[0], pos[1])
            self.pieces[player].append(piece)

    def setup(self):
        self.place_initial_pieces("gold", [(6, i) for i in range(8)])
        self.place_initial_pieces("silver", [(1, i) for i in range(8)])

    def draw_board(self):
        for x in range(BOARD_SIZE):
            for y in range(BOARD_SIZE):
                color = WHITE if (x + y) % 2 == 0 else GRAY
                pygame.draw.rect(
                    self.screen, color,
                    pygame.Rect(y * CELL_SIZE, x * CELL_SIZE + 50, CELL_SIZE,
                                CELL_SIZE))

                if (x, y) in self.board.traps:
                    pygame.draw.rect(
                        self.screen, RED,
                        pygame.Rect(y * CELL_SIZE + 10, x * CELL_SIZE + 60,
                                    CELL_SIZE - 20, CELL_SIZE - 20))

                piece = self.board.grid[x][y]
                if piece:
                    initials = {
                        "Elefante": "E",
                        "Camello": "M",
                        "Caballo": "H",
                        "Perro": "D",
                        "Gato": "C",
                        "Conejo": "R"
                    }
                    text = f"{piece.player[0].upper()}{initials[piece.name]}"
                    color = GOLD_COLOR if piece.player == "gold" else SILVER_COLOR
                    text_surface = self.font.render(text, True, color)
                    text_rect = text_surface.get_rect(
                        center=((y * CELL_SIZE + CELL_SIZE // 2),
                                (x * CELL_SIZE + CELL_SIZE // 2 + 50)))
                    self.screen.blit(text_surface, text_rect)

        if self.selected_piece:
            sx, sy = self.selected_piece
            pygame.draw.rect(
                self.screen, GREEN,
                pygame.Rect(sy * CELL_SIZE, sx * CELL_SIZE + 50, CELL_SIZE,
                            CELL_SIZE), 3)

        for i in range(BOARD_SIZE):
            col_label = self.coord_font.render(chr(97 + i), True, BLACK)
            row_label = self.coord_font.render(str(BOARD_SIZE - i), True,
                                               BLACK)
            self.screen.blit(col_label,
                             (i * CELL_SIZE + CELL_SIZE // 2 - 5, HEIGHT - 30))
            self.screen.blit(row_label,
                             (5, i * CELL_SIZE + CELL_SIZE // 2 + 30))

    def draw_title(self):
        """Muestra el turno y los pasos restantes como un título."""
        title_text = f"Turno: {self.turn.capitalize()} - Pasos restantes: {self.steps_remaining}"
        text_surface = self.title_font.render(title_text, True, WHITE)
        text_rect = text_surface.get_rect(center=(WIDTH // 2, 25))
        self.screen.blit(text_surface, text_rect)

    def get_cell(self, pos):
        x, y = (pos[1] - 50) // CELL_SIZE, pos[0] // CELL_SIZE
        if self.board.is_within_bounds(x, y):
            return x, y
        return None, None

    def run(self):
        self.setup()
        running = True

        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False

                if event.type == pygame.MOUSEBUTTONDOWN:
                    x, y = self.get_cell(event.pos)
                    if x is not None and y is not None:
                        if self.selected_piece:
                            px, py = self.selected_piece
                            result = self.board.move_piece(px, py, x, y)
                            if result:
                                self.move_history.append((px, py, x, y))
                                self.selected_piece = None
                                self.steps_remaining -= 1

                                if result == "silver":
                                    print(
                                        "¡Silver gana al alcanzar el lado de Gold!"
                                    )
                                    running = False

                                if self.steps_remaining == 0:
                                    self.turn = "silver" if self.turn == "gold" else "gold"
                                    self.steps_remaining = 4

                                self.winner = self.board.check_victory()
                                if self.winner:
                                    print(f"¡{self.winner.capitalize()} gana!")
                                    running = False
                        else:
                            if self.board.grid[x][y] and self.board.grid[x][
                                    y].player == self.turn:
                                self.selected_piece = (x, y)

            self.screen.fill(BLACK)
            self.draw_title()  # Dibujar título
            self.draw_board()
            pygame.display.flip()
            self.clock.tick(30)

        pygame.quit()
        sys.exit()


if __name__ == "__main__":
    game = Game()
    game.run()
