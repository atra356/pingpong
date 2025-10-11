import pygame
import sys
import os
import random

# Inisialisasi Pygame
pygame.init()

# Ukuran layar
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pingpong Game")

# Warna
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 128, 255)
YELLOW = (255, 255, 0)
GREEN = (0, 255, 0)

# Paddle
PADDLE_WIDTH, PADDLE_HEIGHT = 10, 100
paddle_speed = 7

# Font
font = pygame.font.SysFont("Arial", 30)
win_font = pygame.font.SysFont("Arial", 50)

# Skor dan logika game
score_left = 0
score_right = 0
winning_score = 5
paused = True
game_over = False

# Objek permainan
left_paddle = pygame.Rect(50, HEIGHT//2 - PADDLE_HEIGHT//2, PADDLE_WIDTH, PADDLE_HEIGHT)
right_paddle = pygame.Rect(WIDTH - 50 - PADDLE_WIDTH, HEIGHT//2 - PADDLE_HEIGHT//2, PADDLE_WIDTH, PADDLE_HEIGHT)
ball = pygame.Rect(WIDTH//2 - 10, HEIGHT//2 - 10, 20, 20)
ball_speed_x = 0
ball_speed_y = 0

# Musik latar
if os.path.exists('undertale.mp3'):
    pygame.mixer.music.load('undertale.mp3')
    pygame.mixer.music.play(-1)
else:
    print("Musik 'undertale.mp3' tidak ditemukan.")

# Background
if os.path.exists('sans.jpg'):
    background_img = pygame.image.load('sans.jpg')
    background_img = pygame.transform.scale(background_img, (WIDTH, HEIGHT))
else:
    print("Gambar 'sans.jpg' tidak ditemukan.")
    background_img = pygame.Surface((WIDTH, HEIGHT))
    background_img.fill((30, 30, 30))

clock = pygame.time.Clock()

def randomize_ball():
    global ball_speed_x, ball_speed_y
    ball_speed_x = random.choice([-1, 1]) * random.randint(4, 6)
    ball_speed_y = random.choice([-1, 1]) * random.randint(3, 5)
    ball.center = (WIDTH//2, HEIGHT//2)

def reset_positions():
    left_paddle.y = HEIGHT//2 - PADDLE_HEIGHT//2
    right_paddle.y = HEIGHT//2 - PADDLE_HEIGHT//2
    randomize_ball()

def reset_game():
    global score_left, score_right, paused, game_over
    score_left = 0
    score_right = 0
    paused = True
    game_over = False
    reset_positions()

def draw():
    screen.blit(background_img, (0, 0))

    # Paddle dan bola
    pygame.draw.rect(screen, WHITE, left_paddle)
    pygame.draw.rect(screen, WHITE, right_paddle)
    pygame.draw.ellipse(screen, RED, ball)

    # Garis tengah
    for i in range(10, HEIGHT, 40):
        pygame.draw.rect(screen, WHITE, (WIDTH//2 - 5, i, 10, 20))

    # Skor
    left_score_text = font.render(str(score_left), True, BLUE)
    right_score_text = font.render(str(score_right), True, YELLOW)
    screen.blit(left_score_text, (WIDTH//4, 20))
    screen.blit(right_score_text, (WIDTH * 3 // 4, 20))

    if paused and not game_over:
        pause_text = win_font.render("PAUSED - Tekan P untuk Mulai", True, GREEN)
        screen.blit(pause_text, (WIDTH//2 - pause_text.get_width()//2, HEIGHT//2 - 40))

    if game_over:
        winner = "Kiri" if score_left == winning_score else "Kanan"
        win_text = win_font.render(f"{winner} Menang!", True, RED)
        restart_text = font.render("Tekan R untuk ulang - ESC untuk keluar", True, WHITE)
        screen.blit(win_text, (WIDTH//2 - win_text.get_width()//2, HEIGHT//2 - 60))
        screen.blit(restart_text, (WIDTH//2 - restart_text.get_width()//2, HEIGHT//2 + 10))

def main():
    global ball_speed_x, ball_speed_y
    global score_left, score_right, paused, game_over

    reset_game()
    running = True

    while running:
        clock.tick(60)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_p and not game_over:
                    paused = not paused
                if event.key == pygame.K_ESCAPE:
                    running = False
                if event.key == pygame.K_r and game_over:
                    reset_game()

        if not paused and not game_over:
            # Kontrol paddle kiri
            keys = pygame.key.get_pressed()
            if keys[pygame.K_w] and left_paddle.top > 0:
                left_paddle.y -= paddle_speed
            if keys[pygame.K_s] and left_paddle.bottom < HEIGHT:
                left_paddle.y += paddle_speed

            # Kontrol paddle kanan
            if keys[pygame.K_UP] and right_paddle.top > 0:
                right_paddle.y -= paddle_speed
            if keys[pygame.K_DOWN] and right_paddle.bottom < HEIGHT:
                right_paddle.y += paddle_speed

            # Gerak bola
            ball.x += ball_speed_x
            ball.y += ball_speed_y

            # Pantulan dinding
            if ball.top <= 0 or ball.bottom >= HEIGHT:
                ball_speed_y *= -1

            # Pantulan paddle
            if ball.colliderect(left_paddle) and ball_speed_x < 0:
                ball_speed_x *= -1
            if ball.colliderect(right_paddle) and ball_speed_x > 0:
                ball_speed_x *= -1

            # Skor
            if ball.left <= 0:
                score_right += 1
                reset_positions()

            if ball.right >= WIDTH:
                score_left += 1
                reset_positions()

            # Menang
            if score_left == winning_score or score_right == winning_score:
                game_over = True
                paused = True

        draw()
        pygame.display.flip()

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
