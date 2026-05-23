Building an LLM-powered chess application inside a single HTML file is an excellent exercise in frontend architecture. To make this robust, performant, and ready for Gemini Canvas, we need to strictly separate our concerns—State, UI, and API logic—while keeping them unified in one document.

Here is the comprehensive architectural plan, designed with a professional, minimalist approach, ensuring the final output is classy, functional, and highly polished.

## 1. Core Architecture & Tech Stack

Since the constraint is a single HTML file without a bundler, we will rely on vanilla JavaScript (ES6+), semantic HTML5, and CSS3, leveraging lightweight CDNs for the complex chess logic.

* **Game Engine (`chess.js`):** We will use the `chess.js` library via CDN to handle move validation, check/checkmate detection, FEN/PGN generation, and the material points table. Writing a chess engine from scratch inside a single file is a recipe for spaghetti code; `chess.js` is the industry standard.
* **UI Rendering:** Custom DOM manipulation for the chessboard. Relying on a pre-built UI library makes custom themes and animations difficult. We will build a CSS Grid-based board.
* **API Integration:** Native `fetch()` API to communicate with the Google Gemini 2.5 Flash endpoint.

## 2. UI/UX & Theming Strategy

The interface should follow an "io philosophy"—communicating state and options seamlessly without cluttered text or overwhelming UI elements.

### Layout Details

* **Hamburger Menu (Top Left):** Houses the game mode toggle ("Play vs LLM" / "Play Offline Friend"), Theme Switcher, and API Key input (saved to `localStorage`).
* **Main Stage:** The chessboard, centered and responsive.
* **Heads-Up Display (HUD):**
* **Top/Bottom:** Player names, captured material advantage (Points Table), and Timers.
* **Controls (Below Board):** Start, Reset, Undo Move, Give Up.
* **Difficulty Slider:** Appears only in LLM mode (adjusts the LLM's system prompt or temperature).



### Theme Specifications

A clean data-attribute approach (`<body data-theme="dark">`) will control CSS variables for seamless switching.

| Theme | Visual Philosophy | Color Palette & Typography |
| --- | --- | --- |
| **Dark Mode** | Minimalistic and classy. Easy on the eyes for long sessions. | Background: `rgb(16, 24, 39)`. Board: Muted slate and soft charcoal. Minimalist SVG pieces. |
| **Terminal Edition** | Developer-focused, raw, and functional. | Monospace font exclusively. Pure black background, neon green or amber accents. ASCII-style piece representations. |
| **Classic** | Traditional, familiar, weighted. | Soft beige and warm brown board squares. Standard weighted-style piece icons. |
| **Futuristic** | Cyberpunk/Sci-Fi aesthetics. | Glassmorphism panels, glowing neon accents for legal moves, sharp sans-serif fonts. |

### Interaction & Animations

* **Piece Selection:** When a piece is clicked, a subtle CSS transform (scale) indicates selection.
* **Legal Moves:** `chess.js` calculates legal moves; the UI maps these to the grid and injects a soft, pulsing radial dot into the center of all valid destination squares.
* **Move Execution:** Smooth CSS transitions for piece translation from square A to square B.

## 3. State Management & Game Mechanics

The app will use a centralized state object to manage the flow, preventing desynchronization between the visual board and the hidden logic.

* **Game Modes:** `STATE.mode = 'llm' | 'pvp'`.
* *PvP:* The board simply flips or allows both white and black moves locally.
* *LLM:* The user plays White (or selects color). After the user's move, the board locks, and the loading state triggers.


* **Timer System:** A `setInterval` loop that deducts time based on whose turn `chess.js` says it is. Pauses during LLM network requests.
* **Points Table (Material Evaluation):** A helper function that parses the current board state array, assigns standard weights (P=1, N=3, B=3, R=5, Q=9), and displays the differential (e.g., "+2" next to the winning side).

## 4. LLM Integration Strategy (Gemini 2.5 Flash)

LLMs are notoriously tricky with chess because they don't inherently "see" the board—they predict text.

1. **State Passing:** On the LLM's turn, the app generates the current FEN (Forsyth-Edwards Notation) string using `chess.js`.
2. **Prompt Engineering:** The prompt must explicitly provide the FEN, the list of legal moves, and the difficulty constraint.
* *Example Prompt:* "You are a chess engine playing Black. The current board FEN is `[FEN]`. The legal moves are `[Moves]`. Respond ONLY with your chosen move in Standard Algebraic Notation (SAN)."


3. **Difficulty Handling:**
* *Easy:* High temperature (more random/suboptimal moves).
* *Hard:* Low temperature, plus a system prompt instructing it to play optimally.


4. **Hallucination Fallback:** If the LLM returns an invalid move, the code must catch the error, quietly re-prompt the LLM up to 3 times, and if it still fails, pick a random legal move to keep the game from freezing.

---

## 5. Implementation Roadmap for Canvas

When you are ready to feed this to Gemini Canvas, do not ask for the whole file at once. The context window and output limits will cause it to truncate or rush the code. Use this sequence of prompts to build the single file iteratively:

1. **Initialize the Skeleton & Engine:** Prompt 1.
"Create a single HTML file. Include the CDN for `chess.js` in the head. Build the CSS Grid layout for an 8x8 chessboard and the basic UI shell (Hamburger menu, HUD for timers/points, control buttons). Write the initial JavaScript to render standard SVG chess pieces onto the grid based on the starting position. Do not implement game logic yet."


2. **Implement Theming & UI States:** Prompt 2.
"Add the CSS variables for four themes: Classic, Futuristic, Dark Mode (using rgb(16, 24, 39) as the background), and Terminal Edition (using strict monospace fonts). Implement the JavaScript logic in the hamburger menu to switch between these themes and toggle between 'Play LLM' and 'Play Friend'."


3. **Add Chess Logic & Animations:** Prompt 3.
"Connect the UI to `chess.js`. Allow the user to drag-and-drop or click-to-move pieces. When a piece is clicked, query `chess.js` for legal moves and display a pulsing animation on those destination squares. Implement the logic for the Reset, Undo, and Give Up buttons."


4. **Build Timers & Material Score:** Prompt 4.
"Implement the chess clock system (e.g., 10 minutes per side). Hook it into the turn state. Next, write a function that calculates the material difference based on the current pieces on the board and displays it in the HUD (e.g., White +3)."


5. **Integrate Gemini 2.5 Flash API:** Prompt 5.
"Add the fetch logic to call the Gemini REST API. Create a settings input for the user to provide their API key. When it's the computer's turn, pass the current FEN string to Gemini, parse its response, validate the move, and execute it on the board. Add a retry mechanism if the LLM hallucinates an illegal move."
