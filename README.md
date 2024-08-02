# picture-animation
project based  on computer graphics and visulation
import pygame
import sys
import cv2
import numpy as np

# Initialize Pygame
pygame.init()
pygame.mixer.init()  # Initialize mixer for sound

# Set up initial display
default_width, default_height = 800, 600
screen = pygame.display.set_mode((default_width, default_height), pygame.RESIZABLE)
pygame.display.set_caption("Picture Animator")

# Load images using OpenCV
background = pygame.image.load('background.jpg')
background = pygame.transform.scale(background, (default_width, default_height))  # Scale to match screen size

# Function to resize the Pygame display
def resize_window(width, height):
    global screen, background
    screen = pygame.display.set_mode((width, height), pygame.RESIZABLE)
    background = pygame.transform.scale(background, (width, height))  # Scale background image accordingly

# Load and preprocess your_image.png using OpenCV
image_path = 'your_image.png'
image_cv = cv2.imread(image_path, cv2.IMREAD_UNCHANGED)

# Check if the image has an alpha channel
if image_cv.shape[2] == 4:
    b, g, r, a = cv2.split(image_cv)
    image_cv = cv2.merge([r, g, b, a])  # OpenCV uses BGR channel order, so we rearrange it to RGB for Pygame
else:
    b, g, r = cv2.split(image_cv)
    image_cv = cv2.merge([r, g, b])  # Convert BGR to RGB

# Convert image to RGB format expected by Pygame
image_cv = cv2.cvtColor(image_cv, cv2.COLOR_BGR2RGB)

# Convert image to Pygame surface
image_original = pygame.image.fromstring(image_cv.tobytes(), image_cv.shape[:2][::-1], 'RGB')

# Create a copy for manipulation
image = image_original.copy()
image_rect = image.get_rect(center=(default_width // 2, default_height // 2))  # Start at the center of the screen

# Animation parameters
x_speed = 5  # Speed of movement along x-axis
y_speed = 5  # Speed of movement along y-axis

# Keyboard control variables
move_left = False
move_right = False
move_up = False
move_down = False

# Mouse control variables
dragging = False
offset_x = 0
offset_y = 0

# Rotation angle
angle = 0

# Font setup
font = pygame.font.Font(None, 36)
text_color = (255, 255, 255)

# Load sound effects
bounce_sound = pygame.mixer.Sound('bounce.mp3')
rotate_sound = pygame.mixer.Sound('rotate.mp3')

# Main loop
running = True
clock = pygame.time.Clock()
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                move_left = True
            elif event.key == pygame.K_RIGHT:
                move_right = True
            elif event.key == pygame.K_UP:
                move_up = True
            elif event.key == pygame.K_DOWN:
                move_down = True
            elif event.key == pygame.K_r:
                angle += 10  # Rotate clockwise
                rotate_sound.play()  # Play rotate sound
            elif event.key == pygame.K_e:
                angle -= 10  # Rotate counterclockwise
                rotate_sound.play()  # Play rotate sound
        elif event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT:
                move_left = False
            elif event.key == pygame.K_RIGHT:
                move_right = False
            elif event.key == pygame.K_UP:
                move_up = False
            elif event.key == pygame.K_DOWN:
                move_down = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            if event.button == 1:  # Left mouse button
                if image_rect.collidepoint(event.pos):
                    dragging = True
                    offset_x = event.pos[0] - image_rect.x
                    offset_y = event.pos[1] - image_rect.y
        elif event.type == pygame.MOUSEBUTTONUP:
            if event.button == 1:
                dragging = False
        elif event.type == pygame.MOUSEMOTION:
            if dragging:
                image_rect.x = event.pos[0] - offset_x
                image_rect.y = event.pos[1] - offset_y
        elif event.type == pygame.VIDEORESIZE:
            # Handle window resize event
            resize_window(event.w, event.h)

    # Acceleration and deceleration parameters
    acceleration = 0.2
    deceleration = 0.1
    max_speed = 10

    # Update image position based on keyboard input
    if move_left:
        x_speed -= acceleration
    elif move_right:
        x_speed += acceleration
    else:
        if x_speed > 0:
            x_speed -= deceleration
        elif x_speed < 0:
            x_speed += deceleration

    if move_up:
        y_speed -= acceleration
    elif move_down:
        y_speed += acceleration
    else:
        if y_speed > 0:
            y_speed -= deceleration
        elif y_speed < 0:
            y_speed += deceleration

    # Limit speed
    if abs(x_speed) > max_speed:
        x_speed = max_speed if x_speed > 0 else -max_speed

    if abs(y_speed) > max_speed:
        y_speed = max_speed if y_speed > 0 else -max_speed

    # Update image position
    image_rect.x += round(x_speed)
    image_rect.y += round(y_speed)

    # Boundary collision handling and sound
    if image_rect.left < 0 or image_rect.right > screen.get_width():
        x_speed = -x_speed
        bounce_sound.play()  # Play bounce sound

    if image_rect.top < 0 or image_rect.bottom > screen.get_height():
        y_speed = -y_speed
        bounce_sound.play()  # Play bounce sound

    # Rotate the image using Pygame
    rotated_image = pygame.transform.rotate(image, angle)

    # Fill the screen with background
    screen.blit(background, (0, 0))

    # Draw the rotated image
    rotated_rect = rotated_image.get_rect(center=image_rect.center)
    screen.blit(rotated_image, rotated_rect)

    # Render text
    position_text = f"Position: ({image_rect.x}, {image_rect.y})"
    angle_text = f"Rotation: {angle} degrees"
    text_surface = font.render(position_text, True, text_color)
    screen.blit(text_surface, (10, 10))
    text_surface = font.render(angle_text, True, text_color)
    screen.blit(text_surface, (10, 50))

    # Update the display
    pygame.display.flip()

    # Cap the frame rate
    clock.tick(60)

# Quit Pygame
pygame.quit()
sys.exit()
