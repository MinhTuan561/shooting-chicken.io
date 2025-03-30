import pygame
import random

# --- Khởi tạo Pygame ---
pygame.init()
screen_width = 800
screen_height = 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Bắn Gà Rút Gọn")
clock = pygame.time.Clock()
font = pygame.font.Font(None, 36) # Sử dụng font mặc định
game_over_font = pygame.font.Font(None, 72)

# --- Tải tài nguyên (Giả định các file ảnh tồn tại) ---
# Nếu không có ảnh, chương trình sẽ báo lỗi. Bạn cần có player.png, enemy.png, bullet.png
player_img = pygame.image.load('player.png').convert_alpha()
enemy_img = pygame.image.load('enemy.png').convert_alpha()
bullet_img = pygame.image.load('bullet.png').convert_alpha()

# Lấy kích thước từ ảnh
player_rect = player_img.get_rect()
enemy_rect = enemy_img.get_rect()
bullet_rect = bullet_img.get_rect()

# --- Cài đặt Game ---
player_rect.centerx = screen_width // 2
player_rect.bottom = screen_height - 10
player_speed = 5
player_x_change = 0

enemies = []
enemy_speed = 2
enemy_spawn_rate = 60
enemy_timer = 0

bullets = []
bullet_speed = 7

score_value = 0
is_game_over = False

# --- Hàm tiện ích ---
def spawn_enemy():
    """Tạo kẻ địch mới ở vị trí ngẫu nhiên trên cùng."""
    enemy_new_rect = enemy_img.get_rect()
    enemy_new_rect.x = random.randint(0, screen_width - enemy_new_rect.width)
    enemy_new_rect.y = -enemy_new_rect.height
    enemies.append(enemy_new_rect)

def fire_bullet():
    """Tạo viên đạn mới từ vị trí người chơi."""
    bullet_new_rect = bullet_img.get_rect()
    bullet_new_rect.centerx = player_rect.centerx
    bullet_new_rect.bottom = player_rect.top
    bullets.append(bullet_new_rect)

# --- Vòng lặp chính ---
running = True
while running:
    # 1. Xử lý Input
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.KEYDOWN:
            if not is_game_over: # Chỉ xử lý input khi game chưa kết thúc
                if event.key == pygame.K_LEFT:
                    player_x_change = -player_speed
                if event.key == pygame.K_RIGHT:
                    player_x_change = player_speed
                if event.key == pygame.K_SPACE:
                    fire_bullet()
        if event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT and player_x_change < 0:
                player_x_change = 0
            if event.key == pygame.K_RIGHT and player_x_change > 0:
                player_x_change = 0

    # 2. Cập nhật Trạng thái (Logic Game)
    if not is_game_over:
        # Di chuyển người chơi
        player_rect.x += player_x_change
        player_rect.left = max(0, player_rect.left) # Không cho ra khỏi lề trái
        player_rect.right = min(screen_width, player_rect.right) # Không cho ra khỏi lề phải

        # Tạo kẻ địch mới
        enemy_timer += 1
        if enemy_timer >= enemy_spawn_rate:
            enemy_timer = 0
            spawn_enemy()

        # Di chuyển kẻ địch và kiểm tra va chạm người chơi
        enemies_to_remove = []
        for i, enemy in enumerate(enemies):
            enemy.y += enemy_speed
            if enemy.colliderect(player_rect): # Va chạm với người chơi
                is_game_over = True
                # break # Có thể dừng ngay hoặc để vẽ hết frame này
            if enemy.top > screen_height: # Ra khỏi màn hình
                enemies_to_remove.append(i)

        # Xóa gà ra khỏi màn hình (duyệt ngược)
        for index in sorted(enemies_to_remove, reverse=True):
            del enemies[index]

        # Di chuyển đạn và kiểm tra va chạm kẻ địch
        bullets_to_remove = []
        enemies_hit_indices = set() # Dùng set để tránh xử lý cùng 1 gà nhiều lần

        for i, bullet in enumerate(bullets):
            bullet.y -= bullet_speed
            if bullet.bottom < 0: # Ra khỏi màn hình
                 if i not in bullets_to_remove: # Tránh thêm trùng lặp
                    bullets_to_remove.append(i)
            else:
                # Kiểm tra va chạm đạn với từng kẻ địch
                for j, enemy in enumerate(enemies):
                    if j not in enemies_hit_indices and bullet.colliderect(enemy):
                        score_value += 1
                        bullets_to_remove.append(i)
                        enemies_hit_indices.add(j)
                        break # Đạn chỉ trúng 1 gà

        # Xóa đạn (duyệt ngược, loại bỏ trùng lặp)
        for index in sorted(list(set(bullets_to_remove)), reverse=True):
            if index < len(bullets):
                del bullets[index]

        # Xóa gà bị bắn trúng (duyệt ngược)
        for index in sorted(list(enemies_hit_indices), reverse=True):
            if index < len(enemies):
                 del enemies[index]

    # 3. Vẽ (Render)
    screen.fill((0, 0, 30)) # Nền xanh đậm

    # Vẽ các đối tượng
    screen.blit(player_img, player_rect)
    for enemy in enemies:
        screen.blit(enemy_img, enemy)
    for bullet in bullets:
        screen.blit(bullet_img, bullet)

    # Vẽ điểm
    score_surf = font.render(f"Score: {score_value}", True, (255, 255, 255))
    screen.blit(score_surf, (10, 10))

    # Vẽ Game Over
    if is_game_over:
        over_surf = game_over_font.render("GAME OVER", True, (255, 0, 0))
        over_rect = over_surf.get_rect(center=(screen_width // 2, screen_height // 2))
        screen.blit(over_surf, over_rect)

    # 4. Cập nhật màn hình
    pygame.display.flip()

    # 5. Giới hạn FPS
    clock.tick(60)

# --- Kết thúc Pygame ---
pygame.quit()

