<script>
	import Player from "../../../components/Player.svelte";
	import { page } from '$app/stores'
	import Navbar from "../../../components/Navbar.svelte";

	console.log($page)
	let videoId = $page?.params?.videoId
	const src = 'https://video-stream-7f9u.onrender.com/' + videoId

</script>

<Navbar />

<Player {src} />