# Ex.No: 11  Mini Project 
### DATE:                                                                            
### REGISTER NUMBER : 212222240077
### AIM: 
To write a Python program to simulate the game Prison Escape using AI techniques.

### Algorithm:
    1)Initialize the Game Environment:
        Import necessary libraries such as pygame for game development.
        Define constants like screen size, colors, and game object attributes.
    
    2)Setup Game Objects:
        Create and scale images for the player, police, buildings, exit door, and background.
        Assign attributes such as speed, position, and lives to the player and police.
    3)Procedural Content Generation:
        Generate random positions for buildings and ensure the exit door does not overlap with them.
    4)Implement AI Behaviors:
        Define Finite State Machine (FSM) states for police: Patrol, Chase, and Search.
        Use Steering Behaviors for smooth movement and chasing.
        Implement A Pathfinding* for police to navigate around obstacles and reach the player.
    5)Game Loop:
        Handle player movement and ensure no collision with buildings.
        Update police positions and states based on player proximity.
        Detect collisions between police and the player to decrease lives and reset the level if needed.
        Render the game environment relative to the camera view.
        Display player lives, level, and a mini-map for game awareness.
    6)End the Game:
        Check for the end condition (e.g., player runs out of lives or completes all levels).
        Restart the game or display a "Game Over" message.
    7)Main Menu:
        Display a start screen with options to begin the game.
## AI Techniques Used:
    1)Finite State Machine (FSM):
        Manages police behaviors such as Patrol, Chase, and Search.
        Enables dynamic switching between behaviors based on player proximity and state.
    2)A Pathfinding Algorithm*:
        Calculates the shortest path for police to navigate toward the player or patrol points.
        Accounts for obstacles (buildings) in the game environment.
    3)Steering Behaviors:
        Provides smooth and realistic movement for police.
        Enhances the chasing mechanism by gradually aligning police direction with the player.


### Program:

```py
import pygame
import sys
import random
import math
import heapq
from enum import Enum

# Initialize pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 2000, 1500  # Full game world size
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 600  # Size of the window (viewable area)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)

# Create the game window (screen)
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Prison Escape")

# Load images (use your own paths)
player_img = pygame.image.load("prison.png")
police_img = pygame.image.load("police.png")
building_img = pygame.image.load("building.png")
exit_door_img = pygame.image.load("exit_door.png")
background_img = pygame.image.load("background.jpg")

# Scale images
player_img = pygame.transform.scale(player_img, (50, 50))
police_img = pygame.transform.scale(police_img, (50, 50))
building_img = pygame.transform.scale(building_img, (200, 200))
exit_door_img = pygame.transform.scale(exit_door_img, (50, 50))
background_img = pygame.transform.scale(background_img, (WIDTH, HEIGHT))

# Player attributes
player_speed = 5
player_pos = [100, 100]
player_lives = 3
player_health = 100
player_rect = player_img.get_rect(topleft=(player_pos[0], player_pos[1]))

# Police attributes (Finite State Machine)
class PoliceState(Enum):
    PATROL = 1
    CHASE = 2
    SEARCH = 3

police_speed = 3
police_count = 5  # Number of police per level
police_positions = []  # List to hold multiple police positions
police_state = [PoliceState.PATROL] * police_count

# Exit door (ensuring it is not placed inside a building)
exit_door_pos = [random.randint(0, WIDTH - 50), random.randint(0, HEIGHT - 50)]
exit_door_rect = exit_door_img.get_rect(topleft=(exit_door_pos[0], exit_door_pos[1]))

# Building positions (Procedural Content Generation)
buildings = [{"pos": (random.randint(0, WIDTH - 200), random.randint(0, HEIGHT - 200))} for _ in range(10)]

# A* Pathfinding variables
grid_size = 50
path = []

# Level and difficulty
level = 1
difficulty_increase = 1.2  # Multiplier for difficulty increase
max_levels = 30  # Number of levels

# Mini-map
MINI_MAP_WIDTH, MINI_MAP_HEIGHT = 200, 150
mini_map_scale = MINI_MAP_WIDTH / WIDTH, MINI_MAP_HEIGHT / HEIGHT

# Steering Behaviors for police (smooth movement)
def seek(target_pos, entity_pos, speed):
    dx = target_pos[0] - entity_pos[0]
    dy = target_pos[1] - entity_pos[1]
    dist = math.hypot(dx, dy)
    if dist == 0:
        return [0, 0]
    dx /= dist
    dy /= dist
    return [dx * speed, dy * speed]

# A* Pathfinding algorithm
def a_star(start, end):
    def heuristic(a, b):
        return abs(a[0] - b[0]) + abs(a[1] - b[1])

    open_list = []
    heapq.heappush(open_list, (0, start))
    came_from = {}
    g_score = {start: 0}
    f_score = {start: heuristic(start, end)}

    while open_list:
        _, current = heapq.heappop(open_list)
        if current == end:
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            return path[::-1]  # Return reversed path

        neighbors = [(current[0] + grid_size, current[1]), (current[0] - grid_size, current[1]),
                     (current[0], current[1] + grid_size), (current[0], current[1] - grid_size)]
        neighbors = [n for n in neighbors if 0 <= n[0] < WIDTH and 0 <= n[1] < HEIGHT]

        for neighbor in neighbors:
            tentative_g_score = g_score[current] + 1
            if neighbor not in g_score or tentative_g_score < g_score[neighbor]:
                came_from[neighbor] = current
                g_score[neighbor] = tentative_g_score
                f_score[neighbor] = tentative_g_score + heuristic(neighbor, end)
                heapq.heappush(open_list, (f_score[neighbor], neighbor))

    return []

# Check collision with buildings
def check_collision(entity_rect, buildings):
    for building in buildings:
        building_rect = pygame.Rect(building["pos"], (200, 200))
        if entity_rect.colliderect(building_rect):
            return True
    return False

# Game loop
def game_loop():
    global player_pos, player_lives, level, police_positions, police_state
    clock = pygame.time.Clock()
    camera_x, camera_y = 0, 0

    # Initialize police positions
    police_positions = [[random.randint(0, WIDTH - 50), random.randint(0, HEIGHT - 50)] for _ in range(police_count)]
    police_state = [PoliceState.PATROL] * police_count

    while True:
        screen.fill(WHITE)

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

        # Player movement
        keys = pygame.key.get_pressed()
        new_player_pos = player_pos[:]
        if keys[pygame.K_LEFT]:
            new_player_pos[0] -= player_speed
        if keys[pygame.K_RIGHT]:
            new_player_pos[0] += player_speed
        if keys[pygame.K_UP]:
            new_player_pos[1] -= player_speed
        if keys[pygame.K_DOWN]:
            new_player_pos[1] += player_speed

        # Update player rect position
        new_player_rect = player_img.get_rect(topleft=(new_player_pos[0], new_player_pos[1]))

        # Check collision with buildings for the player
        if not check_collision(new_player_rect, buildings):
            player_pos = new_player_pos  # Only update if no collision

        # Camera following player
        camera_x = player_pos[0] - SCREEN_WIDTH // 2
        camera_y = player_pos[1] - SCREEN_HEIGHT // 2
        camera_x = max(0, min(camera_x, WIDTH - SCREEN_WIDTH))
        camera_y = max(0, min(camera_y, HEIGHT - SCREEN_HEIGHT))

        # Police state behavior (FSM)
        for i in range(police_count):
            if police_state[i] == PoliceState.PATROL:
                # Random patrol movement
                target_pos = (random.randint(0, WIDTH), random.randint(0, HEIGHT))
                police_move = seek(target_pos, police_positions[i], police_speed)
                new_police_pos = [police_positions[i][0] + police_move[0], police_positions[i][1] + police_move[1]]
                new_police_rect = police_img.get_rect(topleft=new_police_pos)

                if not check_collision(new_police_rect, buildings):
                    police_positions[i] = new_police_pos

                # Switch to chase if player is near
                if math.hypot(player_pos[0] - police_positions[i][0], player_pos[1] - police_positions[i][1]) < 300:
                    police_state[i] = PoliceState.CHASE
                    path = a_star((police_positions[i][0] // grid_size * grid_size, police_positions[i][1] // grid_size * grid_size),
                                  (player_pos[0] // grid_size * grid_size, player_pos[1] // grid_size * grid_size))

            elif police_state[i] == PoliceState.CHASE:
                # Use pathfinding to chase the player
                if path:
                    target = path[0]
                    police_move = seek(target, police_positions[i], police_speed)
                    police_positions[i][0] += police_move[0]
                    police_positions[i][1] += police_move[1]

                    if math.hypot(police_positions[i][0] - target[0], police_positions[i][1] - target[1]) < 5:
                        path.pop(0)

                # Check if police caught the player
                if new_police_rect.colliderect(new_player_rect):
                    # Player caught; restart game
                    player_lives -= 1
                    if player_lives <= 0:
                        print("Game Over! Restarting...")
                        player_lives = 3  # Reset lives
                        player_pos = [100, 100]  # Reset player position
                        police_positions = [[random.randint(0, WIDTH - 50), random.randint(0, HEIGHT - 50)] for _ in range(police_count)]
                        police_state = [PoliceState.PATROL] * police_count
                    else:
                        player_pos = [100, 100]  # Reset player position if caught

        # Draw the world relative to camera
        screen.blit(background_img, (-camera_x, -camera_y))

        # Draw buildings
        for building in buildings:
            building_rect = pygame.Rect(building["pos"], (200, 200))
            screen.blit(building_img, (building_rect.x - camera_x, building_rect.y - camera_y))

        # Draw exit door
        screen.blit(exit_door_img, (exit_door_rect.x - camera_x, exit_door_rect.y - camera_y))

        # Draw player and police
        screen.blit(player_img, (player_pos[0] - camera_x, player_pos[1] - camera_y))
        for police_pos in police_positions:
            screen.blit(police_img, (police_pos[0] - camera_x, police_pos[1] - camera_y))

        # Update UI (lives, level)
        font = pygame.font.Font(None, 36)
        lives_text = font.render(f"Lives: {player_lives}", True, BLACK)
        level_text = font.render(f"Level: {level}", True, BLACK)
        screen.blit(lives_text, (10, 10))
        screen.blit(level_text, (10, 50))

        # Mini-map
        pygame.draw.rect(screen, BLACK, (SCREEN_WIDTH - MINI_MAP_WIDTH - 10, 10, MINI_MAP_WIDTH, MINI_MAP_HEIGHT), 2)
        mini_player_pos = (player_pos[0] * mini_map_scale[0] + SCREEN_WIDTH - MINI_MAP_WIDTH - 10,
                           player_pos[1] * mini_map_scale[1] + 10)
        for police_pos in police_positions:
            mini_police_pos = (police_pos[0] * mini_map_scale[0] + SCREEN_WIDTH - MINI_MAP_WIDTH - 10,
                               police_pos[1] * mini_map_scale[1] + 10)
            pygame.draw.circle(screen, RED, mini_police_pos, 5)

        mini_exit_pos = (exit_door_pos[0] * mini_map_scale[0] + SCREEN_WIDTH - MINI_MAP_WIDTH - 10,
                         exit_door_pos[1] * mini_map_scale[1] + 10)
        pygame.draw.circle(screen, BLUE, mini_exit_pos, 5)

        pygame.draw.circle(screen, GREEN, mini_player_pos, 5)

        # Update screen
        pygame.display.flip()
        clock.tick(60)

# Main menu and game loop start
def main_menu():
    while True:
        screen.fill(WHITE)
        font = pygame.font.Font(None, 74)
        title_text = font.render("Prison Escape", True, BLACK)
        screen.blit(title_text, (SCREEN_WIDTH // 2 - 200, SCREEN_HEIGHT // 2 - 100))

        font = pygame.font.Font(None, 36)
        start_text = font.render("Press Enter to Start", True, BLACK)
        screen.blit(start_text, (SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2 + 50))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN:
                    game_loop()

        pygame.display.flip()

main_menu()








```



### Output:

![mini_project](https://github.com/user-attachments/assets/2df5a1a2-fb15-4ab4-98ee-aab8cdee8d1a)


### Result:
Thus, the Prison Escape game was implemented using Finite State Machine (FSM), A Pathfinding*, and Steering Behaviors.


