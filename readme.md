import pygame
import random
import math

# Initialize Pygame
pygame.init()

# Screen dimensions
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Space Shooter")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
YELLOW = (255, 255, 0)

# Clock to control frame rate
clock = pygame.time.Clock()
FPS = 60

# Player Settings
player_width = 50
player_height = 40
player_x = SCREEN_WIDTH // 2 - player_width // 2
player_y = SCREEN_HEIGHT - 60
player_speed = 6

# Bullet Settings
bullet_width = 5
bullet_height = 15
bullet_speed = 7
bullets = []

# Enemy Settings
enemy_width = 40
enemy_height = 30
enemy_speed = 3
enemies = []
num_enemies = 6

for i in range(num_enemies):
    enemies.append({
        'x': random.randint(0, SCREEN_WIDTH - enemy_width),
        'y': random.randint(-150, -40)
    })

# Score
score = 0
font = pygame.font.SysFont("Arial", 24)

def show_score(x, y):
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (x, y))

def draw_player(x, y):
    # Draws a sleek green triangle representing the spaceship
    pygame.draw.polygon(screen, GREEN, [(x + player_width // 2, y), (x, y + player_height), (x + player_width, y + player_height)])

# Game Loop
running = True
game_over = False

while running:
    screen.fill(BLACK)
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
            
        # Shooting logic (Spacebar)
        if event.type == pygame.KEYDOWN and not game_over:
            if event.key == pygame.K_SPACE:
                # Bullet spawns from the tip of the ship
                bullets.append({'x': player_x + player_width // 2 - bullet_width // 2, 'y': player_y})

    if not game_over:
        # Player Movement
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player_x > 0:
            player_x -= player_speed
        if keys[pygame.K_RIGHT] and player_x < SCREEN_WIDTH - player_width:
            player_x += player_speed

        # Move and update bullets
        for bullet in bullets[:]:
            bullet['y'] -= bullet_speed
            if bullet['y'] < 0:
                bullets.remove(bullet)

        # Move and update enemies
        for enemy in enemies:
            enemy['y'] += enemy_speed
            
            # If an enemy reaches the bottom, respawn it at the top
            if enemy['y'] > SCREEN_HEIGHT:
                enemy['x'] = random.randint(0, SCREEN_WIDTH - enemy_width)
                enemy['y'] = random.randint(-100, -40)

            # Collision Check: Enemy hits Player (Game Over)
            if (player_x < enemy['x'] + enemy_width and player_x + player_width > enemy['x'] and
                player_y < enemy['y'] + enemy_height and player_y + player_height > enemy['y']):
                game_over = True

        # Collision Check: Bullet hits Enemy
        for bullet in bullets[:]:
            for enemy in enemies:
                if (bullet['x'] < enemy['x'] + enemy_width and bullet['x'] + bullet_width > enemy['x'] and
                    bullet['y'] < enemy['y'] + enemy_height and bullet['y'] + bullet_height > enemy['y']):
                    if bullet in bullets:
                        bullets.remove(bullet)
                    # Respawn enemy and boost score
                    enemy['x'] = random.randint(0, SCREEN_WIDTH - enemy_width)
                    enemy['y'] = random.randint(-100, -40)
                    score += 10

        # Drawing everything
        draw_player(player_x, player_y)
        
        for bullet in bullets:
            pygame.draw.rect(screen, YELLOW, (bullet['x'], bullet['y'], bullet_width, bullet_height))
            
        for enemy in enemies:
            pygame.draw.rect(screen, RED, (enemy['x'], enemy['y'], enemy_width, enemy_height))
            
        show_score(10, 10)
    else:
        Game Over Screen
        game_over_text = font.render("GAME OVER! Press R to Restart", True, RED)
        screen.blit(game_over_text, (SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2))
        
        keys = pygame.key.get_pressed()
        if keys[pygame.K_r]:
            game_over = False
            score = 0
            player_x = SCREEN_WIDTH // 2 - player_width // 2
            bullets.clear()
            for enemy in enemies:
                enemy['x'] = random.randint(0, SCREEN_WIDTH - enemy_width)
                enemy['y'] = random.randint(-150, -40)

    pygame.display.update()
    clock.tick(FPS)

pygame.quit()
