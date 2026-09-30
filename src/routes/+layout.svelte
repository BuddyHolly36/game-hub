<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<title>Game Hub</title>
	<meta name="description" content="Play Chess or Tetris online" />
	<style>
		body {
			margin: 0;
			padding: 0;
			font-family: Arial, sans-serif;
			background-color: #f5f5f5;
		}
		
		header {
			background-color: #333;
			color: white;
			padding: 1rem;
			text-align: center;
		}
		
		header h1 {
			margin: 0;
			font-size: 1.5rem;
		}
		
		nav {
			background-color: #444;
			padding: 0.5rem;
			text-align: center;
		}
		
		nav a {
			color: white;
			text-decoration: none;
			margin: 0 1rem;
			padding: 0.5rem 1rem;
			border-radius: 4px;
			transition: background-color 0.3s;
		}
		
		nav a:hover {
			background-color: #555;
		}
		
		main {
			max-width: 1200px;
			margin: 2rem auto;
			padding: 1rem;
			background-color: white;
			border-radius: 8px;
			box-shadow: 0 2px 4px rgba(0,0,0,0.1);
		}
		
		footer {
			text-align: center;
			padding: 1rem;
			color: #666;
			font-size: 0.9rem;
		}
	</style>
</svelte:head>

<header>
	<h1>Game Hub</h1>
</header>
<nav>
	<a href="/">Home</a>
	<a href="/chess">Chess</a>
	<a href="/tetris">Tetris</a>
</nav>

<main>
	{@render children()}
</main>

<footer>
	<p>Personal game collection</p>
</footer>
