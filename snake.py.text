import array
import math
import os
import random
import sys
from collections import deque
import pygame

# --- Configuration ---
WINDOW_WIDTH = 640
WINDOW_HEIGHT = 480
CELL_SIZE = 20

# Difficulty Settings: Speed, Obstacles, and AI Intelligence Tuning
DIFFICULTIES = {
    "Easy": {
        "fps": 9,
        "speed_step": 0.15,
        "obstacles": 6,
        "ai_blunder": 0.28,
        "ai_flood_limit": 0,
        "ai_predict_p1": False,
        "ai_chase_gold": False,
    },
    "Medium": {
        "fps": 13,
        "speed_step": 0.25,
        "obstacles": 14,
        "ai_blunder": 0.08,
        "ai_flood_limit": 20,
        "ai_predict_p1": True,
        "ai_chase_gold": True,
    },
    "Hard": {
        "fps": 18,
        "speed_step": 0.40,
        "obstacles": 24,
        "ai_blunder": 0.0,
        "ai_flood_limit": 60,
        "ai_predict_p1": True,
        "ai_chase_gold": True,
    },
}
DIFFICULTY_ORDER = ["Easy", "Medium", "Hard"]
GAME_MODES = ["1P Solo", "2P Local (WASD vs Arrows)", "1P vs AI Bot"]

# Golden Apple Settings
GOLDEN_POINTS = 30
GOLDEN_DURATION_MS = 6000  # 6 seconds to grab before despawning
GOLDEN_SPAWN_CHANCE = 0.35

# Colors (R, G, B)
BG_COLOR = (18, 18, 24)
GRID_COLOR = (28, 28, 38)
PORTAL_BORDER_COLOR = (168, 85, 247)
P1_HEAD = (52, 211, 153)
P1_BODY = (16, 185, 129)
P2_HEAD = (56, 189, 248)
P2_BODY = (2, 132, 199)
FOOD_COLOR = (244, 63, 94)
GOLDEN_COLOR = (250, 204, 21)
OBSTACLE_COLOR = (100, 116, 139)
TEXT_COLOR = (243, 244, 246)
SUBTEXT_COLOR = (156, 163, 175)

HIGHSCORE_FILE = "highscore.txt"


# --- Procedural Sound Synthesizer (Zero External Files) ---
class SoundFX:
    def __init__(self) -> None:
        self.enabled = True
        self.available = False
        try:
            pygame.mixer.init(frequency=44100, size=-16, channels=1, buffer=512)
            self.eat_sound = self._synth_tone(520, 880, 0.09, "square")
            self.golden_sound = self._synth_tone(660, 1320, 0.18, "square")
            self.crash_sound = self._synth_tone(220, 55, 0.35, "saw")
            self.menu_sound = self._synth_tone(440, 440, 0.05, "square")
            self.available = True
        except pygame.error:
            self.available = False

    def _synth_tone(
        self, freq_start: float, freq_end: float, duration: float, wave: str = "square"
    ) -> pygame.mixer.Sound:
        sample_rate = 44100
        n_samples = int(sample_rate * duration)
        buf = array.array("h")
        phase = 0.0

        for i in range(n_samples):
            t = i / n_samples
            freq = freq_start + (freq_end - freq_start) * t
            phase += (2.0 * math.pi * freq) / sample_rate

            if wave == "square":
                val = 1.0 if math.sin(phase) >= 0 else -1.0
            else:
                val = 2.0 * ((phase / (2.0 * math.pi)) % 1.0) - 1.0

            envelope = (1.0 - t) ** 1.5
            sample = int(val * envelope * 4500)
            buf.append(max(-32768, min(32767, sample)))

        return pygame.mixer.Sound(buffer=buf)

    def play(self, sound_name: str) -> None:
        if self.available and self.enabled:
            sound = getattr(self, f"{sound_name}_sound", None)
            if sound:
                sound.play()


# --- Persistence ---
def load_high_scores() -> dict[str, int]:
    scores = {d: 0 for d in DIFFICULTY_ORDER}
    if os.path.exists(HIGHSCORE_FILE):
        try:
            with open(HIGHSCORE_FILE, "r", encoding="utf-8") as f:
                for line in f:
                    if ":" in line:
                        diff, val = line.strip().split(":", 1)
                        if diff in scores:
                            scores[diff] = int(val)
        except (ValueError, OSError):
            pass
    return scores


def save_high_scores(scores: dict[str, int]) -> None:
    try:
        with open(HIGHSCORE_FILE, "w", encoding="utf-8") as f:
            for diff, val in scores.items():
                f.write(f"{diff}:{val}\n")
    except OSError:
        pass


# --- Board & AI Helpers ---
def get_random_free_cell(occupied: set[tuple[int, int]]) -> tuple[int, int]:
    cols = WINDOW_WIDTH // CELL_SIZE
    rows = WINDOW_HEIGHT // CELL_SIZE
    while True:
        pos = (
            random.randint(0, cols - 1) * CELL_SIZE,
            random.randint(0, rows - 1) * CELL_SIZE,
        )
        if pos not in occupied:
            return pos


def generate_obstacles(
    count: int, safe_zone: set[tuple[int, int]]
) -> list[tuple[int, int]]:
    obstacles: list[tuple[int, int]] = []
    occupied = set(safe_zone)
    for _ in range(count):
        cell = get_random_free_cell(occupied)
        obstacles.append(cell)
        occupied.add(cell)
    return obstacles


def grid_distance(
    a: tuple[int, int], b: tuple[int, int], wrap_enabled: bool
) -> int:
    dx = abs(a[0] - b[0])
    dy = abs(a[1] - b[1])
    if wrap_enabled:
        dx = min(dx, WINDOW_WIDTH - dx)
        dy = min(dy, WINDOW_HEIGHT - dy)
    return (dx + dy) // CELL_SIZE


def count_reachable_space(
    start: tuple[int, int],
    blocked: set[tuple[int, int]],
    wrap_enabled: bool,
    limit: int = 45,
) -> int:
    visited = {start}
    queue = deque([start])
    while queue and len(visited) < limit:
        cx, cy = queue.popleft()
        for dx, dy in (
            (0, -CELL_SIZE),
            (0, CELL_SIZE),
            (-CELL_SIZE, 0),
            (CELL_SIZE, 0),
        ):
            nx, ny = cx + dx, cy + dy
            if wrap_enabled:
                nx %= WINDOW_WIDTH
                ny %= WINDOW_HEIGHT
            elif not (0 <= nx < WINDOW_WIDTH and 0 <= ny < WINDOW_HEIGHT):
                continue

            nxt = (nx, ny)
            if nxt not in blocked and nxt not in visited:
                visited.add(nxt)
                queue.append(nxt)
    return len(visited)


def get_ai_direction(
    ai_snake: list[tuple[int, int]],
    current_dir: tuple[int, int],
    p1_snake: list[tuple[int, int]],
    p1_next_dir: tuple[int, int],
    obstacles: list[tuple[int, int]],
    food: tuple[int, int],
    golden_food: tuple[int, int] | None,
    wrap_enabled: bool,
    diff_cfg: dict,
) -> tuple[int, int]:
    head_x, head_y = ai_snake[0]
    opposite = (-current_dir[0], -current_dir[1])
    candidates = [
        d
        for d in ((0, -CELL_SIZE), (0, CELL_SIZE), (-CELL_SIZE, 0), (CELL_SIZE, 0))
        if d != opposite
    ]

    # Treat tail tips as open since they advance unless a snake just ate
    blocked = set(obstacles) | set(ai_snake[:-1]) | set(p1_snake[:-1])

    # Collect immediate collision-free moves
    valid_moves: list[tuple[tuple[int, int], tuple[int, int]]] = []
    for dx, dy in candidates:
        nx, ny = head_x + dx, head_y + dy
        if wrap_enabled:
            nx %= WINDOW_WIDTH
            ny %= WINDOW_HEIGHT
        elif not (0 <= nx < WINDOW_WIDTH and 0 <= ny < WINDOW_HEIGHT):
            continue

        nxt = (nx, ny)
        if nxt not in blocked:
            valid_moves.append(((dx, dy), nxt))

    if not valid_moves:
        return current_dir

    # Easy/Medium bots occasionally make a random safe turn instead of optimal play
    if random.random() < diff_cfg["ai_blunder"]:
        return random.choice(valid_moves)[0]

    # Predict Player 1's next cell on Medium/Hard so the AI avoids head-on collisions
    p1_pred: tuple[int, int] | None = None
    if diff_cfg["ai_predict_p1"]:
        p1_pred_x = p1_snake[0][0] + p1_next_dir[0]
        p1_pred_y = p1_snake[0][1] + p1_next_dir[1]
        if wrap_enabled:
            p1_pred = (p1_pred_x % WINDOW_WIDTH, p1_pred_y % WINDOW_HEIGHT)
        else:
            p1_pred = (p1_pred_x, p1_pred_y)

    # Target golden apple on Medium/Hard when within reasonable range
    target = food
    if diff_cfg["ai_chase_gold"] and golden_food is not None:
        dist_gold = grid_distance(ai_snake[0], golden_food, wrap_enabled)
        dist_food = grid_distance(ai_snake[0], food, wrap_enabled)
        if dist_gold <= dist_food + 8:
            target = golden_food

    flood_limit: int = diff_cfg["ai_flood_limit"]
    scored_moves: list[tuple[int, tuple[int, int]]] = []

    for move_dir, nxt in valid_moves:
        dist = grid_distance(nxt, target, wrap_enabled)
        score = -dist * 10

        # Bounded flood-fill lookahead scales with difficulty (0 on Easy, 20 Medium, 60 Hard)
        if flood_limit > 0:
            open_cells = count_reachable_space(
                nxt, blocked, wrap_enabled, limit=flood_limit
            )
            min_safe = min(len(ai_snake), flood_limit)
            if open_cells < min_safe:
                score -= 500  # Penalize dead-end traps
            else:
                score += open_cells

        if p1_pred is not None and nxt == p1_pred:
            score -= 350

        scored_moves.append((score, move_dir))

    scored_moves.sort(key=lambda item: item[0], reverse=True)
    return scored_moves[0][1]


def draw_grid(surface: pygame.Surface, wrap_enabled: bool = False) -> None:
    for x in range(0, WINDOW_WIDTH, CELL_SIZE):
        pygame.draw.line(surface, GRID_COLOR, (x, 0), (x, WINDOW_HEIGHT))
    for y in range(0, WINDOW_HEIGHT, CELL_SIZE):
        pygame.draw.line(surface, GRID_COLOR, (0, y), (WINDOW_WIDTH, y))

    if wrap_enabled:
        dash = CELL_SIZE
        for x in range(0, WINDOW_WIDTH, dash * 2):
            pygame.draw.line(surface, PORTAL_BORDER_COLOR, (x, 0), (x + dash, 0), 3)
            pygame.draw.line(
                surface,
                PORTAL_BORDER_COLOR,
                (x, WINDOW_HEIGHT - 2),
                (x + dash, WINDOW_HEIGHT - 2),
                3,
            )
        for y in range(0, WINDOW_HEIGHT, dash * 2):
            pygame.draw.line(surface, PORTAL_BORDER_COLOR, (0, y), (0, y + dash), 3)
            pygame.draw.line(
                surface,
                PORTAL_BORDER_COLOR,
                (WINDOW_WIDTH - 2, y),
                (WINDOW_WIDTH - 2, y + dash),
                3,
            )


def main() -> None:
    pygame.init()
    sfx = SoundFX()
    screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
    pygame.display.set_caption("Snake")
    clock = pygame.time.Clock()

    font_main = pygame.font.SysFont("consolas", 22, bold=True)
    font_large = pygame.font.SysFont("consolas", 38, bold=True)
    font_small = pygame.font.SysFont("consolas", 16)

    high_scores = load_high_scores()

    # Menu / Mode Settings
    state = "MENU"  # "MENU", "PLAYING", "PAUSED", "GAME_OVER"
    diff_idx = 1  # Default: Medium
    mode_idx = 0  # 0 = 1P Solo, 1 = 2P Local, 2 = 1P vs AI Bot
    obstacles_enabled = True
    wrap_enabled = False

    # Game Entities
    p1_snake: list[tuple[int, int]] = []
    p1_dir = (CELL_SIZE, 0)
    p1_next_dir = (CELL_SIZE, 0)
    p1_score = 0

    p2_snake: list[tuple[int, int]] = []
    p2_dir = (-CELL_SIZE, 0)
    p2_next_dir = (-CELL_SIZE, 0)
    p2_score = 0

    obstacles: list[tuple[int, int]] = []
    food = (0, 0)
    golden_food: tuple[int, int] | None = None
    golden_expires_at = 0
    pause_started_at = 0

    current_fps = 12.0
    outcome_text = ""

    def start_game() -> None:
        nonlocal p1_snake, p1_dir, p1_next_dir, p1_score
        nonlocal p2_snake, p2_dir, p2_next_dir, p2_score
        nonlocal obstacles, food, golden_food, golden_expires_at
        nonlocal current_fps, state, outcome_text

        diff_name = DIFFICULTY_ORDER[diff_idx]
        cfg = DIFFICULTIES[diff_name]
        two_snake_mode = mode_idx in (1, 2)
        safe_zone: set[tuple[int, int]] = set()

        if two_snake_mode:
            p1_x = (WINDOW_WIDTH // 4 // CELL_SIZE) * CELL_SIZE
            p1_y = (WINDOW_HEIGHT // 3 // CELL_SIZE) * CELL_SIZE
            p1_snake = [
                (p1_x, p1_y),
                (p1_x - CELL_SIZE, p1_y),
                (p1_x - 2 * CELL_SIZE, p1_y),
            ]
            p1_dir = (CELL_SIZE, 0)
            p1_next_dir = (CELL_SIZE, 0)

            p2_x = ((WINDOW_WIDTH * 3) // 4 // CELL_SIZE) * CELL_SIZE
            p2_y = ((WINDOW_HEIGHT * 2) // 3 // CELL_SIZE) * CELL_SIZE
            p2_snake = [
                (p2_x, p2_y),
                (p2_x + CELL_SIZE, p2_y),
                (p2_x + 2 * CELL_SIZE, p2_y),
            ]
            p2_dir = (-CELL_SIZE, 0)
            p2_next_dir = (-CELL_SIZE, 0)

            safe_zone.update(p1_snake)
            safe_zone.update(p2_snake)
            for step in range(1, 7):
                safe_zone.add(((p1_x + step * CELL_SIZE) % WINDOW_WIDTH, p1_y))
                safe_zone.add(((p2_x - step * CELL_SIZE) % WINDOW_WIDTH, p2_y))
        else:
            start_x = (WINDOW_WIDTH // 2 // CELL_SIZE) * CELL_SIZE
            start_y = (WINDOW_HEIGHT // 2 // CELL_SIZE) * CELL_SIZE
            p1_snake = [
                (start_x, start_y),
                (start_x - CELL_SIZE, start_y),
                (start_x - 2 * CELL_SIZE, start_y),
            ]
            p1_dir = (CELL_SIZE, 0)
            p1_next_dir = (CELL_SIZE, 0)
            p2_snake = []

            safe_zone.update(p1_snake)
            for step in range(1, 7):
                safe_zone.add(((start_x + step * CELL_SIZE) % WINDOW_WIDTH, start_y))

        p1_score = 0
        p2_score = 0
        obstacles = (
            generate_obstacles(cfg["obstacles"], safe_zone) if obstacles_enabled else []
        )
        occupied = set(p1_snake) | set(p2_snake) | set(obstacles)
        food = get_random_free_cell(occupied)
        golden_food = None
        golden_expires_at = 0
        current_fps = float(cfg["fps"])
        outcome_text = ""
        state = "PLAYING"

    while True:
        now = pygame.time.get_ticks()
        diff_name = DIFFICULTY_ORDER[diff_idx]
        cfg = DIFFICULTIES[diff_name]
        two_snake_mode = mode_idx in (1, 2)
        ai_enabled = mode_idx == 2

        # --- Event Handling ---
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    if state in ("PLAYING", "PAUSED", "GAME_OVER"):
                        state = "MENU"
                        sfx.play("menu")
                    else:
                        pygame.quit()
                        sys.exit()

                elif event.key == pygame.K_m:
                    sfx.enabled = not sfx.enabled
                    sfx.play("menu")

                elif state == "MENU":
                    if event.key in (pygame.K_LEFT, pygame.K_a):
                        diff_idx = (diff_idx - 1) % len(DIFFICULTY_ORDER)
                        sfx.play("menu")
                    elif event.key in (pygame.K_RIGHT, pygame.K_d):
                        diff_idx = (diff_idx + 1) % len(DIFFICULTY_ORDER)
                        sfx.play("menu")
                    elif event.key == pygame.K_g:
                        mode_idx = (mode_idx + 1) % len(GAME_MODES)
                        sfx.play("menu")
                    elif event.key == pygame.K_o:
                        obstacles_enabled = not obstacles_enabled
                        sfx.play("menu")
                    elif event.key == pygame.K_t:
                        wrap_enabled = not wrap_enabled
                        sfx.play("menu")
                    elif event.key in (pygame.K_RETURN, pygame.K_SPACE):
                        sfx.play("menu")
                        start_game()

                elif state == "PAUSED":
                    if event.key == pygame.K_p:
                        if golden_food is not None:
                            golden_expires_at += now - pause_started_at
                        state = "PLAYING"
                        sfx.play("menu")

                elif state == "GAME_OVER":
                    if event.key in (pygame.K_r, pygame.K_SPACE, pygame.K_RETURN):
                        sfx.play("menu")
                        start_game()

                elif state == "PLAYING":
                    if event.key == pygame.K_p:
                        pause_started_at = now
                        state = "PAUSED"
                        sfx.play("menu")

                    # Player 1 Controls (WASD only in 2P Local; WASD + Arrows in 1P or vs AI)
                    use_arrows_p1 = mode_idx != 1
                    p1_up = (pygame.K_w, pygame.K_UP) if use_arrows_p1 else (pygame.K_w,)
                    p1_down = (
                        (pygame.K_s, pygame.K_DOWN) if use_arrows_p1 else (pygame.K_s,)
                    )
                    p1_left = (
                        (pygame.K_a, pygame.K_LEFT) if use_arrows_p1 else (pygame.K_a,)
                    )
                    p1_right = (
                        (pygame.K_d, pygame.K_RIGHT)
                        if use_arrows_p1
                        else (pygame.K_d,)
                    )

                    if event.key in p1_up and p1_dir != (0, CELL_SIZE):
                        p1_next_dir = (0, -CELL_SIZE)
                    elif event.key in p1_down and p1_dir != (0, -CELL_SIZE):
                        p1_next_dir = (0, CELL_SIZE)
                    elif event.key in p1_left and p1_dir != (CELL_SIZE, 0):
                        p1_next_dir = (-CELL_SIZE, 0)
                    elif event.key in p1_right and p1_dir != (-CELL_SIZE, 0):
                        p1_next_dir = (CELL_SIZE, 0)

                    # Player 2 Controls (Human only when mode_idx == 1)
                    if mode_idx == 1:
                        if event.key == pygame.K_UP and p2_dir != (0, CELL_SIZE):
                            p2_next_dir = (0, -CELL_SIZE)
                        elif event.key == pygame.K_DOWN and p2_dir != (0, -CELL_SIZE):
                            p2_next_dir = (0, CELL_SIZE)
                        elif event.key == pygame.K_LEFT and p2_dir != (CELL_SIZE, 0):
                            p2_next_dir = (-CELL_SIZE, 0)
                        elif event.key == pygame.K_RIGHT and p2_dir != (-CELL_SIZE, 0):
                            p2_next_dir = (CELL_SIZE, 0)

        # --- Game Logic ---
        if state == "PLAYING":
            if golden_food is not None and now >= golden_expires_at:
                golden_food = None

            p1_dir = p1_next_dir
            raw_p1_x = p1_snake[0][0] + p1_dir[0]
            raw_p1_y = p1_snake[0][1] + p1_dir[1]
            p1_head = (
                (raw_p1_x % WINDOW_WIDTH, raw_p1_y % WINDOW_HEIGHT)
                if wrap_enabled
                else (raw_p1_x, raw_p1_y)
            )

            if two_snake_mode:
                if ai_enabled:
                    p2_next_dir = get_ai_direction(
                        p2_snake,
                        p2_dir,
                        p1_snake,
                        p1_dir,
                        obstacles,
                        food,
                        golden_food,
                        wrap_enabled,
                        cfg,
                    )
                p2_dir = p2_next_dir
                raw_p2_x = p2_snake[0][0] + p2_dir[0]
                raw_p2_y = p2_snake[0][1] + p2_dir[1]
                p2_head = (
                    (raw_p2_x % WINDOW_WIDTH, raw_p2_y % WINDOW_HEIGHT)
                    if wrap_enabled
                    else (raw_p2_x, raw_p2_y)
                )
            else:
                p2_head = (-999, -999)

            def hits_hazard(head: tuple[int, int]) -> bool:
                out_of_bounds = not wrap_enabled and (
                    head[0] < 0
                    or head[0] >= WINDOW_WIDTH
                    or head[1] < 0
                    or head[1] >= WINDOW_HEIGHT
                )
                return out_of_bounds or (head in obstacles)

            p1_dead = (
                hits_hazard(p1_head)
                or (p1_head in p1_snake)
                or (two_snake_mode and p1_head in p2_snake)
            )
            p2_dead = two_snake_mode and (
                hits_hazard(p2_head)
                or (p2_head in p2_snake)
                or (p2_head in p1_snake)
            )

            if two_snake_mode and p1_head == p2_head:
                p1_dead = True
                p2_dead = True

            if p1_dead or p2_dead:
                state = "GAME_OVER"
                sfx.play("crash")
                if not two_snake_mode:
                    outcome_text = "GAME OVER"
                    if p1_score > high_scores[diff_name]:
                        high_scores[diff_name] = p1_score
                        save_high_scores(high_scores)
                else:
                    p2_winner_label = (
                        f"AI ({diff_name}) WINS!" if ai_enabled else "PLAYER 2 WINS!"
                    )
                    if p1_dead and p2_dead:
                        outcome_text = "DRAW!"
                    elif p2_dead:
                        outcome_text = "PLAYER 1 WINS!"
                    else:
                        outcome_text = p2_winner_label
            else:
                ate_regular = False
                p1_snake.insert(0, p1_head)
                if p1_head == food:
                    p1_score += 10
                    ate_regular = True
                else:
                    p1_snake.pop()

                if p1_head == golden_food:
                    p1_score += GOLDEN_POINTS
                    golden_food = None
                    sfx.play("golden")

                if two_snake_mode:
                    p2_snake.insert(0, p2_head)
                    if p2_head == food:
                        p2_score += 10
                        ate_regular = True
                    else:
                        p2_snake.pop()

                    if p2_head == golden_food:
                        p2_score += GOLDEN_POINTS
                        golden_food = None
                        sfx.play("golden")

                if ate_regular:
                    current_fps += cfg["speed_step"]
                    sfx.play("eat")
                    occupied = set(p1_snake) | set(p2_snake) | set(obstacles)
                    if golden_food:
                        occupied.add(golden_food)
                    food = get_random_free_cell(occupied)

                    if golden_food is None and random.random() < GOLDEN_SPAWN_CHANCE:
                        occupied.add(food)
                        golden_food = get_random_free_cell(occupied)
                        golden_expires_at = now + GOLDEN_DURATION_MS

        # --- Rendering ---
        screen.fill(BG_COLOR)
        draw_grid(screen, wrap_enabled=wrap_enabled and state != "MENU")

        if state == "MENU":
            title = font_large.render("PYTHON SNAKE", True, P1_HEAD)
            screen.blit(title, title.get_rect(center=(WINDOW_WIDTH // 2, 70)))

            mode_label = GAME_MODES[mode_idx]
            mode_text = font_main.render(
                f"[G] Mode: {mode_label}",
                True,
                P2_HEAD if two_snake_mode else TEXT_COLOR,
            )
            screen.blit(mode_text, mode_text.get_rect(center=(WINDOW_WIDTH // 2, 145)))

            diff_text = font_main.render(
                f"Difficulty: < {diff_name} >", True, GOLDEN_COLOR
            )
            screen.blit(diff_text, diff_text.get_rect(center=(WINDOW_WIDTH // 2, 190)))

            obs_status = "ON" if obstacles_enabled else "OFF"
            obs_text = font_main.render(
                f"[O] Wall Obstacles: {obs_status}", True, TEXT_COLOR
            )
            screen.blit(obs_text, obs_text.get_rect(center=(WINDOW_WIDTH // 2, 235)))

            wrap_status = "ON (Portals)" if wrap_enabled else "OFF (Solid Walls)"
            wrap_text = font_main.render(
                f"[T] Screen Wrap: {wrap_status}",
                True,
                PORTAL_BORDER_COLOR if wrap_enabled else TEXT_COLOR,
            )
            screen.blit(wrap_text, wrap_text.get_rect(center=(WINDOW_WIDTH // 2, 280)))

            snd_status = "ON" if sfx.enabled else "MUTED"
            snd_text = font_small.render(
                f"[M] Sound FX: {snd_status}", True, SUBTEXT_COLOR
            )
            screen.blit(snd_text, snd_text.get_rect(center=(WINDOW_WIDTH // 2, 325)))

            if not two_snake_mode:
                best_text = font_small.render(
                    f"Best ({diff_name}): {high_scores[diff_name]}", True, SUBTEXT_COLOR
                )
                screen.blit(
                    best_text, best_text.get_rect(center=(WINDOW_WIDTH // 2, 355))
                )

            start_hint = font_main.render(
                "Press ENTER or SPACE to Start", True, P1_BODY
            )
            screen.blit(
                start_hint, start_hint.get_rect(center=(WINDOW_WIDTH // 2, 415))
            )

        else:
            # Draw obstacles
            for obs in obstacles:
                obs_rect = pygame.Rect(
                    obs[0] + 1, obs[1] + 1, CELL_SIZE - 2, CELL_SIZE - 2
                )
                pygame.draw.rect(screen, OBSTACLE_COLOR, obs_rect, border_radius=3)

            # Draw regular food
            food_rect = pygame.Rect(
                food[0] + 2, food[1] + 2, CELL_SIZE - 4, CELL_SIZE - 4
            )
            pygame.draw.rect(screen, FOOD_COLOR, food_rect, border_radius=6)

            # Draw timed golden apple + countdown bar
            if golden_food is not None:
                remaining_ratio = max(
                    0.0, (golden_expires_at - now) / GOLDEN_DURATION_MS
                )
                pulse = 1 if int(now / 150) % 2 == 0 else 2
                g_rect = pygame.Rect(
                    golden_food[0] + pulse,
                    golden_food[1] + pulse,
                    CELL_SIZE - pulse * 2,
                    CELL_SIZE - pulse * 2,
                )
                pygame.draw.rect(screen, GOLDEN_COLOR, g_rect, border_radius=8)

                bar_w = 140
                bar_h = 8
                bar_x = (WINDOW_WIDTH - bar_w) // 2
                bar_y = 14
                pygame.draw.rect(
                    screen, GRID_COLOR, (bar_x, bar_y, bar_w, bar_h), border_radius=4
                )
                pygame.draw.rect(
                    screen,
                    GOLDEN_COLOR,
                    (bar_x, bar_y, int(bar_w * remaining_ratio), bar_h),
                    border_radius=4,
                )

            # Draw Player 1 snake
            for i, segment in enumerate(p1_snake):
                color = P1_HEAD if i == 0 else P1_BODY
                rect = pygame.Rect(
                    segment[0] + 1, segment[1] + 1, CELL_SIZE - 2, CELL_SIZE - 2
                )
                pygame.draw.rect(screen, color, rect, border_radius=4)

            # Draw Player 2 / AI snake
            if two_snake_mode:
                for i, segment in enumerate(p2_snake):
                    color = P2_HEAD if i == 0 else P2_BODY
                    rect = pygame.Rect(
                        segment[0] + 1, segment[1] + 1, CELL_SIZE - 2, CELL_SIZE - 2
                    )
                    pygame.draw.rect(screen, color, rect, border_radius=4)

            # HUD
            if two_snake_mode:
                p2_tag = f"AI ({diff_name})" if ai_enabled else "P2"
                p1_surf = font_main.render(f"P1: {p1_score}", True, P1_HEAD)
                p2_surf = font_main.render(f"{p2_tag}: {p2_score}", True, P2_HEAD)
                screen.blit(p1_surf, (14, 10))
                screen.blit(p2_surf, (WINDOW_WIDTH - p2_surf.get_width() - 14, 10))
            else:
                score_surf = font_main.render(f"Score: {p1_score}", True, TEXT_COLOR)
                high_surf = font_small.render(
                    f"Best ({diff_name}): {high_scores[diff_name]}", True, SUBTEXT_COLOR
                )
                speed_surf = font_small.render(
                    f"Speed: {current_fps:.1f}x", True, SUBTEXT_COLOR
                )
                screen.blit(score_surf, (14, 10))
                screen.blit(high_surf, (14, 36))
                screen.blit(speed_surf, (WINDOW_WIDTH - speed_surf.get_width() - 14, 14))

            if state == "PAUSED":
                overlay = font_large.render("PAUSED", True, TEXT_COLOR)
                screen.blit(
                    overlay,
                    overlay.get_rect(center=(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2)),
                )

            elif state == "GAME_OVER":
                banner_color = GOLDEN_COLOR if two_snake_mode else FOOD_COLOR
                over_surf = font_large.render(outcome_text, True, banner_color)
                retry_surf = font_small.render(
                    "Press R / SPACE to Retry  |  ESC for Menu", True, TEXT_COLOR
                )
                screen.blit(
                    over_surf,
                    over_surf.get_rect(
                        center=(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2 - 22)
                    ),
                )
                screen.blit(
                    retry_surf,
                    retry_surf.get_rect(
                        center=(WINDOW_WIDTH // 2, WINDOW_HEIGHT // 2 + 24)
                    ),
                )

        pygame.display.flip()
        clock.tick(current_fps if state == "PLAYING" else 30)


if __name__ == "__main__":
    main()
