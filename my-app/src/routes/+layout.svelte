<script lang="ts">
	import Navbar from '$lib/components/Navbar.svelte';
	import '../app.css';
 	let { children } = $props();
</script>
 

 <main class="h-screen flex flex-col">
	<Navbar />
	<div class="p-12 flex-1">		
		{@render children?.()}
	</div>
 </main>
