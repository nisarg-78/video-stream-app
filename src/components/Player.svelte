<script>
	import { onMount } from 'svelte';
	import Hls from 'hls.js';

	let video;
	let hls;
	const maxVolume = 100;
	let isMuted = false;
	let resolutions = [];
	let fullscreen = false;
	let playing = false;
	$: level = hls ? hls.currentLevel : 0;
	let bufferedTime = 0;
	$: playbackTime = video ? video.currentTime : 0;
	$: duration = hls && playbackTime ? video.duration : 0;

	function handlePause() {
		video.pause();
		playing = false;
	}

	function handlePlay() {
		if (!video) return;
		if (!playing) {
			video.play();
			playing = true;
		} else {
			video.pause();
			playing = false;
		}
	}

	function handleVolume(event) {
		const volume = event.target.value;
		video.volume = volume / maxVolume;
		if (volume === 0) isMuted = true;
		else isMuted = false;
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
		console.log(event.target);
		const newTime = (event.offsetX / event.target.offsetWidth) * duration;
		video.currentTime = newTime;
		playbackTime = newTime;
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
			let src =
				'https://bitdash-a.akamaihd.net/content/sintel/hls/playlist.m3u8';
			hls.loadSource(src);
			hls.attachMedia(video);
			hls.enableWorker = true;

			//TODO: Add subtitle support
			hls.subtitleDisplay = false;

			console.log('Duration:', video.duration);
			duration = video.duration;

			hls.on(Hls.Events.MEDIA_ATTACHED, function () {});
			hls.on(Hls.Events.MANIFEST_PARSED, function (event, data) {
				console.log(data.levels);
				for (let i = 0; i < data.levels.length; i++) {
					resolutions = [...resolutions, [data.levels[i].height, data.levels[i].bitrate]];
				}
				console.log(resolutions);
			});

			video.addEventListener('progress', function () {
				let buffered = video.buffered;
				if (buffered.length > 0) {
					bufferedTime = buffered.end(buffered.length - 1);
				}
			});
		}
	});
</script>

<div class="container">
	<div class="player-container">
		<!-- svelte-ignore a11y-media-has-caption -->
		<video
			class="video"
			bind:this={video}
			on:click={handlePlay}
			bind:currentTime={playbackTime}
			loop
		/>

		<div class="controls">
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<div class="video-progress" on:click={(e) => handleSeek(e)}>
				<div class="video-bar" />
				<div class="video-consumed" style="min-width: {(playbackTime / duration) * 100}%" />
				<div class="video-buffered" style="min-width: {(bufferedTime / duration) * 100}%" />
			</div>

			<div class="control-handles">
				<!-- play/pause handle  -->
				<div>
					{#if playing}
						<button on:click={handlePlay}>
							<i class="fa fa-pause" aria-hidden="true" />
						</button>
					{:else}
						<button on:click={handlePlay}>
							<i class="fa fa-play" aria-hidden="true" />
						</button>
					{/if}
				</div>

				<!-- playback time -->
				<div class="progress-time">
					{new Date(Number(Number(playbackTime).toFixed(0)) * 1000)
						.toISOString()
						.substring(14, 19)}/{new Date(duration * 1000).toISOString().substring(14, 19)}
				</div>

				<!-- volume hanlde -->
				<div class="volume-hanlde">
					{#if isMuted}
						<button on:click={handleMute}>
							<i class="fa fa-volume-off" aria-hidden="true" />
						</button>
					{:else}
						<button on:click={handleMute}>
							<i class="fa fa-volume-up" aria-hidden="true" />
						</button>
					{/if}
					<input
						type="range"
						on:input={handleVolume}
						id="volume"
						name="volume"
						min="0"
						max={maxVolume}
					/>
				</div>

				<!-- quality handle -->
				<div class="resolution-selector">
					<select on:change={handleQaulity} name="quality" id="quality">
						{#each resolutions as res}
							<option value={res}>{res[0]}p ({(res[1] / 1024 / 1024).toFixed(2)}Mbps)</option>
						{/each}
						<option value="auto">Auto {hls ? hls?.levels[level]?.height + 'p' : ''}</option>
					</select>
				</div>
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
<link rel="preconnect" href="https://fonts.bunny.net">
<link href="https://fonts.bunny.net/css?family=andika:400" rel="stylesheet" />

<style>
	.container {
		font-family: 'Andika', sans-serif;
		background-color: #232a30;
		height: 100vh;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.player-container {
		width: 90%;
		display: flex;
		flex-direction: column;
		position: relative;
	}

	.video-consumed,
	.video-buffered {
		position: absolute;
		height: 3px;
	}

	.video-progress {
		width: 100%;
		height: 3px;
		background-color: rgba(255, 255, 255, 0.336);
		display: flex;
		position: relative;
	}

	.video-bar {
		min-width: 100%;
		height: 3px;
		z-index: 2;
	}

	.video-consumed {
		background-color: red;
		z-index: 1;
	}

	.video-buffered {
		background-color: #4d81af;
	}

	.controls {
		position: absolute;
		bottom: 0;
		width: 100%;
		background-color: #3e4b56;
		display: flex;
		flex-direction: column;
		align-items: center;
		color: rgb(255, 255, 255);
		background: linear-gradient(
			0deg,
			rgba(0, 0, 0, 0.4009978991596639) 0%,
			rgba(0, 0, 0, 0) 74%,
			rgba(255, 255, 255, 0) 100%
		);
		opacity: 0;
		transition: opacity 0.3s;
	}

	.player-container:hover .controls {
		opacity: 1;
	}

	.volume-hanlde {
		display: flex;
		align-items: center;
	}

	.control-handles {
		width: 100%;
		display: flex;
		justify-content: flex-start;
		margin: 0.35rem 0;
	}

	.control-handles > div {
		margin: 0 10px;
	}

	.container button {
		all: unset;
		cursor: pointer;
		color: rgb(201, 201, 201);
		font-size: 0.75rem;
		margin: 0 0.25rem;
	}

	.control-handles .resolution-selector {
		margin-left: auto;
		display: flex;
		align-items: center;
	}

	
</style>
