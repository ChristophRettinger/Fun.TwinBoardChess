# PRD: Twin Board Chess

## Problem Statement

A chess player wants a novel variant that forces simultaneous consistency across two independent games. Standard chess offers no constraint linking multiple concurrent games — the player wants a mode where every move must be legally playable on two separate boards at once, creating a unique strategic challenge where material divergence between boards is the central tension.

## Solution

A single-page web application hosting two simultaneous chess games against the same bot. The player must select a move that is legal on both boards (same origin square → same destination square). The UI enforces this by restricting selectable moves to the intersection of legal moves across both boards. Winning requires checkmating the bot on both boards; losing happens the moment no shared legal move exists.

## User Stories

1. As a player, I want to see two chess boards side by side, so that I can compare the state of both games at a glance.
2. As a player, I want to interact with only the left board (board 1) to make moves, so that I have a single clear input point.
3. As a player, I want only moves legal on both boards to be selectable on board 1, so that I never accidentally make an illegal cross-board move.
4. As a player, I want my chosen move to be applied to both boards simultaneously, so that the constraint is enforced automatically.
5. As a player, I want the bot to respond on each board independently after my move, so that the two board states diverge naturally over time.
6. As a player, I want the bot to respond with a short delay (300–500ms) after my move, so that the interaction feels responsive rather than mechanical.
7. As a player, I want to choose between a random-move bot and Stockfish before starting a game, so that I can control the difficulty.
8. As a player using Stockfish, I want to set the skill level via a slider (0–5), so that I can tune how bad the bot plays.
9. As a player, I want to choose to play as white or black before starting a game, so that I control which side I play.
10. As a player, I want the board to flip when I play as black, so that my pieces are always at the bottom of each board.
11. As a player, I want all pre-game settings (bot type, skill level, color) to be locked once the game starts, so that the game rules stay consistent.
12. As a player, I want to see a move counter showing the current move number, so that I have a minimal sense of game progress without visual clutter.
13. As a player, I want to see a full-victory banner when I checkmate the bot on both boards simultaneously, so that I know I achieved the best outcome.
14. As a player, I want to see a partial-victory banner when I checkmate the bot on one board while the other is still in play, so that I know I've earned a mid-game win.
15. As a player, I want the won board to freeze on its final checkmate position after a partial victory, so that I can see my achievement while continuing play.
16. As a player, I want board 2 to become the active control board after a partial victory, so that I can continue the surviving game with full move freedom.
17. As a player, I want the surviving board to have no move intersection constraint after a partial victory, so that I can play it as a normal chess game.
18. As a player, I want to see a loss banner when the intersection of legal moves across both boards becomes empty, so that I understand why the game ended.
19. As a player, I want to see a loss banner immediately if either board reaches stalemate, so that I know stalemate is a losing condition in this variant.
20. As a player, I want a "New Game" button always visible, so that I can restart at any point without waiting for a loss banner.
21. As a player, I want a "New Game" button in the result banner, so that restarting after a completed game is one click away.
22. As a player, I want an optional highlight toggle (off by default), so that I can enable move-origin highlighting when I want help seeing my options.
23. As a player with the highlight toggle enabled, I want the pieces on board 1 that have at least one move in the intersection to be visually marked, so that I can quickly identify actionable pieces.
24. As a player, I want Stockfish's WASM binary to only load if I've selected the Stockfish bot, so that the page loads quickly by default.
25. As a player, I want the page to work by opening a single HTML file with no build step, so that setup is trivial.

## Implementation Decisions

### Core Modules

**GameState (deep module)**
Encapsulates two independent `chess.js` instances. Exposes:
- `getLegalMovesIntersection()` → array of `{from, to}` move objects valid on both boards simultaneously
- `applyPlayerMove(from, to)` → applies the move to both boards, returns updated state
- `getStatus()` → returns one of: `playing`, `full-win`, `partial-win-board1`, `partial-win-board2`, `loss-no-moves`, `loss-stalemate`
- `getBotMoves()` → returns the bot's chosen move for each board independently

After a partial victory, `GameState` tracks which board is "free" and removes the intersection constraint for that board.

**BotEngine (deep module)**
Abstracts random vs. Stockfish behind a single interface:
- `getMove(fen)` → Promise resolving to `{from, to}`
- Implementations: `RandomBot` (pure random legal move) and `StockfishBot` (WASM, skill 0–5)
- `StockfishBot` lazy-loads the WASM binary only on first call

**BoardUI**
Wraps two `cm-chessboard` instances. Responsibilities:
- Render both board positions from FEN strings
- Show legal destination squares on board 1 when a piece is selected (restricted to intersection)
- Mark origin squares of intersection-movable pieces on board 1 when highlight toggle is on (using cm-chessboard markers extension)
- Freeze a board (disable interaction, preserve position) on partial victory
- Flip both boards when player plays black

**GameController**
Orchestrates `GameState`, `BotEngine`, and `BoardUI`. Handles:
- Player move selection → validation → `GameState.applyPlayerMove` → board render → bot delay timer → bot moves → board render → status check → UI update
- Game end detection and banner display
- New Game reset (re-initializes all modules with current settings)

### Key Decisions

- **Move identity**: a move is uniquely identified by `(from_square, to_square)`. Piece type is implicit — if the piece on the origin square differs between boards (due to promotion or divergence), the move is still valid as long as a legal move exists from that square to that destination on both boards.
- **Stalemate**: stalemate on either board is an immediate full loss. There is no per-board draw outcome.
- **Partial victory trigger**: the first checkmate on either board triggers partial victory immediately. The player does not need to deliver checkmate on both in the same move.
- **Post-partial-victory**: the surviving board becomes a fully unconstrained normal chess game. The `GameState` intersection logic is bypassed; the surviving board's `chess.js` instance is used directly.
- **Bot independence**: each board gets its own bot call with its own FEN. The bot has no awareness of the other board.
- **Bot timing**: a fixed 400ms `setTimeout` is applied before the bot move is computed and applied, regardless of bot type.
- **Stockfish skill**: mapped to Stockfish's `Skill Level` UCI option, range 0–5. Level 0 is the weakest available setting.
- **Color selection**: when the player selects black, both boards are oriented with black at the bottom and the bot (white) makes the first move on both boards immediately after game start.
- **Highlight toggle**: stored as a boolean flag in `GameController`. When toggled on, `BoardUI` recomputes and renders intersection move origins immediately on the current position.
- **Tech stack**: chess.js (game logic), cm-chessboard (rendering + markers), Stockfish WASM (optional bot) — all loaded via CDN. Single `index.html`, no build step.

## Testing Decisions

Good tests for this project verify **external behavior** — what the game rules produce given a board state — not internal implementation details like which array methods are used or how modules call each other.

**What makes a good test here:**
- Set up a specific FEN position on both boards, trigger a game action, assert the resulting game status or available move set
- Do not assert on internal state of `chess.js` instances or DOM structure
- Tests should be runnable in a browser console or a lightweight test runner (e.g., a standalone test HTML file) with no build step

**Modules to test:**

- **`GameState`** — highest priority; pure logic, no DOM dependency
  - Intersection of legal moves returns only moves valid on both boards
  - Empty intersection correctly returns `loss-no-moves` status
  - Stalemate on board 1 or board 2 returns `loss-stalemate`
  - Checkmate on one board transitions status to `partial-win-boardN`
  - Checkmate on both boards returns `full-win`
  - After partial victory, the free board accepts any legal move without intersection filtering

- **`BotEngine`** — test `RandomBot` only (deterministic to test)
  - `getMove(fen)` always returns a move that is legal in the given FEN
  - `getMove(fen)` never returns a move when no legal moves exist (edge: passed a terminal FEN)

- **`BoardUI`** — not unit tested; verify visually via manual play

- **`GameController`** — not unit tested; behavior is covered by `GameState` + manual integration

## Out of Scope

- Mobile / responsive layout (desktop-only)
- Move history or game notation (move counter only)
- Undo / take-back functionality
- Saving or loading game state
- Multiplayer or remote play
- Clock / time controls
- Opening book for the bot
- Piece themes or board color customization
- Accessibility features (screen reader support, keyboard navigation)
- Analysis mode or position setup

## Further Notes

- The "Martin" Lichess bot (random legal moves) is the conceptual inspiration for the random bot. The implementation is a direct random selection from `chess.js`'s `moves()` output — no Lichess API involved.
- The intersection constraint is the entire challenge. The bot's quality is almost irrelevant; even a random bot will win frequently once the boards diverge enough that the player's intersection becomes empty.
- cm-chessboard's `MarkersExtension` is the recommended mechanism for the optional piece highlighting. It supports custom marker classes which can be styled with CSS.
- Stockfish WASM can be loaded from a public CDN (e.g., `lichess.org`'s hosted build or `npmjs.com` CDN). Confirm CDN availability and CORS headers before implementation.
