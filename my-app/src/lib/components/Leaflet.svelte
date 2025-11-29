<script lang="ts">
    import { onDestroy, onMount, setContext } from 'svelte';
    import L from 'leaflet';
    import 'leaflet/dist/leaflet.css';

    const { bounds, view, zoom, children } = $props();

    let map = $state<L.Map | undefined>(undefined);
    let mapElement: HTMLDivElement;
    
    let tileLayer: L.TileLayer | undefined;

    let isDark = $state(false);

    const LIGHT_URL = 'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png';
    const DARK_URL = 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png';

    onMount(() => {
        if (!bounds && (!view || !zoom)) {
            throw new Error("Leaflet component requires either 'bounds' or both 'view' and 'zoom' props.");
        }

        map = L.map(mapElement);
        
        isDark = document.documentElement.classList.contains('dark');
        
        tileLayer = L.tileLayer(isDark ? DARK_URL : LIGHT_URL, {
            attribution: '© OpenStreetMap contributors © CARTO',
        }).addTo(map);

        const observer = new MutationObserver((mutations) => {
            mutations.forEach((mutation) => {
                if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
                    isDark = document.documentElement.classList.contains('dark');
                }
            });
        });

        observer.observe(document.documentElement, {
            attributes: true, 
            attributeFilter: ['class'] 
        });

        return () => observer.disconnect();
    });

    setContext('map', {
        getMap: () => map
    });

    $effect(() => {
        if (map) {
            if (bounds) {
                map.fitBounds(bounds);
            } else if (view && zoom) {
                map.setView(view, zoom);
            }
        }
    });

    $effect(() => {
        if (tileLayer) {
            const newUrl = isDark ? DARK_URL : LIGHT_URL;
            tileLayer.setUrl(newUrl); 
        }
    });

    onDestroy(() => {
        map?.remove();
        map = undefined;
    });
</script>

<div class="w-full h-full" bind:this={mapElement}>
    {#if map}
        {@render children?.()}
    {/if}
</div>