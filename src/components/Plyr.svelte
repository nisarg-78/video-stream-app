<script>
	import { onMount } from 'svelte';
	import Hls from 'hls.js';

	onMount(async () => {
		const video = document.getElementById('player');
		const src = 'https://demo.unified-streaming.com/k8s/features/stable/video/tears-of-steel/tears-of-steel.mp4/.m3u8';
		const defaultOptions = {};
		if (Hls.isSupported()) {
			const hls = new Hls();
			hls.loadSource(src);

			hls.on(Hls.Events.MANIFEST_PARSED, function () {
				const availableQualities = hls.levels.map((l) => l.height);
				console.log(hls.levels);
				defaultOptions.controls = [
					'play-large',
					'play',
					'progress',
					'current-time',
					'mute',
					'volume',
					'captions',
					'settings',
					'fullscreen'
				];
				defaultOptions.quality = {
					options: availableQualities,
					forced: true,
					onChange: (e) => updateQuality(e)
				};
				function updateQuality(newQuality) {
					console.log(newQuality);
					hls.levels.forEach((level, levelIndex) => {
						if (level.height === newQuality) {
							hls.currentLevel = levelIndex;
						}
					});
				}
				hls.attachMedia(video);
				new Plyr(video, defaultOptions);
			});
		}
	});
</script>

<svelte:head>
	<script src="https://cdn.plyr.io/3.7.8/plyr.js"></script>
</svelte:head>

<!-- svelte-ignore a11y-media-has-caption -->
<video id="player" controls />

<link rel="stylesheet" href="https://cdn.plyr.io/3.7.8/plyr.css" />

<style>
	:root {
		--plyr-color-main: #495057;
		--plyr-video-control-color: #dee2e6;
		--plyr-video-control-background-hover: #495057;
		--plyr-video-controls-background: linear-gradient(rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.75));
		--plyr-control-padding: --plyr-control-spacing * 0;
		--plyr-tooltip-padding: --plyr-control-spacing * 0;
	}

	#player {
		max-height: 100vh;
	}
</style>
