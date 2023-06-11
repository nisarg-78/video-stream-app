<script>
	import Player from '../../../components/Player.svelte';
	import { page } from '$app/stores';
	import Navbar from '../../../components/Navbar.svelte';

	let videoId = $page?.params?.videoId;
	const src = 'https://video-stream-7f9u.onrender.com/videos/' + videoId;
</script>

<div class="container">
	<Navbar />
	<div class="player">
		<Player {src} />
	</div>
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		height: 100vh;
	}
	.player {
		flex-grow: 1;
		background-color: #434768;
	}
</style>
