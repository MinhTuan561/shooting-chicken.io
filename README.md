# shooting-chicken.io<!DOCTYPE html>
import pygame
import random
import math

# 1. Khởi tạo Pygame
pygame.init()

# 2. Cài đặt màn hình
screen_width = 800
screen_height = 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Bắn Gà (Chicken Invaders Clone)")

# --- Tải tài nguyên (Nếu không có ảnh, thay bằng vẽ hình) ---
try:
    # Người chơi
    player_img = pygame.image.load('player.png') # Cần có file player.png
    player_img = pygame.transform.scale(player_img, (64, 64)) # Thay đổi kích thước nếu cần
    player_width = 64
    player_height = 64
except pygame.error as e:
    print("Không thể tải ảnh player.png:", e)
    # Vẽ hình chữ nhật thay thế nếu không tải được ảnh
    player_img = pygame.Surface((64, 64))
    player_img.fill((0, 0, 255)) # Màu xanh dương
    player_width = 64
    player_height = 64


try:
    # Kẻ địch (Gà)
    enemy_img = pygame.image.load('enemy.png') # Cần có file enemy.png
    enemy_img = pygame.transform.scale(enemy_img, (64, 64)) # Thay đổi kích thước nếu cần
    enemy_width = 64
    enemy_height = 64
except pygame.error as e:
    print("Không thể tải ảnh enemy.png:", e)
    # Vẽ hình chữ nhật thay thế
    enemy_img = pygame.Surface((64, 64))
    enemy_img.fill((255, 0, 0)) # Màu đỏ
    enemy_width = 64
    enemy_height = 64

try:
    # Đạn
    bullet_img = pygame.image.load('bullet.png') # Cần có file bullet.png
    bullet_img = pygame.transform.scale(bullet_img, (16, 32)) # Thay đổi kích thước nếu cần
    bullet_width = 16
    bullet_height = 32
except pygame.error as e:
    print("Không thể tải ảnh bullet.png:", e)
     # Vẽ hình chữ nhật thay thế
    bullet_img = pygame.Surface((10, 20))
    bullet_img.fill((255, 255, 0)) # Màu vàng
    bullet_width = 10
    bullet_height = 20


# Hình nền (tùy chọn)
# background = pygame.image.load('background.png')
# background = pygame.transform.scale(background, (screen_width, screen_height))

# --- Cài đặt Game ---
# Người chơi
player_x = (screen_width - player_width) / 2
player_y = screen_height - player_height - 10
player_speed = 5
player_x_change = 0

# Kẻ địch (Gà)
enemies = []
enemy_speed = 2
enemy_spawn_rate = 60 # Số khung hình giữa mỗi lần tạo gà mới (thấp hơn = nhiều gà hơn)
enemy_timer = 0

# Đạn
bullets = []
bullet_speed = 7
bullet_state = "ready" # "ready" - sẵn sàng bắn, "fire" - đang bay

# Điểm số
score_value = 0
font = pygame.font.Font('freesansbold.ttf', 32) # Font chữ và cỡ chữ
text_x = 10
text_y = 10

# Game Over
game_over_font = pygame.font.Font('freesansbold.ttf', 64)
is_game_over = False

# Clock (điều khiển FPS)
clock = pygame.time.Clock()

# --- Hàm vẽ ---
def show_score(x, y):
    score = font.render("Score : " + str(score_value), True, (255, 255, 255)) # Màu trắng
    screen.blit(score, (x, y))

def game_over_text():
    over_text = game_over_font.render("GAME OVER", True, (255, 255, 255))
    text_rect = over_text.get_rect(center=(screen_width/2, screen_height/2))
    screen.blit(over_text, text_rect)

def draw_player(x, y):
    screen.blit(player_img, (x, y))

def draw_enemy(x, y):
    screen.blit(enemy_img, (x, y))

def fire_bullet(x, y):
    global bullet_state
    bullet_state = "fire"
    # Vị trí đạn sẽ ở giữa phía trên tàu
    bullet_x = x + (player_width / 2) - (bullet_width / 2)
    bullet_y = y
    bullets.append([bullet_x, bullet_y])

def draw_bullet(x, y):
     screen.blit(bullet_img, (x, y))

# --- Hàm kiểm tra va chạm ---
def is_collision(obj1_x, obj1_y, obj1_width, obj1_height, obj2_x, obj2_y, obj2_width, obj2_height):
    # Tạo hình chữ nhật bao quanh đối tượng để kiểm tra va chạm
    rect1 = pygame.Rect(obj1_x, obj1_y, obj1_width, obj1_height)
    rect2 = pygame.Rect(obj2_x, obj2_y, obj2_width, obj2_height)
    return rect1.colliderect(rect2) # Trả về True nếu va chạm

# --- Vòng lặp chính của Game ---
running = True
while running:

    # 3. Xử lý sự kiện (Input)
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        # Xử lý nhấn phím
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                player_x_change = -player_speed
            if event.key == pygame.K_RIGHT:
                player_x_change = player_speed
            if event.key == pygame.K_SPACE:
                # Chỉ bắn khi đạn trước đã bay xa hoặc chưa bắn
                # (Có thể bỏ điều kiện này nếu muốn bắn liên tục)
                # if bullet_state == "ready":
                fire_bullet(player_x, player_y)

        # Xử lý thả phím
        if event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                player_x_change = 0

    # --- Cập nhật trạng thái Game ---

    if not is_game_over:
        # Cập nhật vị trí người chơi
        player_x += player_x_change
        # Giới hạn di chuyển trong màn hình
        if player_x <= 0:
            player_x = 0
        elif player_x >= screen_width - player_width:
            player_x = screen_width - player_width

        # --- Quản lý kẻ địch ---
        enemy_timer += 1
        if enemy_timer >= enemy_spawn_rate:
            enemy_timer = 0
            enemy_x = random.randint(0, screen_width - enemy_width)
            enemy_y = -enemy_height # Xuất hiện từ mép trên
            enemies.append([enemy_x, enemy_y])

        # Di chuyển kẻ địch và kiểm tra va chạm với người chơi
        enemies_to_remove = []
        for i in range(len(enemies)):
            enemies[i][1] += enemy_speed # Di chuyển xuống

            # Kiểm tra va chạm giữa gà và người chơi
            collision_player = is_collision(player_x, player_y, player_width, player_height,
                                            enemies[i][0], enemies[i][1], enemy_width, enemy_height)
            if collision_player:
                is_game_over = True
                # Không cần break, để vẽ hết gà hiện tại rồi mới hiện Game Over
                # break

            # Xóa gà nếu ra khỏi màn hình
            if enemies[i][1] > screen_height:
                enemies_to_remove.append(i)

        # Xóa gà đã xử lý (đi ra khỏi màn hình) - duyệt ngược để tránh lỗi index
        for index in sorted(enemies_to_remove, reverse=True):
            del enemies[index]


        # --- Quản lý đạn ---
        bullets_to_remove = []
        enemies_hit_indices = set() # Dùng set để tránh xóa cùng 1 gà nhiều lần trong 1 frame

        for i in range(len(bullets)):
            bullets[i][1] -= bullet_speed # Di chuyển đạn lên

            # Kiểm tra va chạm giữa đạn và gà
            for j in range(len(enemies)):
                 # Chỉ kiểm tra nếu gà chưa bị đánh dấu để xóa trong frame này
                if j not in enemies_hit_indices:
                    collision_bullet_enemy = is_collision(bullets[i][0], bullets[i][1], bullet_width, bullet_height,
                                                        enemies[j][0], enemies[j][1], enemy_width, enemy_height)
                    if collision_bullet_enemy:
                        score_value += 1
                        bullets_to_remove.append(i) # Đánh dấu đạn cần xóa
                        enemies_hit_indices.add(j) # Đánh dấu gà cần xóa
                        # Có thể thêm hiệu ứng nổ ở đây
                        break # Một viên đạn chỉ trúng một con gà

            # Xóa đạn nếu bay ra khỏi màn hình
            if bullets[i][1] < -bullet_height: # Kiểm tra vị trí y của đạn
                 if i not in bullets_to_remove: # Tránh thêm trùng lặp
                    bullets_to_remove.append(i)


        # Xóa đạn đã xử lý (va chạm hoặc ra khỏi màn hình) - duyệt ngược
        for index in sorted(list(set(bullets_to_remove)), reverse=True): # Dùng set để loại bỏ trùng lặp index đạn
            if index < len(bullets): # Kiểm tra index hợp lệ trước khi xóa
               del bullets[index]


        # Xóa gà đã bị bắn trúng - duyệt ngược
        for index in sorted(list(enemies_hit_indices), reverse=True):
             if index < len(enemies): # Kiểm tra index hợp lệ
                del enemies[index]


    # --- Vẽ lên màn hình (Render) ---
    screen.fill((0, 0, 0)) # Màu nền đen (hoặc vẽ ảnh nền)
    # if 'background' in locals():
    #     screen.blit(background, (0, 0))

    # Vẽ người chơi
    draw_player(player_x, player_y)

    # Vẽ tất cả kẻ địch
    for enemy in enemies:
        draw_enemy(enemy[0], enemy[1])

    # Vẽ tất cả đạn
    for bullet in bullets:
        draw_bullet(bullet[0], bullet[1])

    # Vẽ điểm số
    show_score(text_x, text_y)

    # Hiển thị Game Over nếu cần
    if is_game_over:
        game_over_text()

    # 4. Cập nhật màn hình hiển thị
    pygame.display.flip() # Hoặc pygame.display.update()

    # 5. Giới hạn FPS (Frames Per Second)
    clock.tick(60) # Giới hạn game chạy ở 60 FPS

# 6. Kết thúc Pygame khi vòng lặp kết thúc
pygame.quit()
