# 🐍 Python Snake Game — Arcade & Browser Edition

> **Play instantly in your browser:** [👉 Launch Live Game on GitHub Pages](https://sudhamondal.github.io/Python-Snake-Game/)

---

## 🎮 Welcome, Player

Step into a neon-lit, retro-inspired take on the timeless arcade classic—now rebuilt twice: once as a crisp **Python & Pygame** desktop game (`snake.py`) and once as a zero-install **HTML5 Canvas & JavaScript** web experience (`index.html`). Whether you are chasing a personal high score, battling a friend shoulder-to-shoulder on one keyboard, or testing your reflexes against a pathfinding AI bot, every round ramps up the speed and stakes.

---

## 💡 Why I Built It

Most beginner Snake tutorials stop at a green square eating a red square inside an empty box. I wanted to push a classic, single-file game much further—exploring how **game loops, pathfinding algorithms, collision edge cases, and procedural audio synthesis** work under the hood without relying on external `.wav`, `.mp3`, or sprite files. Building both the Python desktop version and the browser port also made it easy for anyone visiting this repository to play immediately with a single click.

---

## 🛠️ Technologies Used

* **Python 3.10+ & Pygame (`snake.py`):** Core desktop game engine, grid rendering, keyboard event buffering, and `pygame.mixer` audio synthesis.
* **HTML5 Canvas, CSS3 & Vanilla JavaScript (`index.html`):** Full browser port powered by `requestAnimationFrame`, Web Audio API oscillators, and `localStorage` score persistence.
* **Math, Array & Collections (`deque`):** Used for bounded breadth-first search (BFS) flood-fill lookahead and raw PCM waveform generation.
* **GitHub Pages:** Hosts the live playable web version directly from the repository root.

---

## ⚙️ Core Features & Mechanics

* **Three Game Modes (`G`):**
  * **1P Solo:** Classic high-score survival with automatic best-score saving per difficulty (`highscore.txt` / `localStorage`).
  * **2P Local Multiplayer:** Two snakes share the same arena (`WASD` vs `Arrow Keys`) with head-to-head and body collision detection.
  * **1P vs AI Bot:** Face off against a smart bot whose intelligence scales with your chosen difficulty.
* **Difficulty-Scaled AI Opponent:**
  * *Easy:* Purely greedy movement with occasional random turns so newcomers can outmaneuver it.
  * *Medium:* Chases golden apples, predicts Player 1's next move to dodge head-on crashes, and checks shallow escape routes.
  * *Hard:* Zero random blunders paired with deep flood-fill lookahead to avoid trapping itself inside tight coils.
* **Screen-Wrap Portal Mode (`T`):** Turns solid outer walls into glowing purple portals—slither off the left edge and emerge seamlessly on the right.
* **Timed Bonus Golden Apples:** Randomly spawn after eating regular food, awarding **+30 points** if grabbed before the 6-second countdown bar runs out.
* **Optional Interior Wall Obstacles (`O`):** Randomly generated stone blocks with guaranteed spawn-corridor protection so you never crash on frame one.
* **Procedural 8-Bit Sound Effects (`M`):** Retro square-wave and sawtooth tones generated mathematically in memory—zero external audio files needed.

---

## 🕹️ What Users Can Do

* **Play right away in any browser** via GitHub Pages—no setup or downloads required.
* **Customize every match from the start menu** by mixing and matching difficulty (`Easy`, `Medium`, `Hard`), game mode, wall obstacles, and screen-wrap portals.
* **Challenge a friend locally** on a single keyboard or benchmark their skills against the `Hard` AI bot.
* **Download and hack on `snake.py` locally**—the entire game lives in one clean, readable Python file.

---

## ⌨️ Keyboard Shortcuts & Controls

| Key | Action |
| :--- | :--- |
| `W` `A` `S` `D` | **Player 1 Movement** *(also Arrow Keys in `1P Solo` & `1P vs AI Bot`)* |
| `Arrow Keys` | **Player 2 Movement** *(in `2P Local`)* / **Select Difficulty** *(in Menu)* |
| `G` | Cycle Mode: `1P Solo` / `2P Local` / `1P vs AI Bot` *(Menu)* |
| `O` | Toggle Interior Wall Obstacles *(Menu)* |
| `T` | Toggle Screen-Wrap Portals *(Menu)* |
| `M` | Mute / Unmute Synthesized Sound Effects |
| `P` | Pause / Resume Game *(automatically pauses golden apple timers)* |
| `Enter` / `Space` / `R` | Start or Restart Match |
| `Esc` | Return to Main Menu / Quit |

---

## 🔨 How I Built It

1. **Grid & Tick Architecture:** Modeled the arena on a `640×480` canvas divided into `20×20` cells, decoupling input capture from discrete movement ticks so fast double-taps never cause accidental 180-degree self-collisions.
2. **Toroidal Distance & BFS Flood-Fill AI:** Wrote a wrap-aware Manhattan distance function and a bounded Breadth-First Search (`collections.deque`) so the AI bot can measure reachable open space before committing to a turn.
3. **In-Memory Audio Synthesis:** Generated 16-bit signed PCM audio buffers using `math.sin` and fade-out envelopes, feeding them directly into `pygame.mixer.Sound` (and `AudioContext` oscillators in JS).
4. **Browser Port:** Translated the exact game loop, AI heuristics, and sound synth into a standalone `index.html` with clickable toolbar controls for effortless GitHub Pages hosting.

---

## 🧠 Learnings

* **Input Buffering Matters:** Without queuing `next_direction` separately from `direction`, pressing two orthogonal keys within a single tick lets a snake reverse into its own neck.
* **Greedy Pathfinding Isn't Enough:** An AI that only minimizes distance to food will eagerly dive into a closed U-turn. Adding even a 20-to-60-cell flood-fill check dramatically improves bot survival.
* **State-Aware Timers:** Pausing the game means shifting expiration timestamps (`golden_expires_at += pause_duration`) so bonus apples don't unfairly vanish while paused.

---

## 🚀 Future Expansion

* 📱 **Mobile Touch & Swipe Gestures:** Add swipe detection on the HTML5 canvas for smooth smartphone play.
* ⚡ **Power-Up Pickups:** Introduce temporary speed boosts, shield armor, or obstacle-breaching drills.
* 🏆 **Global Leaderboard & Replay Ghost:** Save and race against a ghost replay of your personal best run.
* 🎨 **Unlockable Snake Skins & Themes:** Add retro Game Boy, Cyberpunk, and Matrix color palettes.

---

## 💻 How to Run the Game

### Option 1: Play Online (No Installation)
Open **[https://sudhamondal.github.io/Python-Snake-Game/](https://sudhamondal.github.io/Python-Snake-Game/)** in any modern web browser.

### Option 2: Run the Python Version Locally
1. **Clone the repository:**
   ```bash
   git clone [https://github.com/SudhaMondal/Python-Snake-Game.git](https://github.com/SudhaMondal/Python-Snake-Game.git)
   cd Python-Snake-Game
   ```

2. **Install Pygame:**
   ```bash
   pip install pygame
   ```

3. **Run `snake.py`:**
   ```bash
   python snake.py
   ```

---

## 📄 License

Distributed under the [MIT License](LICENSE). Feel free to fork, remix, and build upon it!
