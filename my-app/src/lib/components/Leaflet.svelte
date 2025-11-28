<script lang="ts">
    import { onDestroy, onMount, setContext } from 'svelte';
    import L from 'leaflet';
    import 'leaflet/dist/leaflet.css';

    let map: L.Map | undefined
    let mapElement: HTMLDivElement;

    export let bounds: L.LatLngBoundsExpression | undefined = undefined;
    export let view: L.LatLngExpression | undefined = undefined;
    export let zoom: number | undefined = undefined;
    
    onMount(() => {
        if (!bounds && (!view || !zoom)) {
            throw new Error("Leaflet component requires either 'bounds' or both 'view' and 'zoom' props.");
        }
        map = L.map(mapElement);

        L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', {
            attribution: '© OpenStreetMap contributors © CARTO',
        }).addTo(map);
    });

    onDestroy(() => {
        map?.remove();
        map = undefined;
    });

    setContext('map', {
        getMap: () => map
    })

    $: if(map) {
        if(bounds) {
            map.fitBounds(bounds);
        } else if(view && zoom) {
            map.setView(view, zoom);
        }
    }
</script>

<div class="w-full h-full" bind:this={mapElement}>
    {#if map}
        <slot />
    {/if}
</div>