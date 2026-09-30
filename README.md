# Game Hub - Chess & Tetris

A personal project featuring Chess and Tetris games built with SvelteKit and TypeScript.

## Features

- **Chess**: Play chess against yourself with standard rules, move history, and undo functionality
- **Tetris**: Classic Tetris game with score tracking, level progression, and pause/resume
- **Persistence**: Both games save your progress to localStorage
- **Responsive**: Works on desktop browsers (mobile support coming soon)

## Tech Stack

- **Framework**: SvelteKit 2.x with TypeScript
- **Chess Logic**: [chess.js](https://github.com/jhlywa/chess.js)
- **Tetris Logic**: [tetris-engine](https://github.com/petelinmn/tetris-engine)
- **Styling**: Custom CSS with no external frameworks
- **Deployment**: Configured for GitHub Pages with static adapter

## Project Structure

```
src/
├── routes/
│   ├── +layout.svelte      # Shared layout with navigation
│   ├── +page.svelte        # Homepage with game selection
│   ├── chess/
│   │   ├── +page.svelte    # Chess game
│   │   └── +page.ts        # Prerender config
│   └── tetris/
│       ├── +page.svelte    # Tetris game
│       └── +page.ts        # Prerender config
```

## Setup

1. Clone the repository:
   ```bash
   git clone <your-repo-url>
   cd game-hub
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure the base path (if needed):
   - The default `paths.base` in `vite.config.ts` is set to `/game-hub`
   - This works automatically for GitHub Pages project sites (e.g., `username.github.io/game-hub/`)
   - If you're using a custom domain or different deployment, update the `paths.base` accordingly

4. Run the development server:
   ```bash
   npm run dev
   ```

5. Open [http://localhost:5173](http://localhost:5173)

## Build for GitHub Pages

1. Build the static site:
   ```bash
   npm run build
   ```

2. The output is in the `build` directory

3. Deploy to GitHub Pages:
   - Push to the `main` or `gh-pages` branch
   - Enable GitHub Pages in your repository settings
   - Select the branch and `/` (root) or `/docs` folder (if you move the build output)

## Game Controls

### Chess
- Click on a piece to select it
- Click on a destination square to move
- **New Game**: Start a new game
- **Undo**: Undo the last move

### Tetris
- **← →**: Move left/right
- **↑**: Rotate piece
- **↓**: Move down faster
- **Space**: Hard drop (instant drop)
- **P**: Pause/resume
- **New Game**: Start a new game
- **Pause/Resume**: Toggle pause

## LocalStorage

Both games automatically save your progress:
- **Chess**: Saves the current game state (FEN notation)
- **Tetris**: Saves the game state (board, score, level, lines)

To clear saved games, use your browser's clear storage option or manually remove the items from localStorage.

## License

MIT
