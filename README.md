# KEY-DOOR-GAME-
import pygame
import sys

pygame.init()

# Screen setup
screen = pygame.display.set_mode((600, 450))
pygame.display.set_caption("Key and Door Puzzle Game")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 28)
large_font = pygame.font.SysFont(None, 48)

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
DARK_GRAY = (50, 50, 50)
BLUE = (0, 0, 255)
YELLOW = (255, 215, 0)
BROWN = (139, 69, 19)
GREEN = (0, 128, 0)
RED = (200, 0, 0)
LIGHT_GRAY = (200, 200, 200)

# Load sound (key collection only)
key_collect_sound = pygame.mixer.Sound('C:/Users/vignan/Desktop/wav/key_collect.wav')  # Change path as needed
door_unlock_sound = pygame.mixer.Sound('C:/Users/vignan/Desktop/wav/door_unlock.wav')
interaction_sound = pygame.mixer.Sound('C:/Users/vignan/Desktop/wav/interact.wav')

# Load background images
backgrounds = [
    pygame.image.load('C:/Users/vignan/Desktop/wav/background1.jpg'),
    pygame.image.load('C:/Users/vignan/Desktop/wav/background2.jpg'),
]

# Player setup
player_radius = 15
player_pos = [50, 50]
player_speed = 8
player_color = BLUE

# Messages
interaction_message = ""
message_timer = 0

# Level data
levels = [
    {
        "objects": [
            {"name": "bookshelf", "rect": pygame.Rect(100, 100, 120, 80), "interactive": True, "has_key": True, "hint": ""},
            {"name": "drawer", "rect": pygame.Rect(350, 150, 100, 60), "interactive": True, "puzzle": {"solved": False, "code": "372", "attempt": ""}, "hint": "Try the drawer code!"},
            {"name": "painting", "rect": pygame.Rect(200, 300, 140, 90), "interactive": True, "hint": "Looks old and dusty."},
        ],
        "door": pygame.Rect(520, 180, 50, 90),
        "door_locked": True,
    },
    {
        "objects": [
            {"name": "bookshelf", "rect": pygame.Rect(120, 100, 120, 80), "interactive": True, "has_key": False, "hint": ""},
            {"name": "drawer", "rect": pygame.Rect(380, 140, 100, 60), "interactive": True, "puzzle": {"solved": False, "code": "845", "attempt": ""}, "hint": "The drawer needs a code."},
            {"name": "painting", "rect": pygame.Rect(250, 290, 140, 90), "interactive": True, "has_key": True, "hint": "Behind the painting?"},
        ],
        "door": pygame.Rect(520, 180, 50, 90),
        "door_locked": True,
    }
]

current_level = 0
has_key = False
input_active = False

def draw_player():
    pygame.draw.circle(screen, player_color, player_pos, player_radius)

def draw_objects(level):
    for obj in level["objects"]:
        rect = obj["rect"]
        color = BROWN if obj["name"] == "bookshelf" else (LIGHT_GRAY if obj["name"] == "drawer" else DARK_GRAY)
        pygame.draw.rect(screen, color, rect)
        if obj.get("interactive", False):
            pygame.draw.rect(screen, YELLOW, rect, 3)

def draw_door(level):
    door = level["door"]
    color = RED if level["door_locked"] else GREEN
    pygame.draw.rect(screen, color, door)
    pygame.draw.rect(screen, BLACK, door, 2)
    door_text = "Locked" if level["door_locked"] else "Unlocked"
    text = font.render(door_text, True, BLACK)
    screen.blit(text, (door.x + 5, door.y + door.height // 2 - 10))

def check_collision(rect):
    px, py = player_pos
    return rect.collidepoint(px, py)

def show_message(text, duration=120):
    global interaction_message, message_timer
    interaction_message = text
    message_timer = duration

def handle_drawer_input(puzzle):
    global input_active
    input_box = pygame.Rect(220, 350, 160, 40)
    pygame.draw.rect(screen, WHITE, input_box)
    pygame.draw.rect(screen, BLACK, input_box, 2)
    input_text = font.render("Code: " + puzzle["attempt"], True, BLACK)
    screen.blit(input_text, (input_box.x + 10, input_box.y + 10))

    for event in pygame.event.get():
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_BACKSPACE:
                puzzle["attempt"] = puzzle["attempt"][:-1]
            elif event.key == pygame.K_RETURN:
                if puzzle["attempt"] == puzzle["code"]:
                    puzzle["solved"] = True
                    input_active = False
                    show_message("Code correct! Drawer unlocked.", 180)
                else:
                    show_message("Wrong code! Try again.", 180)
                    puzzle["attempt"] = ""
            elif event.unicode.isdigit() and len(puzzle["attempt"]) < 5:
                puzzle["attempt"] += event.unicode
        elif event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

def reset_level():
    global player_pos, has_key, interaction_message, message_timer, input_active
    player_pos = [50, 50]
    has_key = False
    interaction_message = ""
    message_timer = 0
    input_active = False
    for obj in levels[current_level]["objects"]:
        if "puzzle" in obj:
            obj["puzzle"]["solved"] = False
            obj["puzzle"]["attempt"] = ""

def next_level():
    global current_level, has_key
    current_level += 1
    if current_level >= len(levels):
        return False
    reset_level()
    has_key = False
    return True

# Main game loop
running = True
win = False

while running:
    screen.blit(backgrounds[current_level], (0, 0))
    level = levels[current_level]

    draw_objects(level)
    draw_door(level)
    draw_player()

    if interaction_message:
        msg_surface = font.render(interaction_message, True, BLACK)
        screen.blit(msg_surface, (20, 410))
        message_timer -= 1
        if message_timer <= 0:
            interaction_message = ""

    if input_active:
        handle_drawer_input(level["objects"][1]["puzzle"])

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if event.type == pygame.KEYDOWN and not input_active:
            move_x = move_y = 0
            if event.key == pygame.K_LEFT: move_x = -player_speed
            elif event.key == pygame.K_RIGHT: move_x = player_speed
            elif event.key == pygame.K_UP: move_y = -player_speed
            elif event.key == pygame.K_DOWN: move_y = player_speed

            new_x, new_y = player_pos[0] + move_x, player_pos[1] + move_y
            if 0 + player_radius <= new_x <= 600 - player_radius:
                player_pos[0] = new_x
            if 0 + player_radius <= new_y <= 450 - player_radius:
                player_pos[1] = new_y

            for obj in level["objects"]:
                if check_collision(obj["rect"]):
                    if obj.get("interactive", False):
                        interaction_sound.play()
                        if obj.get("has_key", False) and not has_key:
                            has_key = True
                            key_collect_sound.play()
                            show_message("You found the key!", 180)
                        elif "puzzle" in obj and not obj["puzzle"]["solved"]:
                            input_active = True
                            show_message("Enter 3-digit code.", 180)
                        else:
                            show_message(obj.get("hint", "Just decoration."), 180)

            if check_collision(level["door"]):
                if level["door_locked"]:
                    if has_key:
                        level["door_locked"] = False
                        door_unlock_sound.play()
                        show_message("You unlocked the door!", 180)
                    else:
                        show_message("Find the key first!", 180)
                else:
                    win = True

    if win:
        win_text = large_font.render("You unlocked the door! Level Complete!", True, GREEN)
        screen.blit(win_text, (60, 200))
        pygame.display.update()
        pygame.time.delay(2000)
        win = False
        if not next_level():
            screen.fill(WHITE)
            end_text = large_font.render("Congratulations! All levels complete!", True, GREEN)
            screen.blit(end_text, (30, 200))
            pygame.display.update()
            pygame.time.delay(3000)
            running = False

    pygame.display.update()
    clock.tick(60)

pygame.quit()
