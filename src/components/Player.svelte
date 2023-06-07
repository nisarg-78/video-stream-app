<script>
	import { onMount } from 'svelte';
	import Hls from 'hls.js';

	let video;
	let hls;
	$: playing = false;
	$: volume = video ? video.volume : 0.2;
	$: isMuted = false;
	$: items = [];

	function handlePause() {
		video.pause();
		playing = false;
	}

	function handlePlay() {
		video.play();
		playing = true;
	}

	function handleVolume(event) {
		volume = event.target.value / 100;
		video.volume = volume;
		if (volume === 0) isMuted = true;
		else isMuted = false;
	}

	function handleMute() {
		if (isMuted) {
			video.volume = volume;
			isMuted = false;
		} else {
			isMuted = true;
			video.volume = 0;
		}
		return;
	}

	function handleQaulity(event) {
		if(hls){
			hls.currentLevel = event.target.value
			console.log('hls.currentLevel', hls.currentLevel)
		}
	}

	onMount(() => {
		if (Hls.isSupported()) {
			hls = new Hls();
			hls.loadSource('http://localhost:3000/videos/video0');
			hls.attachMedia(video);

			video.volume = volume;

			hls.on(Hls.Events.MEDIA_ATTACHED, function () {
				console.log('video and hls.js are now bound together !');
			});
			hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
				console.log(data.levels)
				for(let i = 0; i < data.levels.length; i++) {
					items = [...items, data.levels[i].height];
				}
			});
		}
	});
</script>

<div class="container">
	<!-- svelte-ignore a11y-media-has-caption -->
	<div class="player-container">
		<video class="player" bind:this={video} loop />

		<div class="controls">
			{#if playing}
				<button on:click={handlePause}>Pause</button>
			{:else}
				<button on:click={handlePlay}>Play</button>
			{/if}
			{#if isMuted}
				<button on:click={handleMute}>Unmute</button>
			{:else}
				<button on:click={handleMute}>Mute</button>
			{/if}
			<input type="range" on:change={handleVolume} id="volume" name="volume" min="0" max="100" />

			<select on:change={handleQaulity} name="quality" id="quality">
				{#each items as item, i}
					<option value="{i}">{item} {i}</option>
				{/each}
			</select>

		</div>
	</div>
</div>

<style>
	.container {
		background-color: #232a30;
		width: 100vw;
		height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.player-container {
		width: 100%;
		max-width: 800px;
	}

	.player {
		width: 100%;
	}

	.controls {
		background-color: #3e4b56;
		padding-top: 10px;
		display: flex;
		align-items: center;
		width: 100%;
	}

	.controls button {
		margin: 0 10px;
	}
</style>
