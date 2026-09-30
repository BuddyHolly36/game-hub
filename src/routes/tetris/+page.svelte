<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { Tetris } from 'tetris-engine';

	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D | null;
	let tetris: any = null;
	let score = $state(0);
	let level = $state(1);
	let lines = $state(0);
	let gameOver = $state(false);
	let paused = $state(false);
	let animationFrameId: number | null = null;
	let lastTime = 0;
	
	// Cell size in pixels
	const CELL_SIZE = 30;
	const BOARD_WIDTH = 10;
	const BOARD_HEIGHT = 20;
	
	// Colors for tetrominoes
	const COLORS = {
		I: '#00FFFF',
		J: '#0000FF',
		L: '#FF7F00',
		O: '#FFFF00',
		S: '#00FF00',
		T: '#800080',
		Z: '#FF0000',
		ghost: 'rgba(255, 255, 255, 0.3)'
	};

	function initCanvas() {
		if (canvas) {
			ctx = canvas.getContext('2d');
			if (ctx) {
				// Scale canvas to match display size
				const dpr = window.devicePixelRatio || 1;
				canvas.width = BOARD_WIDTH * CELL_SIZE * dpr;
				canvas.height = BOARD_HEIGHT * CELL_SIZE * dpr;
				canvas.style.width = `${BOARD_WIDTH * CELL_SIZE}px`;
				canvas.style.height = `${BOARD_HEIGHT * CELL_SIZE}px`;
				ctx.scale(dpr, dpr);
			}
		}
	}

	function drawBoard() {
		if (!ctx) return;
		
		// Clear canvas
		ctx.fillStyle = '#111';
		ctx.fillRect(0, 0, BOARD_WIDTH * CELL_SIZE, BOARD_HEIGHT * CELL_SIZE);
		
		// Draw grid
		ctx.strokeStyle = '#333';
		ctx.lineWidth = 1;
		for (let row = 0; row <= BOARD_HEIGHT; row++) {
			ctx.beginPath();
			ctx.moveTo(0, row * CELL_SIZE);
			ctx.lineTo(BOARD_WIDTH * CELL_SIZE, row * CELL_SIZE);
			ctx.stroke();
		}
		for (let col = 0; col <= BOARD_WIDTH; col++) {
			ctx.beginPath();
			ctx.moveTo(col * CELL_SIZE, 0);
			ctx.lineTo(col * CELL_SIZE, BOARD_HEIGHT * CELL_SIZE);
			ctx.stroke();
		}
	}

	function drawMatrix(matrix: any[], offsetX = 0, offsetY = 0, isGhost = false) {
		if (!ctx) return;
		
		const color = isGhost ? COLORS.ghost : COLORS[matrix.type] || '#FFF';
		
		matrix.matrix.forEach((row: any[], y: number) => {
			row.forEach((value: number, x: number) => {
				if (value) {
					const cellX = (offsetX + x) * CELL_SIZE;
					const cellY = (offsetY + y) * CELL_SIZE;
					
					// Draw cell
					ctx.fillStyle = color;
					ctx.fillRect(cellX + 1, cellY + 1, CELL_SIZE - 2, CELL_SIZE - 2);
					
					// Draw highlight
					ctx.fillStyle = isGhost ? 'rgba(255, 255, 255, 0.1)' : 'rgba(255, 255, 255, 0.3)';
					ctx.fillRect(cellX + 2, cellY + 2, CELL_SIZE - 4, CELL_SIZE - 4);
				}
			});
		});
	}

	function draw() {
		if (!tetris || !ctx) return;
		
		drawBoard();
		
		// Draw locked pieces (the board state)
		if (tetris.board) {
			tetris.board.forEach((row: any[], y: number) => {
				row.forEach((cell: any, x: number) => {
					if (cell) {
						drawMatrix({ type: cell.type, matrix: [[cell.value]] }, x, y);
					}
				});
			});
		}
		
		// Draw current piece
		if (tetris.piece && !gameOver) {
			drawMatrix(tetris.piece, tetris.pieceX, tetris.pieceY);
			
			// Draw ghost piece
			const ghostY = tetris.drop();
			drawMatrix(tetris.piece, tetris.pieceX, ghostY, true);
		}
	}

	function gameLoop(timestamp: number) {
		if (paused || gameOver) {
			animationFrameId = requestAnimationFrame(gameLoop);
			return;
		}
		
		const deltaTime = timestamp - lastTime;
		lastTime = timestamp;
		
		// Only move down at intervals based on level
		const dropInterval = 1000 / level;
		if (deltaTime >= dropInterval && tetris) {
			tetris.step();
			if (tetris.gameOver) {
				gameOver = true;
			}
			lastTime = 0;
		}
		
		draw();
		animationFrameId = requestAnimationFrame(gameLoop);
	}

	function handleKeyDown(e: KeyboardEvent) {
		if (gameOver || paused || !tetris) return;
		
		switch (e.key) {
			case 'ArrowLeft':
				tetris.moveLeft();
				break;
			case 'ArrowRight':
				tetris.moveRight();
				break;
			case 'ArrowDown':
				tetris.moveDown();
				break;
			case 'ArrowUp':
				tetris.rotate();
				break;
			case ' ':
				// Space: hard drop
				tetris.drop();
				break;
			case 'p':
				paused = !paused;
				if (!paused) {
					lastTime = performance.now();
					requestAnimationFrame(gameLoop);
				}
				break;
		}
		
		if (tetris.gameOver) {
			gameOver = true;
		}
	}

	function newGame() {
		if (tetris) {
			tetris.reset();
		} else {
			tetris = new Tetris();
			
			// Override step to update score
			const originalStep = tetris.step;
			tetris.step = function() {
				const result = originalStep.call(this);
				if (result && result.lines) {
					const linesCleared = result.lines;
					lines += linesCleared;
					
					// Score calculation (Tetris scoring)
					const linePoints = [0, 100, 300, 500, 800]; // 0, 1, 2, 3, 4 lines
					score += linePoints[linesCleared] * level;
					
					// Level up every 10 lines
					level = Math.floor(lines / 10) + 1;
				}
				
				if (this.gameOver) {
					gameOver = true;
				}
				
				saveGame();
				return result;
			};
		}
		
		score = 0;
		level = 1;
		lines = 0;
		gameOver = false;
		paused = false;
		
		initCanvas();
		draw();
		
		// Start game loop
		if (animationFrameId) {
			cancelAnimationFrame(animationFrameId);
		}
		lastTime = 0;
		animationFrameId = requestAnimationFrame(gameLoop);
		
		// Focus canvas for keyboard input
		canvas.focus();
	}

	function saveGame() {
		if (!tetris) return;
		
		const savedGame = {
			board: tetris.board,
			currentPiece: tetris.piece,
			currentPieceX: tetris.pieceX,
			currentPieceY: tetris.pieceY,
			score,
			level,
			lines,
			gameOver
		};
		
		localStorage.setItem('tetrisGame', JSON.stringify(savedGame));
	}

	function loadGame() {
		const saved = localStorage.getItem('tetrisGame');
		if (saved) {
			const savedGame = JSON.parse(saved);
			// For now, just create a new game - full state restoration is complex
			// We'll just start a new game but could implement full restore
			newGame();
		}
	}

	onMount(() => {
		initCanvas();
		newGame();
		window.addEventListener('keydown', handleKeyDown);
		loadGame();
	});

	onDestroy(() => {
		if (animationFrameId) {
			cancelAnimationFrame(animationFrameId);
		}
		window.removeEventListener('keydown', handleKeyDown);
	});

	function pauseGame() {
		paused = !paused;
		if (!paused) {
			lastTime = performance.now();
			if (animationFrameId) {
				cancelAnimationFrame(animationFrameId);
			}
			animationFrameId = requestAnimationFrame(gameLoop);
		}
	}
</script>

<h1>Tetris</h1>

<div class="tetris-container">
	<div class="game-area">
		<canvas 
			bind:this={canvas}
			tabindex="0"
			style="background: #111; display: block;"
		></canvas>
		
		<div class="controls-info">
			<p><strong>Controls:</strong></p>
			<p>← → : Move left/right</p>
			<p>↑ : Rotate</p>
			<p>↓ : Move down</p>
			<p>Space : Hard drop</p>
			<p>P : Pause</p>
		</div>
	</div>
	
	<div class="game-stats">
		<div class="stat">
			<h3>Score</h3>
			<p>{score}</p>
		</div>
		<div class="stat">
			<h3>Level</h3>
			<p>{level}</p>
		</div>
		<div class="stat">
			<h3>Lines</h3>
			<p>{lines}</p>
		</div>
		<div class="stat">
			<h3>Status</h3>
			<p>{gameOver ? 'GAME OVER' : paused ? 'PAUSED' : 'Playing'}</p>
		</div>
		
		<div class="buttons">
			<button onclick={newGame}>New Game</button>
			<button onclick={pauseGame}>{paused ? 'Resume' : 'Pause'}</button>
		</div>
	</div>
</div>

<style>
	.tetris-container {
		display: flex;
		gap: 2rem;
		flex-wrap: wrap;
		justify-content: center;
	}
	
	.game-area {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
	}
	
	canvas {
		border: 2px solid #333;
		border-radius: 4px;
		cursor: pointer;
		outline: none;
	}
	
	.controls-info {
		background: #222;
		color: white;
		padding: 1rem;
		border-radius: 8px;
		font-size: 0.9rem;
		line-height: 1.6;
	}
	
	.game-stats {
		min-width: 200px;
		padding: 1rem;
		background: #f9f9f9;
		border-radius: 8px;
		display: flex;
		flex-direction: column;
		gap: 1rem;
	}
	
	.stat {
		background: white;
		padding: 0.75rem;
		border-radius: 4px;
		border: 1px solid #ddd;
		text-align: center;
	}
	
	.stat h3 {
		margin: 0 0 0.25rem 0;
		font-size: 0.9rem;
		color: #666;
	}
	
	.stat p {
		margin: 0;
		font-size: 1.5rem;
		font-weight: bold;
	}
	
	.buttons {
		display: flex;
		gap: 1rem;
		margin-top: 1rem;
		justify-content: center;
	}
	
	.buttons button {
		padding: 0.75rem 1.5rem;
		border: none;
		background: #4CAF50;
		color: white;
		border-radius: 4px;
		cursor: pointer;
		font-size: 1rem;
		transition: background-color 0.3s;
	}
	
	.buttons button:hover {
		background: #45a049;
	}
	
	@media (max-width: 768px) {
		.tetris-container {
			flex-direction: column;
			align-items: center;
		}
		
		.game-stats {
			width: 100%;
			max-width: 400px;
		}
	}
</style>
