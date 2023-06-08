<script>
	import { onMount } from 'svelte';
	import Hls from 'hls.js';

	let video;
	let hls;
	const maxVolume = 100;
	$: playing = false;
	$: isMuted = false;
	$: items = [];
	$: level = hls ? hls.currentLevel : 0;
	$: fullscreen = false;
	$: playbackTime = 0;
	$: duration = hls && playbackTime ? video.duration : 0;

	function handlePause() {
		video.pause();
		playing = false;
	}

	function handlePlay() {
		video.play();
		playing = true;
	}

	function handleVolume(event) {
		const volume = event.target.value;
		video.volume = volume / maxVolume;
		if (volume === 0) isMuted = true;
		else isMuted = false;
		console.log(volume)
	}

	function handleMute() {
		if (isMuted) {
			isMuted = false;
			video.volume = volume;
		} else {
			isMuted = true;
			video.volume = 0;
		}
		return;
	}

	function handleSeek(event) {
		console.log(event.target.value)
		const seek = event.target.value / duration;
		console.log(seek)
		video.currentTime = seek;
		playbackTime = event.target.value;
	}

	setInterval(() => {
		if (hls && hls.currentLevel !== level) {
			console.log('hls.currentLevel:', hls.currentLevel);
			level = hls.currentLevel;
		}
	}, 1000);

	function handleQaulity(event) {
		if (hls) {
			if (event.target.value === 'auto') {
				hls.currentLevel = -1;
				return;
			}
			const [height, bitrate] = event.target.value.split(',');
			hls.levels.forEach((level, levelIndex) => {
				if (level.height === parseInt(height) && level.bitrate === parseInt(bitrate)) {
					console.log(hls.levels[levelIndex]);
					hls.currentLevel = levelIndex;
					console.log(hls.currentLevel);
				}
			});
		}
	}

	onMount(() => {
		if (Hls.isSupported()) {
			hls = new Hls();
			hls.loadSource(
				'https://demo.unified-streaming.com/k8s/features/stable/video/tears-of-steel/tears-of-steel.mp4/.m3u8'
			);
			hls.attachMedia(video);

			console.log('Duration:', video.duration);
			duration = video.duration;

			hls.on(Hls.Events.MEDIA_ATTACHED, function () {});
			hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
				console.log(data.levels);
				for (let i = 0; i < data.levels.length; i++) {
					items = [...items, [data.levels[i].height, data.levels[i].bitrate]];
				}
				console.log(items);
			});
		}
	});
</script>

<div class="container">
	<div class="player-container">
		<!-- svelte-ignore a11y-media-has-caption -->
		<video class="player" bind:this={video} bind:currentTime={playbackTime} loop />

		<div class="controls">

			<div>
				{#if playing}
					<button on:click={handlePause}>
						<i class="fa fa-pause" aria-hidden="true" />
					</button>
				{:else}
					<button on:click={handlePlay}>
						<i class="fa fa-play" aria-hidden="true" />
					</button>
				{/if}
			</div>

			<div>
				{#if isMuted}
					<button on:click={handleMute}>
						<i class="fa fa-volume-off" aria-hidden="true" />
					</button>
				{:else}
					<button on:click={handleMute}>
						<i class="fa fa-volume-up" aria-hidden="true" />
					</button>
				{/if}
				<input type="range" on:input={handleVolume} id="volume" name="volume" min="0" max={maxVolume} />
			</div>

			<div class="videoProgress">
				<span
					>{new Date(Number(Number(playbackTime).toFixed(0)) * 1000).toISOString().substring(14, 19)}</span
				>
				<input
					type="range"
					id="seek"
					value={playbackTime}
					on:change={handleSeek}
					on:mousedown={handlePause}
					on:mouseup={handlePlay}
					name="seek"
					min="0"
					max={duration ? duration : 0}
				/>
				{new Date(duration * 1000).toISOString().substring(14, 19)}
			</div>

			<div class="resolutionSelector">
				<select on:change={handleQaulity} name="quality" id="quality">
					{#each items as item}
						<option value={item}>{item[0]}p {(item[1]/1024/1024).toFixed(2)}Mbps</option>
					{/each}
					<option value="auto">Auto {hls ? hls?.levels[level]?.height + 'p' : ''}</option>
				</select>
			</div>

		</div>
	</div>
</div>

<link
	rel="stylesheet"
	href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"
	integrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw=="
	crossorigin="anonymous"
	referrerpolicy="no-referrer"
/>

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
		display: flex;
		flex-direction: column;
	}

	.player {
		width: 100%;
	}

	.controls{
		width: 100%;
		background-color: #3e4b56;
		display: flex;
		align-items: center;
		color: rgb(201, 201, 201);
	}

	.controls div{
		display: flex;
		align-items: center;
	}

	.container .videoProgress{
		flex-grow: 1;
		margin: 0 1rem;
	}

	.videoProgress{
		display: flex;
	}

	.videoProgress input{
		flex-grow: 1;
	}


	.resolutionSelector {
		width: fit-content;
	}
</style>
