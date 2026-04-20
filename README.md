import pygame
import sys

# Constants for the game window and board
ROWS = 6
COLS = 7
SQUARESIZE = 100
WIDTH = COLS * SQUARESIZE
HEIGHT = (ROWS + 1) * SQUARESIZE
SIZE = (WIDTH, HEIGHT)
RADIUS = int(SQUARESIZE / 2 - 5)

# Colors (RGB format)
BLUE = (0, 0, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)

def create_board():
    """Creates and initializes the board as a 2D list (Matrix)."""
    board = [[0 for _ in range(COLS)] for _ in range(ROWS)]
    return board

def draw_board(board, screen):
    """Displays the board graphically using the pygame library."""
    for c in range(COLS):
        for r in range(ROWS):
            # Draw the blue background rectangle for the grid
            pygame.draw.rect(screen, BLUE, (c * SQUARESIZE, r * SQUARESIZE + SQUARESIZE, SQUARESIZE, SQUARESIZE))
            # Draw the empty black circles (holes)
            pygame.draw.circle(screen, BLACK, (int(c * SQUARESIZE + SQUARESIZE / 2), int(r * SQUARESIZE + SQUARESIZE + SQUARESIZE / 2)), RADIUS)
    
    # Draw the pieces (1 for Red, 2 for Yellow)
    for c in range(COLS):
        for r in range(ROWS):
            if board[r][c] == 1:
                pygame.draw.circle(screen, RED, (int(c * SQUARESIZE + SQUARESIZE / 2), HEIGHT - int(r * SQUARESIZE + SQUARESIZE / 2)), RADIUS)
            elif board[r][c] == 2:
                pygame.draw.circle(screen, YELLOW, (int(c * SQUARESIZE + SQUARESIZE / 2), HEIGHT - int(r * SQUARESIZE + SQUARESIZE / 2)), RADIUS)
    pygame.display.update()

def get_move(event):
    """Determines which column a player selects based on their mouse click."""
    if event.type == pygame.MOUSEBUTTONDOWN:
        posx = event.pos[0]
        col = int(posx // SQUARESIZE)
        return col
    return None

def drop_piece(board, column, player):
    """Places a piece in the lowest available row of the selected column (Gravity)."""
    for r in range(ROWS):
        if board[r][column] == 0:
            board[r][column] = player
            return True
    return False

def check_win(board, player):
    """Scans the board for four consecutive pieces in any direction."""
    # Check horizontal locations for win
    for c in range(COLS - 3):
        for r in range(ROWS):
            if board[r][c] == player and board[r][c+1] == player and board[r][c+2] == player and board[r][c+3] == player:
                return True

    # Check vertical locations for win
    for c in range(COLS):
        for r in range(ROWS - 3):
            if board[r][c] == player and board[r+1][c] == player and board[r+2][c] == player and board[r+3][c] == player:
                return True

    # Check positively sloped diagonals
    for c in range(COLS - 3):
        for r in range(ROWS - 3):
            if board[r][c] == player and board[r+1][c+1] == player and board[r+2][c+2] == player and board[r+3][c+3] == player:
                return True

    # Check negatively sloped diagonals
    for c in range(COLS - 3):
        for r in range(3, ROWS):
            if board[r][c] == player and board[r-1][c+1] == player and board[r-2][c+2] == player and board[r-3][c+3] == player:
                return True
    
    return False

def check_draw(board):
    """Determines if the board is full (no empty spaces left)."""
    for c in range(COLS):
        if board[ROWS-1][c] == 0:
            return False
    return True

def play_game():
    """Main game loop function that controls the flow of the game."""
    pygame.init()
    screen = pygame.display.set_mode(SIZE)
    board = create_board()
    game_over = False
    turn = 0 # 0 for Player 1 (Red), 1 for Player 2 (Yellow)

    draw_board(board, screen)
    myfont = pygame.font.SysFont("monospace", 75)

    while not game_over:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()

            # Visual feedback: show piece at top before dropping
            if event.type == pygame.MOUSEMOTION:
                pygame.draw.rect(screen, BLACK, (0, 0, WIDTH, SQUARESIZE))
                posx = event.pos[0]
                if turn == 0:
                    pygame.draw.circle(screen, RED, (posx, int(SQUARESIZE / 2)), RADIUS)
                else:
                    pygame.draw.circle(screen, YELLOW, (posx, int(SQUARESIZE / 2)), RADIUS)
                pygame.display.update()

            col = get_move(event)
            if col is not None:
                player = 1 if turn == 0 else 2
                
                if drop_piece(board, col, player):
                    if check_win(board, player):
                        label = myfont.render(f"P{player} WINS!!", 1, RED if player == 1 else YELLOW)
                        screen.blit(label, (40, 10))
                        game_over = True
                    elif check_draw(board):
                        label = myfont.render("DRAW!!", 1, BLUE)
                        screen.blit(label, (40, 10))
                        game_over = True
                    
                    draw_board(board, screen)
                    turn = (turn + 1) % 2 # Alternate turns

                    if game_over:
                        pygame.time.wait(3000) # Wait 3 seconds before closing

if __name__ == "__main__":
    play_game()
