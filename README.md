# hackman
A penman like game i made in 10 minutes



import pygame
import sys
import random

# --- Initialize Pygame ---
pygame.init()

# --- Screen setup ---
TILE_SIZE = 40
ROWS, COLS = 7, 10
WIDTH, HEIGHT = COLS * TILE_SIZE, ROWS * TILE_SIZE
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mini Pac-Man with Smarter Ghost")

# --- Colors ---
BLACK = (0, 0, 0)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
BLUE = (0, 0, 255)
RED = (255, 0, 0)

# --- Map: 1 = wall, 0 = dot/empty ---
level = [
    [1,1,1,1,1,1,1,1,1,1],
    [1,0,0,0,1,0,0,0,0,1],
    [1,0,1,0,1,0,1,1,0,1],
    [1,0,1,0,0,0,0,1,0,1],
    [1,0,1,1,1,1,0,1,0,1],
    [1,0,0,0,0,0,0,1,0,1],
    [1,1,1,1,1,1,1,1,1,1],
]

# --- Pac-Man start ---
pac_x, pac_y = 1, 1

# --- Ghost start ---
ghost_x, ghost_y = 8, 5
ghost_dir = random.choice([(0,1),(0,-1),(1,0),(-1,0)])  # Initial random direction

# --- Score ---
score = 0
dots_total = sum(row.count(0) for row in level)

# --- Clock ---
clock = pygame.time.Clock()
running = True
game_over = False

# --- Game loop ---
while running:
    clock.tick(5)  # control speed

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    if not game_over:
        # --- Pac-Man movement ---
        keys = pygame.key.get_pressed()
        new_x, new_y = pac_x, pac_y
        if keys[pygame.K_w]: new_y -= 1
        if keys[pygame.K_s]: new_y += 1
        if keys[pygame.K_a]: new_x -= 1
        if keys[pygame.K_d]: new_x += 1
        if level[new_y][new_x] != 1:
            pac_x, pac_y = new_x, new_y

        # --- Eat dots ---
        if level[pac_y][pac_x] == 0:
            level[pac_y][pac_x] = -1
            score += 1
            if score == dots_total:
                game_over = True  # Victory

        # --- Ghost movement ---
        dx, dy = ghost_dir
        next_x, next_y = ghost_x + dx, ghost_y + dy
        if level[next_y][next_x] != 1:
            ghost_x, ghost_y = next_x, next_y
        else:
            # Hit a wall → pick new random direction
            directions = [(0,1),(0,-1),(1,0),(-1,0)]
            random.shuffle(directions)
            for d in directions:
                nx, ny = ghost_x + d[0], ghost_y + d[1]
                if level[ny][nx] != 1:
                    ghost_dir = d
                    break

        # --- Check collision ---
        if pac_x == ghost_x and pac_y == ghost_y:
            game_over = True

    # --- Draw map ---
    screen.fill(BLACK)
    for row in range(ROWS):
        for col in range(COLS):
            rect = pygame.Rect(col*TILE_SIZE, row*TILE_SIZE, TILE_SIZE, TILE_SIZE)
            if level[row][col] == 1:
                pygame.draw.rect(screen, BLUE, rect)
            elif level[row][col] == 0:
                pygame.draw.circle(screen, WHITE, rect.center, 5)

    # --- Draw Pac-Man ---
    pygame.draw.circle(screen, YELLOW, (pac_x*TILE_SIZE + TILE_SIZE//2, pac_y*TILE_SIZE + TILE_SIZE//2), TILE_SIZE//2 - 2)

    # --- Draw Ghost ---
    pygame.draw.circle(screen, RED, (ghost_x*TILE_SIZE + TILE_SIZE//2, ghost_y*TILE_SIZE + TILE_SIZE//2), TILE_SIZE//2 - 2)

    # --- Score ---
    font = pygame.font.SysFont(None, 24)
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (5, HEIGHT - 25))

    # --- Game over / victory ---
    if game_over:
        font_big = pygame.font.SysFont(None, 48)
        if score == dots_total:
            text = font_big.render("You Win!", True, WHITE)
        else:
            text = font_big.render("Game Over", True, WHITE)
        screen.blit(text, (WIDTH//2 - text.get_width()//2, HEIGHT//2 - text.get_height()//2))

    pygame.display.flip()

pygame.quit()
sys.exit()
