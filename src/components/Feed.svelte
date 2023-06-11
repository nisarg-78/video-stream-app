<script>
	import Categories from './Categories.svelte';
	import VideoSmall from './VideoSmall.svelte';
</script>

<main>
	<div class="categories">
		<Categories />
	</div>

	<div class="videos">
		<div class="video">
			<VideoSmall src="/videos/valoMusic" thumbnail="https://i.imgur.com/3kdF0RT.jpg" title="Valorant Lobby Music"/>
		</div>
		<div class="video">
			<VideoSmall src="/videos/1686374699" thumbnail="https://i.imgur.com/3hhRFzy.png" title="Die For You - VALORANT"/>
		</div>
		<div class="video">
			<VideoSmall src="/videos/overwatchClip" thumbnail="https://i.imgur.com/E6EekBf.jpg" title="Random Overwatch Clip"/>
		</div>
		<!-- <div class="video">
			<VideoSmall src="/videos/tarik" />
		</div> -->
		<div class="video">
			<VideoSmall src="/videos/moonFormation" thumbnail="https://i.imgur.com/vKstGXb.jpg" title="Moon Formation - Our Universe | Netflix"/>
		</div>
	</div>
</main>

<style>
	.videos {
		width: 100%;
		padding: 2em;
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-template-rows: repeat(3, 1fr);
		gap: 2em;
	}
    .video{
        transition: scale 0.2s ease-in-out;
    }
    .video:hover{
        cursor: pointer;
        scale: 1.05;
    }

	main {
		display: flex;
		height: 100%;
	}

	.categories {
		width: 15%;
		height: 100%;
		border-right: 1px solid gray;
	}
</style>
