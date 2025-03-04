# Flying-bird-game-
Flying bird game 
import pygame
import random
import socket
import threading

# Initialize pygame
pygame.init()

# Game Constants
WIDTH, HEIGHT = 500, 600
BIRD_RADIUS = 20
GRAVITY = 0.5
JUMP_STRENGTH = -10
PIPE_WIDTH = 70
PIPE_GAP = 150

# Colors
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# Load assets
bird_img = pygame.image.load("bird.png")
pipe_img = pygame.image.load("pipe.png")
background_img = pygame.image.load("background.png")
flap_sound = pygame.mixer.Sound("flap.wav")
point_sound = pygame.mixer.Sound("point.wav")

# Create window
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flying Bird Game - Multiplayer Competition")

# Bird setup
bird_y = HEIGHT // 2
bird_velocity = 0
score = 0
high_score = 0

# Pipes setup
pipes = []
for i in range(3):
    pipe_x = WIDTH + i * 200
    pipe_height = random.randint(100, 400)
    pipes.append([pipe_x, pipe_height])

clock = pygame.time.Clock()
running = True

# Networking setup
server = ("localhost", 5555)
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.setblocking(False)

def send_score():
    global score
    message = f"SCORE:{score}"
    sock.sendto(message.encode(), server)

def receive_data():
    while running:
        try:
            data, _ = sock.recvfrom(1024)
            print("Server:", data.decode())
        except:
            pass

threading.Thread(target=receive_data, daemon=True).start()

while running:
    screen.blit(background_img, (0, 0))
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_SPACE:
                bird_velocity = JUMP_STRENGTH
                flap_sound.play()
    
    # Bird movement
    bird_velocity += GRAVITY
    bird_y += bird_velocity
    screen.blit(bird_img, (100, int(bird_y)))
    
    # Pipe movement
    for pipe in pipes:
        pipe[0] -= 3
        if pipe[0] < -PIPE_WIDTH:
            pipe[0] = WIDTH
            pipe[1] = random.randint(100, 400)
            score += 1
            send_score()
            point_sound.play()
        screen.blit(pipe_img, (pipe[0], 0))
        screen.blit(pipe_img, (pipe[0], pipe[1] + PIPE_GAP))
    
    # Display score
    font = pygame.font.Font(None, 36)
    score_text = font.render(f"Score: {score}", True, (0, 0, 0))
    screen.blit(score_text, (20, 20))
    
    pygame.display.update()
    clock.tick(30)

    # Check for bird going out of screen (Leave condition)
    if bird_y < 0 or bird_y > HEIGHT:
        running = False
        if score > high_score:
            high_score = score

pygame.quit()
