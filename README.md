import pygame
import sys

# Initialize Pygame
pygame.init()

# Screen settings
width, height = 800, 600
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption("Ball Movement Demo")

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
GRAY = (100, 100, 100)
LIGHT_GRAY = (200, 200, 200)

# Default ball settings
circle_radius = 30
circle_color = WHITE
animation_duration = 4000  # Default speed (4 seconds)

# Game variables
clock = pygame.time.Clock()
target_fps = 60
circle_pos = (width // 2, height - circle_radius * 2)  # Initialize circle position
target_pos = circle_pos
start_time = None
trail_length = 5
trail_positions = []
in_menu = True  # Start in the menu

# Load font
font = pygame.font.Font(None, 48)  # Use a larger font size for buttons

# Function to update position
def update_position(start_pos, end_pos, t):
    return (
        start_pos[0] + (end_pos[0] - start_pos[0]) * t,
        start_pos[1] + (end_pos[1] - start_pos[1]) * t
    )

# Function to draw circle
def draw_circle(surface, pos, radius, color, opacity=255):
    circle_surface = pygame.Surface((radius * 2, radius * 2), pygame.SRCALPHA)
    pygame.draw.circle(circle_surface, (*color, opacity), (radius, radius), radius)
    surface.blit(circle_surface, (pos[0] - radius, pos[1] - radius))

# Function to display text
def draw_text(surface, text, pos, font, color=WHITE):
    text_surface = font.render(text, True, color)
    surface.blit(text_surface, pos)

# Button class
class Button:
    def __init__(self, text, pos, width, height, action):
        self.text = text
        self.rect = pygame.Rect(pos, (width, height))
        self.action = action
        self.default_color = GRAY
        self.hover_color = LIGHT_GRAY
        self.current_color = self.default_color

    def draw(self, surface):
        pygame.draw.rect(surface, self.current_color, self.rect, border_radius=10)
        draw_text(surface, self.text, (self.rect.x + 10, self.rect.y + 10), font)

    def update(self, mouse_pos):
        if self.rect.collidepoint(mouse_pos):
            self.current_color = self.hover_color
            if pygame.mouse.get_pressed()[0]:  # Left mouse button clicked
                self.action()
        else:
            self.current_color = self.default_color

# Opening animation
def opening_animation():
    logo_radius = 0
    growing = True
    fade_out_duration = 1200  # Duration to fade out
    start_time = pygame.time.get_ticks()
    fade_opacity = 255  # Initialize fade_opacity

    while True:
        elapsed_time = pygame.time.get_ticks() - start_time

        # Handle the growing animation
        if growing:
            logo_radius = min(200, logo_radius + 2)  # Increase radius
            if logo_radius >= 200:
                growing = False  # Start fading out after reaching max size
                start_time = pygame.time.get_ticks()  # Reset timer for fade out
        else:
            # Handle the fading animation
            if elapsed_time < fade_out_duration:
                fade_opacity = max(0, 255 - (255 * elapsed_time / fade_out_duration))
            else:
                fade_opacity = 0
                break  # Exit the loop after fading out

        # Clear the screen
        screen.fill(BLACK)
        draw_circle(screen, (width // 2, height // 2), logo_radius, WHITE, fade_opacity)
        pygame.display.flip()
        clock.tick(target_fps)

# Menu loop
def menu():
    global circle_radius, circle_color, animation_duration, in_menu

    # Menu settings
    buttons = [
        Button("Small Ball", (50, 50), 200, 50, lambda: set_ball_size(20)),
        Button("Medium Ball", (50, 120), 200, 50, lambda: set_ball_size(30)),
        Button("Large Ball", (50, 190), 200, 50, lambda: set_ball_size(40)),
        Button("Slow Speed", (50, 260), 200, 50, lambda: set_ball_speed(6000)),
        Button("Normal Speed", (50, 330), 200, 50, lambda: set_ball_speed(4000)),
        Button("Fast Speed", (50, 400), 200, 50, lambda: set_ball_speed(2000)),
        Button("White Ball", (50, 470), 200, 50, lambda: set_ball_color(WHITE)),
        Button("Red Ball", (50, 540), 200, 50, lambda: set_ball_color(RED)),
        Button("Blue Ball", (50, 610), 200, 50, lambda: set_ball_color(BLUE)),
        Button("Fire Ball", (50, 680), 200, 50, lambda: set_ball_color((255, 165, 0))),  # Set fire ball color
        Button("Start Game", (50, 750), 200, 50, lambda: start_game())
    ]

    while in_menu:
        screen.fill(BLACK)

        # Draw buttons
        for button in buttons:
            button.draw(screen)

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

        mouse_pos = pygame.mouse.get_pos()
        for button in buttons:
            button.update(mouse_pos)

        pygame.display.flip()
        clock.tick(target_fps)

# Setters for ball settings
def set_ball_size(size):
    global circle_radius
    circle_radius = size

def set_ball_speed(duration):
    global animation_duration
    animation_duration = duration

def set_ball_color(color):
    global circle_color
    circle_color = color

# Create a fireball effect
def draw_fire_ball(surface, pos, radius, opacity):
    gradient_colors = [
        (255, 0, 0, opacity),    # Red
        (255, 165, 0, opacity),  # Orange
        (255, 255, 0, opacity),  # Yellow
    ]
    
    for i, color in enumerate(gradient_colors):
        pygame.draw.circle(surface, color, pos, radius - i * (radius // len(gradient_colors)))

def start_game():
    global in_menu, start_time, circle_pos, target_pos
    in_menu = False
    global start_time  # Declare start_time as global to use it here
    global circle_pos  # Declare circle_pos as global to avoid UnboundLocalError
    circle_pos = (width // 2, height - circle_radius * 2)
    target_pos = circle_pos
    start_time = None

# Main loop
def main():
    global start_time, circle_pos, in_menu  # Declare global variables here
    opening_animation()  # Play the opening animation
    while True:
        if in_menu:
            menu()
        else:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    # Set new target position on mouse click
                    target_pos = event.pos
                    start_time = pygame.time.get_ticks()

            # Clear the screen
            screen.fill(BLACK)

            # Update ball position if moving
            if start_time is not None:
                elapsed_time = pygame.time.get_ticks() - start_time
                t = min(elapsed_time / animation_duration, 1)  # Normalized time between 0 and 1
                circle_pos = update_position(circle_pos, target_pos, t)
                if t >= 1:  # Stop updating if reached target
                    start_time = None  # Reset start_time to stop further updates

            # Track trail positions for motion blur effect
            trail_positions.append(circle_pos)
            if len(trail_positions) > trail_length:
                trail_positions.pop(0)

            # Draw trail effect for motion blur
            for i, pos in enumerate(trail_positions):
                opacity = int(255 * (i + 1) / trail_length)
                draw_circle(screen, pos, circle_radius, circle_color, opacity)

            # Draw the main ball with a fire effect if it's a fire ball
            if circle_color == (255, 165, 0):
                draw_fire_ball(screen, circle_pos, circle_radius, 255)  # Draw fire ball
            else:
                draw_circle(screen, circle_pos, circle_radius, circle_color)  # Regular ball

            # Update the display and control the frame rate
            pygame.display.flip()
            clock.tick(target_fps)

# Start the game
if __name__ == "__main__":
    main()  # Call the main function to start the game
