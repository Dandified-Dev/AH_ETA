<script lang="ts">
    import { onDestroy, getContext, setContext } from 'svelte';
    import L from 'leaflet';

    const { latLngList, weatherState } = $props();

    let polyline = $state<L.Polyline | undefined>(undefined);
    let polylineElement: HTMLDivElement;

    const { getMap }: { getMap: () => L.Map | undefined } = getContext('map');
    const map = getMap();

    setContext('layer', {
        getLayer: () => polyline
    });

    function getColor(intensity: number) {
        if (intensity > 60) return '#ef4444'; 
        if (intensity > 20) return '#f97316'; 
        return '#22c55e'; 
    }

    async function getRouteData() {
        if (!latLngList || latLngList.length === 0) return [];

        let coordinates = latLngList.map((coord: any) => {
            if (Array.isArray(coord)) {
                return [coord[1], coord[0]]; // [lat, lng] -> [lng, lat]
            }
            return [coord.lng, coord.lat];
        });

        let coordinatesStr = coordinates
            .map((coord: any) => coord.join(','))
            .join(';');

        const url = `https://router.project-osrm.org/route/v1/car/${coordinatesStr}?overview=full&geometries=geojson`;

        try {
            const response = await fetch(url);      
            const data = await response.json();
            
            if (!data.routes || data.routes.length === 0) return [];

            // GeoJSON is [lng, lat], Leaflet wants [lat, lng], so reverse
            return data.routes[0].geometry.coordinates.map((coord: number[]) => [coord[1], coord[0]]);
        } catch (error) {
            console.error("Route ophalen mislukt:", error);
            return [];
        }
    }

    async function updateRoute() {
        if (!map) return;

        const coords = await getRouteData();
        if (coords.length === 0) return;

        if (polyline) {
            polyline.remove();
        }

        polyline = L.polyline(coords, {
            color: getColor(weatherState),
            weight: 5,
            opacity: 0.8,
            lineCap: 'round'
        }).addTo(map);

        // Auto-Zoom to new route
        map.fitBounds(polyline.getBounds(), { 
            padding: [50, 50],
            animate: true
        });
    }

    $effect(() => {
        // Read latLngList to trigger update when it changes
        latLngList; 
        updateRoute();
    });

    $effect(() => {
        if (polyline) {             
            polyline.setStyle({
                color: getColor(weatherState),
            });
        }
    });

    onDestroy(() => {
        polyline?.remove();
        polyline = undefined;
    });
</script>

<div bind:this={polylineElement}>
    {#if polyline}
        <slot />
    {/if}
</div>