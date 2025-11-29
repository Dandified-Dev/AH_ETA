<script lang="ts">
	import Navbar from '$lib/components/Navbar.svelte';
	import { ModeWatcher } from 'mode-watcher';
	import '../app.css';
 	let { children } = $props();
</script>
 
<ModeWatcher />
 <main class="h-screen flex flex-col">
	<Navbar />
	<div class="p-12 flex-1">		
		{@render children?.()}
	</div>
 </main>
