import pygame
import random
import math

# 初期設定
pygame.init()
screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("FPS Game")
clock = pygame.time.Clock()

# 色
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# プレイヤー設定
player_img = pygame.Surface((50, 50))
player_img.fill(WHITE)
player_rect = player_img.get_rect(center=(400, 300))
player_speed = 5

# 敵の設定
enemy_img = pygame.Surface((30, 30))
enemy_img.fill(RED)
enemy_speed = 2
enemies = []

# 弾の設定
bullet_img = pygame.Surface((10, 10))
bullet_img.fill(BLACK)
bullets = []
bullet_speed = 7

# 敵生成
def spawn_enemy():
    x = random.randint(0, 800)
    y = random.randint(0, 600)
    while math.hypot(x - player_rect.centerx, y - player_rect.centery) < 100:
        x = random.randint(0, 800)
        y = random.randint(0, 600)
    enemies.append(enemy_img.get_rect(center=(x, y)))

# メインループ
running = True
while running:
    screen.fill((0, 100, 200))
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            bullet = bullet_img.get_rect(center=player_rect.center)
            bullets.append((bullet, pygame.mouse.get_pos()))

    # プレイヤー移動
    keys = pygame.key.get_pressed()
    if keys[pygame.K_w]: player_rect.y -= player_speed
    if keys[pygame.K_s]: player_rect.y += player_speed
    if keys[pygame.K_a]: player_rect.x -= player_speed
    if keys[pygame.K_d]: player_rect.x += player_speed

    # 弾移動
    for bullet, target in bullets[:]:
        angle = math.atan2(target[1] - bullet.centery, target[0] - bullet.centerx)
        bullet.x += int(bullet_speed * math.cos(angle))
        bullet.y += int(bullet_speed * math.sin(angle))
        if not screen.get_rect().contains(bullet):
            bullets.remove((bullet, target))

    # 敵生成と移動
    if random.random() < 0.02:
        spawn_enemy()
    for enemy in enemies[:]:
        angle = math.atan2(player_rect.centery - enemy.centery, player_rect.centerx - enemy.centerx)
        enemy.x += int(enemy_speed * math.cos(angle))
        enemy.y += int(enemy_speed * math.sin(angle))
        if player_rect.colliderect(enemy):
            running = False
        for bullet, _ in bullets:
            if bullet.colliderect(enemy):
                enemies.remove(enemy)
                bullets.remove((bullet, _))
                break

    # 描画
    screen.blit(player_img, player_rect)
    for bullet, _ in bullets:
        screen.blit(bullet_img, bullet)
    for enemy in enemies:
        screen.blit(enemy_img, enemy)
    
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
