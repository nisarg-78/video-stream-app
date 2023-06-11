<script>
	import { onMount } from 'svelte';
	import Hls from 'hls.js';

	export let src;
	let video;
	let playerContainer;
	let playerControls;
	let loaderIcon;
	let hls;
	const maxVolume = 100;
	let isMuted = false;
	let resolutions = [];
	let isFullscreen = false;
	$: playing = video ? !video.paused : false;
	$: level = hls ? hls.currentLevel : 0;
	let bufferedTime = 0;
	let playbackTime = 0;
	$: duration = hls && playbackTime ? video.duration : 0;
	let showControlsTimeout = null;

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

	function handleWaiting() {
		loaderIcon.style.display = 'block';
	}

	function handlePlaying() {
		loaderIcon.style.display = 'none';
	}

	function handleMouseMove() {
		document.body.style.cursor = 'auto';
		if (showControlsTimeout) clearTimeout(showControlsTimeout);
		playerControls.style.opacity = 1;
		showControlsTimeout = setTimeout(() => {
			playerControls.style.opacity = 0;
			document.body.style.cursor = 'none';
		}, 3000);
	}

	function throttle(func, delay) {
		let timer = null;
		return function (...args) {
			if (!timer) {
				func.apply(this, args);
				timer = setTimeout(() => {
					timer = null;
				}, delay);
			}
		};
	}

	const throttledHandleVolume = throttle(handleVolume, 200);

	function handleVolume(event) {
		const volume = event.target.value;
		video.volume = volume / maxVolume;
		if (volume === 0) isMuted = true;
		else isMuted = false;
		console.log(hls.config);
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

	function handleFullscreen() {
		if (!isFullscreen) {
			isFullscreen = true;
			playerContainer.requestFullscreen();
		} else {
			isFullscreen = false;
			document.exitFullscreen();
		}
	}

	onMount(() => {
		console.log('src', src);
		if (Hls.isSupported()) {
			hls = new Hls({
				forceKeyFrameOnDiscontinuity: true
			});
			hls.loadSource(src);
			console.log(hls);
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

			hls.on(Hls.Events.LEVEL_LOADING, function (event, data) {
				if (data.buffering) {
					// Video is buffering
					console.log('Video is buffering');
					// Perform actions when buffering starts
				} else {
					// Buffering has ended
					console.log('Buffering has ended');
					// Perform actions when buffering ends
				}
			});

			console.log(video);

			video.addEventListener('progress', function () {
				let buffered = video?.buffered;
				if (buffered && buffered.length > 0) {
					bufferedTime = buffered.end(buffered.length - 1);
				}
			});
		}
	});
</script>

<div class="container">
	<div
		class="player-container"
		bind:this={playerContainer}
		on:mousemove={handleMouseMove}
		on:mouseleave={() => {
			playerControls.style.opacity = 0;
		}}
	>
		<!-- svelte-ignore a11y-media-has-caption -->
		<video
			class="video"
			bind:this={video}
			on:click={handlePlay}
			on:dblclick={handleFullscreen}
			on:waiting={handleWaiting}
			on:canplay={handlePlaying}
			on:playing={handlePlaying}
			bind:currentTime={playbackTime}
			loop
		/>

		<!-- loading icon -->
		<svg
			xmlns="http://www.w3.org/2000/svg"
			xmlns:xlink="http://www.w3.org/1999/xlink"
			style="margin: auto; background: none; display: block; shape-rendering: auto;"
			width="200px"
			height="200px"
			viewBox="0 0 100 100"
			preserveAspectRatio="xMidYMid"
			class="loader"
			bind:this={loaderIcon}
		>
			<circle
				cx="50"
				cy="50"
				fill="none"
				stroke="#85a2b6"
				stroke-width="10"
				r="35"
				stroke-dasharray="164.93361431346415 56.97787143782138"
			>
				<animateTransform
					attributeName="transform"
					type="rotate"
					repeatCount="indefinite"
					dur="1s"
					values="0 50 50;360 50 50"
					keyTimes="0;1"
				/>
			</circle>
		</svg>

		<div class="controls" bind:this={playerControls}>
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
						on:input={throttledHandleVolume}
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
						<option value="auto"
							>Auto {hls?.levels[level]?.height ? hls?.levels[level]?.height + 'p' : ''}</option
						>
					</select>
				</div>

				<!-- fullscreen handle -->
				<div>
					{#if isFullscreen}
						<button on:click={handleFullscreen}>
							<i class="fa fa-compress" aria-hidden="true" />
						</button>
					{:else}
						<button on:click={handleFullscreen}>
							<i class="fa fa-expand" aria-hidden="true" />
						</button>
					{/if}
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
<link rel="preconnect" href="https://fonts.bunny.net" />
<link href="https://fonts.bunny.net/css?family=andika:400" rel="stylesheet" />

<style>
	.container {
		font-family: 'Andika', sans-serif;
		background-color: #232a30;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.player-container {
		background-color: #000;
		height: 90vh;
		min-width: calc(90vh * 16 / 9);
		display: flex;
		flex-direction: column;
		position: relative;
		justify-content: center;
	}

	.loader {
		position: absolute;
		left: 50%;
		transform: translateX(-50%);
	}

	.video {
		object-fit: contain;
		max-width: 100%;
		max-height: 100%;
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
		position: absolute;
		bottom: -50%;
		min-width: 100%;
		height: 6px;
		z-index: 2;
		cursor: pointer;
	}

	.video-consumed {
		background-color: #ef233c;
		z-index: 1;
	}

	.video-buffered {
		background-color: rgba(255, 232, 232, 0.5);
	}

	.controls {
		position: absolute;
		bottom: 0;
		width: 100%;
		background-color: rgba(0, 0, 0, 0.75);
		display: flex;
		flex-direction: column;
		align-items: center;
		color: rgb(255, 255, 255);
		opacity: 0;
		transition: opacity 0.3s;
	}

	.controls i {
		font-size: 1.4em;
	}

	.player-container:hover .controls {
		opacity: 1;
	}

	.volume-hanlde {
		display: flex;
		align-items: center;
	}

	/* The slider itself */
	.volume-hanlde input[type='range'] {
		-webkit-appearance: none; /* Override default CSS styles */
		appearance: none;
		width: 75%;
		height: 5px;
		background: rgba(255, 0, 0, 0.25);
		outline: none;
		opacity: 1;
		-webkit-transition: 0.2s;
		transition: opacity 0.2s;
	}
	/* The slider handle (use -webkit- (Chrome, Opera, Safari, Edge) and -moz- (Firefox) to override default look) */
	.volume-hanlde input[type='range']::-webkit-slider-thumb,
	.volume-hanlde input[type='range']::-moz-range-thumb{
		-webkit-appearance: none;
        appearance: none;
        width: 25px;
        height: 5px;
        background: #ef233c;
        cursor: pointer;
	}

	input[type='range']::-moz-range-progress{
		background-color: #ef233c;
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
