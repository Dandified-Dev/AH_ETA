<script lang="ts">
    import { onMount, onDestroy, getContext, setContext } from 'svelte';
    import L from 'leaflet';

    const { latLng, width, height } = $props();

    let marker = $state<L.Marker | undefined>(undefined);
    let markerElement: HTMLDivElement;

    const { getMap }: { getMap: () => L.Map | undefined } = getContext('map');
    const map = getMap();

    setContext('layer', {
        // L.Marker inherits from L.Layer
        getLayer: () => marker
    });

    onMount(() => {
        if (map) {
            let icon = L.divIcon({
                html: markerElement,
                className: 'map-marker',
                iconSize: L.point(width, height)
            });            
            marker = L.marker(latLng, { icon }).addTo(map);
        }
    });

    $effect(() => {
        if (marker) {
            marker.setLatLng(latLng);
        }
    });

    onDestroy(() => {
        marker?.remove();
        marker = undefined;
    });
</script>

<div bind:this={markerElement}>
    {#if marker}
        <slot />
    {/if}
</div>