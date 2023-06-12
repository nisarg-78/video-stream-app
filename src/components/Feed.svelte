<script>
	import { onMount } from 'svelte';
	import VideoSmall from './VideoSmall.svelte';

	let tags = [];
	$: videos = [];

	const logTags = (e) => {
		if (e.target.checked) {
			tags = [...tags, e.target.name];
		} else {
			tags = tags.filter((tag) => tag !== e.target.name);
		}
		getVideos(tags);
	};

	function getVideos(tags) {
		console.log(tags);
		let queryTags = tags.toString();
		console.log('http://localhost:3000/videos?tags=' + (queryTags ? queryTags : ''));
		fetch('http://localhost:3000/videos?tags=' + (queryTags ? queryTags : ''))
			.then((res) => res.json())
			.then((data) => {
				console.log(data);
				videos = data;
			});
	}

	onMount(() => {
		getVideos(tags);
	});
</script>

<main>
	<div class="categories">
		<h1>
			<label for="tags">Tags</label>
		</h1>
		<div class="tags">
			<div class="tag">
				<input type="checkbox" group={tags} on:change={logTags} name="Valorant" id="Valorant" />
				<label for="Valorant">Valorant</label>
			</div>
			<div class="tag">
				<input type="checkbox" group={tags} on:change={logTags} name="Music" id="Music" />
				<label for="Music">Music</label>
			</div>
			<div class="tag">
				<input type="checkbox" group={tags} on:change={logTags} name="Gaming" id="Gaming" />
				<label for="Gaming">Gameplay</label>
			</div>
			<div class="tag">
				<input type="checkbox" group={tags} on:change={logTags} name="Science" id="Science" />
				<label for="Science">Science</label>
			</div>
		</div>
	</div>

	<div class="videos">
		{#each videos as video}
			<div class="video">
				<VideoSmall src="/videos/{video.id}" thumbnail={video.thumbnail} title={video.title} />
			</div>
		{/each}
	</div>
</main>

<style>
	@import url(https://fonts.bunny.net/css?family=abeezee:400);

	.videos {
		width: 80%;
		padding: 2em;
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-template-rows: repeat(3, 1fr);
		gap: 2em;
	}
	.video {
		transition: scale 0.2s ease-in-out;
	}
	.video:hover {
		cursor: pointer;
		scale: 1.05;
	}

	main {
		display: flex;
		height: 100%;
	}

	.categories {
		height: 100%;
		width: 10%;
		border-right: 1px solid gray;
		color: aliceblue;
	}

	.categories {
		font-family: 'AbeeZee', sans-serif;
		padding: 20px;
	}

	.tags {
		display: flex;
		flex-direction: column;
	}

	.tag {
		display: flex;
		align-items: center;
		margin-bottom: 10px;
	}

	.tag input[type='checkbox'] {
		display: none;
	}

	.tag label {
		display: inline-block;
		padding: 5px;
		background-color: transparent;
		color: white;
		border: 1px solid gray;
		cursor: pointer;
		transition: background-color 0.3s;
	}

	.tag input[type='checkbox']:checked + label {
		background-color: #ef233c;
		color: #000000
	}

	.tag label:hover {
		background-color: lightgray;
	}
</style>
