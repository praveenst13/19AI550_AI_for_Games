# Ex.No: 5  Implementation of Jumping behavior 
#### DATE: 23/08/2024   
### NAME: PRAVEEN S
### REGISTER NUMBER : 212222240077
### AIM: 
To write a python program to simulate Jumbing behavior. 
### Algorithm:
1. Start the program
2. Import the necessary modules
3. Initiate the pygame engine and window
4. Specify the necessary parameter for player height,depth,gravity,jump power. 
5. Create a game loop to simulate the continuous behavior.
6. If Quit button is pressed then quit the pygame window.
7. Move the player left when left button is pressed
8. Move the player right when right button is pressed
9. If space bar is pressed then enable the jump by increasing y axis value.
10. land the player and display the player at every timestep
11.  Stop the program
 ### Program:
```py
import pygame

# Initialize Pygame
pygame.init()

# Screen dimensions
width, height = 800, 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Simple Jumping")

# Colors
black = (255, 255, 255)

# Load the player image and scale it if needed
player_image = pygame.image.load(r'C:\Users\SEC\Downloads\images.jpeg')
player_image = pygame.transform.scale(player_image, (40, 60))  

# Player properties
player_width = player_image.get_width()
player_height = player_image.get_height()
player_x = 100
player_y = height - player_height
player_velocity = 5
jump_power = -15
gravity = 1
is_jumping = False

# Initial vertical speed
vertical_speed = 0

# Game loop
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    keys = pygame.key.get_pressed()

    # Horizontal movement
    if keys[pygame.K_LEFT]:
        player_x -= player_velocity
    if keys[pygame.K_RIGHT]:
        player_x += player_velocity

    # Jumping logic
    if not is_jumping:
        if keys[pygame.K_SPACE]:
            is_jumping = True
            vertical_speed = jump_power

    # Apply gravity
    if is_jumping:
        player_y += vertical_speed
        vertical_speed += gravity

        # Check if player lands on the ground
        if player_y >= height - player_height:
            player_y = height - player_height
            is_jumping = False

    # Clear screen
    screen.fill(black)

    # Draw player
    screen.blit(player_image, (player_x, player_y))

    # Update display
    pygame.display.flip()

    # Frame rate
    pygame.time.delay(30)

# Quit Pygame
pygame.quit()
```










### Output:


![Screenshot 2024-08-23 155201](https://github.com/user-attachments/assets/3604ad2b-4a2c-4f88-9e76-60f9bae307b7)


### Result:
Thus the simple jumping behavior  was implemented.
