# ♠ Scoundrel — A Solo Card Dungeon

A polished, mobile-first **portfolio implementation** of the solo dungeon-crawl card game **Scoundrel**. Built with vanilla HTML, CSS, and JavaScript. No framework, build step, network connection, or server is required.

## Quick start

1. Download or clone the project.
2. Open **`index.html`** directly in a modern browser (Chrome, Edge, Firefox, Safari).
3. Play! Your progress saves automatically in the same browser via `localStorage`.

> Important: Keep the directory structure intact; `index.html` loads `styles.css`, `js/game.js`, `js/storage.js`, and `js/app.js` using relative paths and **classic scripts**, so it also works from a local `file://` URL. No `npm install`, bundler, or localhost server is needed.

## Game overview

You enter the dungeon with **20 HP**. Survive **44 cards** by choosing the right order of fights, weapon upgrades, and potions. You win when **every card is resolved**; you lose as soon as health reaches 0.

### The dungeon deck

Start from a standard 52-card deck (without jokers) and remove **all red J, Q, K, and A** (♥/♦). Shuffle the remaining **44 cards**:

| Type | Cards | Count | Role |
| --- | --- | ---: | --- |
| Monsters | ♣/♠ 2–10, J (11), Q (12), K (13), A (14) | 26 | Fight or take damage |
| Weapons | ♦ 2–10 | 9 | Reduce monster damage |
| Potions | ♥ 2–10 | 9 | Restore HP (up to 20) |

### Rooms

- Reveal enough cards to have a **4-card room**.
- **Face the room:** resolve any **3 cards**, in any order. The untouched 4th card becomes the first card of the next room.
- **Avoid the room:** *before* selecting a card, put all 4 visible cards at the bottom of the draw pile **in their existing order**. You cannot avoid twice consecutively.
- **Endgame edge case:** When fewer than 4 cards remain, the final room contains 1–3 cards and you must resolve **all of them**. An empty draw pile does **not** instantly win the game if unresolved room cards are still visible.

### Card effects

**♦ Weapon** — Equip it immediately, discarding any old weapon *and its stacked monsters*. The new weapon has no previous defeated-monster restriction.

**♥ Potion** — Heal its value, capped at 20 HP. Only the **first potion selected in a room** works. Any additional potion is discarded without healing. A potion selected at 20 HP is used even if it restores 0 HP.

**♣/♠ Monster** — Choose a combat method in the encounter dialog:

- **Bare-handed:** Take damage equal to the monster's full value. This is always allowed, even while carrying a weapon.
- **Use your weapon (if allowed):** Take `max(0, monsterValue - weaponValue)` damage.
- A **0-damage weapon victory** stacks that monster on the weapon and updates its *last defeated monster* value. Future weapon targets must have value **≤** that last defeated value; the sequence is non-increasing.
- A weapon fight that deals **positive damage** discards the monster, **without updating the last-defeated limit**. This follows the rules stated for this portfolio implementation.

**Worked weapon example:** Equip ♦8. Defeat ♠7 for 0 damage: the last defeated value becomes **7**. Defeat ♣5: the limit becomes **5**. You cannot use that weapon against ♠9, but you *can* still fight it bare-handed. Swapping to a new weapon resets the limit.

**Worked potion example:** Select ♥4 at 15 HP → 19 HP. Select ♥8 in the *same room* → 0 additional healing; the second potion is discarded. Your potion allowance resets in the next room.

### Scoring

- **Win:** score = remaining HP (from 1 to 20).
- **Loss:** score = `−(sum of values of all unresolved monsters)`. Unresolved monsters in **both the draw pile and the visible room** count. Monsters already resolved or stacked on a weapon do not count.
- If you lose on the very last monster and none remain unresolved, the specified formula produces a score of **0** (even though the result is a loss).

## Controls and accessibility

- **Click / tap** a visible card to resolve it (monsters open a combat-choice dialog).
- **Avoid this room** before choosing a card, if the previous room was not avoided.
- **New game** creates a new shuffled dungeon. **Restart** replays the **same original deck order** from full health.
- **How to play** explains all rules and examples.
- **Keyboard:** Tab through interactive controls; Enter/Space activate buttons; arrow keys, Home and End move between focused room cards. H or ? opens rules; A avoids (when allowed); N starts a new game; R restarts (when focus is not inside an input or button). Escape closes rules/confirm/combat dialogs.
- Semantic `<button>` cards, visible focus rings, descriptive ARIA labels, a `role="log"` live action history, a labeled health progress bar, native accessible `<dialog>` modals, and `prefers-reduced-motion` support.
- The log is scrollable. Colors, spacing, and animations are configured in `styles.css`.

## File responsibilities

```text
scoundrel-portfolio/
├── index.html          # App layout, HUD, card room, guide and modals
├── styles.css          # Responsive theme, card appearance, transitions, focus states
├── js/
│   ├── game.js         # All deck, scoring, turn, combat and potion rules
│   ├── storage.js      # Versioned, validated localStorage save/load
│   └── app.js          # DOM rendering, inputs, dialog logic and keyboard behavior
├── tests/
│   └── test-game.js    # Optional engine regression tests (Node.js built-ins)
└── README.md           # This document
```

`game.js` exposes `window.ScoundrelEngine`; `storage.js` exposes `window.ScoundrelStorage`. This **classic-script setup** intentionally avoids ES module CORS restrictions when opening `index.html` directly from the filesystem.

## Developer tools and reproducibility

Open the **Developer tools** disclosure at the bottom to:

1. Enter a custom seed (e.g. `portfolio-demo`) and start a reproducible dungeon.
2. Inspect the current draw-pile order, top to bottom.
3. See remaining room cards and state checks.

Leave the seed field empty for a normal random shuffle. A seeded shuffle uses Fisher–Yates with a small deterministic PRNG. Game state, the initial shuffled deck, turn position, and action log are saved to `localStorage`. Refresh the page to resume. If storage is blocked or invalid, the app simply starts a new game.

### Optional regression tests

The game itself needs **no development tools**. If you happen to have Node.js installed, test the pure rules engine with:

```bash
node tests/test-game.js
```

Tests use only Node.js built-in modules; they do not install packages or start a server.

## Portfolio notes

- Original implementation and UI treatment for an educational/portfolio project.
- Uses built-in CSS, Unicode suit symbols, and local system fonts: **no third-party assets or dependencies**.
- Suggested portfolio improvements: sound effects with a mute toggle, session statistics, optional daily seeded challenge, and browser-based automated UI testing.
