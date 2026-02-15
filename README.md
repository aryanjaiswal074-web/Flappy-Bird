# Flappy-Bird
This project is a simple flappy bird game built using python and pygame The player controls a bird by tapping or clicking to keep it flying between pipes The game uses gravity velocity and collision detection to create smooth and realistic movement 
import pygame, random, sys, math, array

pygame.init()
pygame.mixer.init()

W, H = 450, 650
screen = pygame.display.set_mode((W, H))
pygame.display.set_caption("Flappy Bird")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 34, bold=True)

def sound(freq, dur, vol=0.4):
    rate = 44100
    n = int(rate * dur)
    buf = array.array("h")
    for i in range(n):
        buf.append(int(vol * 32767 * math.sin(2 * math.pi * freq * i / rate)))
    return pygame.mixer.Sound(buffer=buf)

snd_flap = sound(750, 0.08)
snd_score = sound(1200, 0.1)
snd_hit = sound(200, 0.3)

SKY_A = (120, 190, 255)
SKY_B = (200, 230, 255)
GREEN = (0, 180, 0)
GREEN_D = (0, 140, 0)
GROUND = (210, 180, 140)
WHITE = (255, 255, 255)
YELLOW = (255, 220, 0)
ORANGE = (255, 140, 0)

bird = {
    "x": 110,
    "y": H // 2,
    "r": 18,
    "vel": 0
}

GRAVITY = 0.45
JUMP = -8

PIPE_W = 70
PIPE_GAP = 170
PIPE_SPEED = 3
pipes = []

def new_pipe():
    h = random.randint(130, 380)
    return {"x": W, "top": h, "bot": h + PIPE_GAP, "pass": False}

pipes.append(new_pipe())
score = 0

def draw_sky():
    for y in range(H):
        t = y / H
        r = SKY_A[0] + (SKY_B[0] - SKY_A[0]) * t
        g = SKY_A[1] + (SKY_B[1] - SKY_A[1]) * t
        b = SKY_A[2] + (SKY_B[2] - SKY_A[2]) * t
        pygame.draw.line(screen, (int(r), int(g), int(b)), (0, y), (W, y))

def draw_bird():
    x, y = bird["x"], int(bird["y"])
    pygame.draw.circle(screen, YELLOW, (x, y), bird["r"])
    pygame.draw.circle(screen, WHITE, (x + 6, y - 5), 4)
    pygame.draw.circle(screen, (0, 0, 0), (x + 7, y - 5), 2)
    pygame.draw.polygon(screen, ORANGE, [(x + 18, y), (x + 28, y - 5), (x + 28, y + 5)])
    pygame.draw.ellipse(screen, (255, 200, 0), (x - 15, y, 18, 10))

def draw_pipe(p):
    pygame.draw.rect(screen, GREEN, (p["x"], 0, PIPE_W, p["top"]), border_radius=8)
    pygame.draw.rect(screen, GREEN, (p["x"], p["bot"], PIPE_W, H), border_radius=8)
    pygame.draw.rect(screen, GREEN_D, (p["x"] - 5, p["top"] - 15, PIPE_W + 10, 15))
    pygame.draw.rect(screen, GREEN_D, (p["x"] - 5, p["bot"], PIPE_W + 10, 15))

running = True
while running:
    draw_sky()

    for e in pygame.event.get():
        if e.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if e.type in (pygame.KEYDOWN, pygame.MOUSEBUTTONDOWN):
            if e.type != pygame.KEYDOWN or e.key == pygame.K_SPACE:
                bird["vel"] = JUMP
                snd_flap.play()

    bird["vel"] += GRAVITY
    bird["y"] += bird["vel"]
    draw_bird()

    for p in pipes:
        p["x"] -= PIPE_SPEED
        draw_pipe(p)

        if bird["x"] + bird["r"] > p["x"] and bird["x"] - bird["r"] < p["x"] + PIPE_W:
            if bird["y"] - bird["r"] < p["top"] or bird["y"] + bird["r"] > p["bot"]:
                snd_hit.play()
                running = False

        if p["x"] + PIPE_W < bird["x"] and not p["pass"]:
            p["pass"] = True
            score += 1
            snd_score.play()

    if pipes[-1]["x"] < W - 260:
        pipes.append(new_pipe())
    if pipes[0]["x"] < -PIPE_W:
        pipes.pop(0)

    pygame.draw.rect(screen, GROUND, (0, H - 40, W, 40))

    if bird["y"] < 0 or bird["y"] > H - 40:
        snd_hit.play()
        running = False

    screen.blit(font.render(str(score), True, WHITE), (W // 2 - 10, 25))
    pygame.display.update()
    clock.tick(60)

screen.fill((0, 0, 0))
screen.blit(font.render("GAME OVER", True, WHITE), (W // 2 - 105, H // 2 - 40))
screen.blit(font.render(f"Score: {score}", True, WHITE), (W // 2 - 85, H // 2 + 10))
pygame.display.update()
pygame.time.delay(2500)
pygame.quit()
