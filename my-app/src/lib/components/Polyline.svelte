<script lang="ts">
    import { onMount, onDestroy, getContext, setContext } from 'svelte';
    import L from 'leaflet';

    const { latLngList, weatherState} = $props();

    let polyline = $state<L.Polyline | undefined>(undefined);
    let polylineElement: HTMLDivElement;

    const { getMap }: { getMap: () => L.Map | undefined } = getContext('map');
    const map = getMap();

    setContext('layer', {
        // L.Polyline inherits from L.Layer
        getLayer: () => polyline
    });

    async function getRouteData() {
        let coordinates = latLngList.map((coord) => {
            if (Array.isArray(coord)) {
                // coord is [lat, lng] → reverse to [lng, lat]
                return [coord[1], coord[0]];
            }
            // coord is LatLngLiteral → convert to array
            return [coord.lng, coord.lat];
        });

        let coordinatesStr = coordinates
            .map(coord => coord.join(','))
            .join(';');

        const url = `https://router.project-osrm.org/route/v1/car/${coordinatesStr}?overview=full&geometries=geojson`;

        try {
            const response = await fetch(url);      
            const data = await response.json();
            

            // coord is [lng, lat] → reverse to [lat, lng]
            const routeCoordinates = data.routes[0].geometry.coordinates.map((coord: number[]) => [coord[1], coord[0]]);

          return routeCoordinates;
        } catch (error) {
          console.error("Route ophalen mislukt:", error);
          return [];
        }
    }

    onMount(async () => {
        if (map) {
            const coords = await getRouteData();
            console.log(weatherState);
            
            
            polyline = L.polyline(coords, {
                color: weatherState > 60 ? 'darkblue' : weatherState > 20 ? 'blue' : 'green',
                weight: 4,
                opacity: 0.7,
            }).addTo(map);
        }
    });

    onDestroy(() => {
        polyline?.remove();
        polyline = undefined;
    });

    $effect(() => {
        if (polyline) { 
            polyline.setStyle({
                color: weatherState > 60 ? 'red' : weatherState > 20 ? 'blue' : 'green',
            });
        }
    })
</script>

<div bind:this={polylineElement}>
    {#if polyline}
        <slot />
    {/if}
</div>