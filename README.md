# Twin Board Chess

A browser-based chess variant where every move must be legal on two independent boards at once.

## The Game

You play the same side on two chess boards simultaneously against the same bot. The boards start from the same position but diverge as the bot responds independently on each.

**The constraint**: your move must be legal on both boards at once. Only squares reachable from your selected piece on *both* boards are highlighted. If no such move exists, you lose.

**The goal**: checkmate the bot on both boards simultaneously. That is the true victory.

**Partial victory**: checkmating the bot on one board freezes it. The surviving board continues as unconstrained normal chess — but this is a lesser outcome, not the real win.

**Losing**: no shared legal move exists across both boards; stalemate on either board; the bot checkmates you on the surviving board after a partial victory.

## Setup

No build step. Open `index.html` directly in a browser — `file://` works fine.

## Controls

- **Difficulty** — a single slider from Random through Novice 1–5 to Stockfish 0–5. Locked once the game starts.
- **Color** — choose white or black before starting.
- **Rules** — in-game overlay explaining the rules in full.
- **Highlights** — toggle to mark pieces that have at least one move in the intersection.
- **Piece set** — four options (Staunty, CBurnett, Merida, Unicode).

## How It Was Built

This project was built entirely through [Claude Code](https://claude.ai/code) using [Matt Pocock's Claude Code skills](https://github.com/mattpocock/skills).

The workflow:

1. `/grill-me` — relentless design interview to resolve every branch of the decision tree before writing a line of code
2. `/to-prd` — synthesize the conversation into a structured PRD, published as [GitHub issue #1](https://github.com/ChristophRettinger/Fun.TwinBoardChess/issues/1)
3. `/to-issues` — break the PRD into independently-grabbable vertical-slice issues
4. `implement issue #N` — implement each issue one at a time in a focused session

The process repeated for each significant feature. Every commit in this repository corresponds to a completed issue. No code was written without a prior design session.

## Tech Stack

- [chess.js](https://github.com/jhlywa/chess.js) — move generation and game state
- [Stockfish 16](https://stockfishchess.org/) — WASM chess engine loaded from CDN
- Piece sets from [lichess](https://lichess.org/) CDN
- Vanilla JavaScript, no framework, no build step
