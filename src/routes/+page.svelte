<h1>Choose a Game</h1>

<div class="game-grid">
	<a href="/chess" class="game-card">
		<h2>Chess</h2>
		<p>Classic strategy game for two players</p>
		<div class="game-icon">♟️</div>
	</a>
	
	<a href="/tetris" class="game-card">
		<h2>Tetris</h2>
		<p>Addictive puzzle game</p>
		<div class="game-icon">🧩</div>
	</a>
</div>

<style>
	.game-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
		gap: 2rem;
		margin-top: 2rem;
	}
	
	.game-card {
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		color: white;
		padding: 2rem;
		border-radius: 12px;
		text-decoration: none;
		transition: transform 0.3s, box-shadow 0.3s;
		text-align: center;
		cursor: pointer;
	}
	
	.game-card:hover {
		transform: translateY(-5px);
		box-shadow: 0 8px 25px rgba(0,0,0,0.2);
	}
	
	.game-card h2 {
		margin: 0 0 0.5rem 0;
		font-size: 2rem;
	}
	
	.game-card p {
		margin: 0 0 1rem 0;
		opacity: 0.9;
	}
	
	.game-icon {
		font-size: 3rem;
	}
</style>
