<script>
	import Player from "../../../components/Player.svelte";
	import { page } from '$app/stores'
	import Navbar from "../../../components/Navbar.svelte";

	console.log($page)
	let videoId = $page?.params?.videoId
	const src = 'http://localhost:3000/videos/' + videoId

</script>

<Navbar />

<Player {src} />