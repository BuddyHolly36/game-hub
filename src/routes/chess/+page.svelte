<script lang="ts">
	import { Chess } from 'chess.js';
	import { onMount } from 'svelte';

	let game = new Chess();
	let board: string[][] = [];
	let status = $state('White to move');
	let selectedSquare = $state<string | null>(null);
	let gameHistory = $state<string[]>([]);
	
	function getSquareClass(row: number, col: number, pieceSquare: string) {
		const isLight = (row + col) % 2 === 0;
		const isSelected = selectedSquare === pieceSquare;
		return `square ${isLight ? 'light' : 'dark'}${isSelected ? ' selected' : ''}`;
	}
	
	function getPieceSymbol(piece: string): string {
		const pieceType = piece.split('_')[0];
		const color = piece.split('_')[1];
		
		const symbols: Record<string, Record<string, string>> = {
			king: { w: '♔', b: '♚' },
			queen: { w: '♕', b: '♛' },
			rook: { w: '♖', b: '♜' },
			bishop: { w: '♗', b: '♝' },
			knight: { w: '♘', b: '♞' },
			pawn: { w: '♙', b: '♟' }
		};
		
		return symbols[pieceType]?.[color] || piece;
	}

	// Initialize the board
	function initBoard() {
		const newBoard = [];
		for (let row = 0; row < 8; row++) {
			newBoard[row] = [];
			for (let col = 0; col < 8; col++) {
				const square = String.fromCharCode(97 + col) + (8 - row);
				const piece = game.get(square);
				newBoard[row][col] = piece ? piece.type + (piece.color === 'w' ? '_w' : '_b') : '';
			}
		}
		board = newBoard;
	}

	// Update game status
	function updateStatus() {
		if (game.isGameOver()) {
			if (game.isCheckmate()) {
				status = `Checkmate! ${game.turn() === 'w' ? 'Black' : 'White'} wins!`;
			} else if (game.isDraw()) {
				status = 'Game ended in a draw';
			} else {
				status = 'Game over';
			}
		} else if (game.isCheck()) {
			status = `${game.turn() === 'w' ? 'White' : 'Black'} is in check!`;
		} else {
			status = `${game.turn() === 'w' ? 'White' : 'Black'} to move`;
		}
	}

	// Handle square click
	function handleSquareClick(row: number, col: number) {
		const square = String.fromCharCode(97 + col) + (8 - row);
		
		if (selectedSquare) {
			try {
				const move = game.move({
					from: selectedSquare,
					to: square,
					promotion: 'q' // Always promote to queen for simplicity
				});
				
				if (move) {
					gameHistory = [...gameHistory, move.san];
					initBoard();
					updateStatus();
					saveGame();
				}
				selectedSquare = null;
			} catch (e) {
				console.error('Invalid move:', e);
				selectedSquare = null;
			}
		} else {
			const piece = game.get(square);
			if (piece && piece.color === game.turn()) {
				selectedSquare = square;
			}
		}
	}

	// Reset game
	function resetGame() {
		game = new Chess();
		initBoard();
		selectedSquare = null;
		gameHistory = [];
		status = 'White to move';
		localStorage.removeItem('chessGame');
	}

	// Save game to localStorage
	function saveGame() {
		localStorage.setItem('chessGame', game.fen());
	}

	// Load game from localStorage
	function loadGame() {
		const savedFen = localStorage.getItem('chessGame');
		if (savedFen) {
			game = new Chess(savedFen);
			initBoard();
			updateStatus();
			
			// Reconstruct history from moves
			const moves = game.history();
			if (moves.length > 0) {
				gameHistory = moves;
			}
		}
	}

	// Undo last move
	function undoMove() {
		game.undo();
		initBoard();
		updateStatus();
		if (gameHistory.length > 0) {
			gameHistory = gameHistory.slice(0, -1);
		}
		saveGame();
	}

	onMount(() => {
		initBoard();
		updateStatus();
		loadGame();
	});
</script>

<h1>Chess</h1>

<div class="chess-container">
	<div class="board-wrapper">
		<div class="board" style="grid-template-columns: repeat(8, 60px); grid-template-rows: repeat(8, 60px);">
			{#each board as row, rowIdx}
				{#each row as piece, colIdx}
					{@const square = String.fromCharCode(97 + colIdx) + (8 - rowIdx)}
					<button 
						class={getSquareClass(rowIdx, colIdx, square)}
						onclick={() => handleSquareClick(rowIdx, colIdx)}
						aria-label={square}
					>
						{#if piece}
							<span class="piece {piece}">{getPieceSymbol(piece)}</span>
						{/if}
					</button>
				{/each}
			{/each}
		</div>
		<div class="coordinates right">
			{#each Array(8) as _, i}
				<div>{8 - i}</div>
			{/each}
		</div>
		<div class="coordinates bottom">
			{#each Array(8) as _, i}
				<div>{String.fromCharCode(97 + i)}</div>
			{/each}
		</div>
	</div>
	
	<div class="game-info">
		<div class="status">{status}</div>
		<div class="controls">
			<button onclick={resetGame}>New Game</button>
			<button onclick={undoMove} disabled={gameHistory.length === 0}>Undo</button>
		</div>
		<div class="history">
			<h3>Moves</h3>
			<div class="moves-list">
				{#each gameHistory as move, i}
					<span>{i % 2 === 0 ? (i / 2 + 1) + '. ' : ''}{move} </span>
				{/each}
			</div>
		</div>
	</div>
</div>

<style>
	.chess-container {
		display: flex;
		gap: 2rem;
		flex-wrap: wrap;
		justify-content: center;
	}
	
	.board-wrapper {
		position: relative;
	}
	
	.board {
		display: grid;
		border: 2px solid #333;
		border-radius: 4px;
		overflow: hidden;
	}
	
	.square {
		margin: 0;
		padding: 0;
		border: none;
		cursor: pointer;
		position: relative;
		font-size: 2rem;
		transition: background-color 0.2s;
	}
	
	.square.light {
		background-color: #f0d9b5;
	}
	
	.square.dark {
		background-color: #b58863;
	}
	
	.square:hover {
		background-color: rgba(255, 255, 0, 0.3);
	}
	
	.square.selected {
		box-shadow: inset 0 0 0 3px #ffd700;
	}
	
	.piece {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 2.5rem;
		user-select: none;
	}
	
	.piece_king_w, .piece_queen_w, .piece_rook_w, .piece_bishop_w, .piece_knight_w, .piece_pawn_w {
		color: white;
		text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
	}
	
	.piece_king_b, .piece_queen_b, .piece_rook_b, .piece_bishop_b, .piece_knight_b, .piece_pawn_b {
		color: black;
		text-shadow: 1px 1px 2px rgba(255,255,255,0.5);
	}
	
	.coordinates {
		display: flex;
		flex-direction: column;
		position: absolute;
		font-size: 0.8rem;
		color: #666;
	}
	
	.coordinates.right {
		top: 0;
		left: 100%;
		margin-left: 5px;
		text-align: center;
	}
	
	.coordinates.bottom {
		bottom: -25px;
		left: 0;
		flex-direction: row;
		justify-content: space-around;
		width: 100%;
	}
	
	.game-info {
		min-width: 250px;
		padding: 1rem;
		background: #f9f9f9;
		border-radius: 8px;
	}
	
	.status {
		font-size: 1.2rem;
		font-weight: bold;
		margin-bottom: 1rem;
		text-align: center;
	}
	
	.controls {
		display: flex;
		gap: 1rem;
		margin-bottom: 1rem;
	}
	
	.controls button {
		padding: 0.5rem 1rem;
		border: none;
		background: #4CAF50;
		color: white;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.3s;
	}
	
	.controls button:hover:not(:disabled) {
		background: #45a049;
	}
	
	.controls button:disabled {
		background: #cccccc;
		cursor: not-allowed;
	}
	
	.history {
		margin-top: 1rem;
	}
	
	.history h3 {
		margin: 0 0 0.5rem 0;
		font-size: 1rem;
	}
	
	.moves-list {
		font-size: 0.9rem;
		max-height: 300px;
		overflow-y: auto;
		padding: 0.5rem;
		background: white;
		border-radius: 4px;
		border: 1px solid #ddd;
		white-space: pre-wrap;
		word-break: break-word;
	}
</style>
