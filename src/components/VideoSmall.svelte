<script>
	export let src;
	export let thumbnail;
	export let title;
</script>

<div class="video">
	<div class="thumbnail">
		<a href={src}>
			<img src={thumbnail} alt="" />
		</a>
	</div>
	<div class="title">{title}</div>
</div>

<style>
	@import url(https://fonts.bunny.net/css?family=abeezee:400);

	* {
		font-family: 'ABeeZee', sans-serif;
	}

	.video {
		display: flex;
		flex-direction: column;
	}

	.thumbnail {
		width: 100%;
		height: 100%;
		gap: 1em;
	}

	img {
		width: 100%;
		height: 100%;
		display: block

	}

	.title {
		color: aliceblue;
        font-size: 1.2em;
        padding: 5px 0 5px 5px;
        background-color: #292d33;
	}
</style>
