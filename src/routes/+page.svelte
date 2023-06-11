<script>
	import Navbar from '../components/Navbar.svelte';
	import Feed from '../components/Feed.svelte';
</script>

<div class="container">
	<Navbar />
	<div class="feed">
		<Feed />
	</div>
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		height: 100vh;
	}
	.feed {
		width: 100%;
		flex-grow: 1;
		background-color: #343a40;
	}
</style>
