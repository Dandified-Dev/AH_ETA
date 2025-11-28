<script lang="ts">
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';
  
  // Props: de slider waarde (0 tot 100)
  let { trafficIntensity } = $props(); 

  let mapElement: HTMLDivElement;
  let map: any;
  let routeLine: any;

  // Hardcoded route: AH Zaandam DC -> AH Amsterdam Museumplein
  // Je kunt deze punten gewoon van Google Maps plukken (rechtermuisknop -> "wat is hier")
// Coördinaten: [Latitude (Breedte), Longitude (Lengte)]
  const startPoint: [number, number] = [52.4485, 4.8275]; // AH DC Zaandam
  const endPoint: [number, number] = [52.694112, 5.177883];   // AH Museumplein

  // OSRM wil: Longitude,Latitude (Let op de volgorde!)
  // Dus we moeten ze even omdraaien voor de URL string
  async function getRouteData() {
    const startStr = `${startPoint[1]},${startPoint[0]}`;
    const endStr = `${endPoint[1]},${endPoint[0]}`;
    
    // De gratis API call
    const url = `https://router.project-osrm.org/route/v1/driving/${startStr};${endStr}?overview=full&geometries=geojson`;

    try {
      const response = await fetch(url);      
      const data = await response.json();
      
      // OSRM geeft coördinaten terug als [Lon, Lat], maar Leaflet wil [Lat, Lon]
      // Dus we mappen ze even om:
      const routeCoordinates = data.routes[0].geometry.coordinates.map((coord: number[]) => [coord[1], coord[0]]);
      
      return routeCoordinates;
    } catch (error) {
      console.error("Route ophalen mislukt:", error);
      return [];
    }
  }

  onMount(async () => {
    if (browser) {
      const L = (await import('leaflet')).default;
      map = L.map(mapElement).setView(startPoint, 11);

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap'
      }).addTo(map);

      // --- HIER ROEP JE DE API AAN ---
      const coords = await getRouteData();

      if (coords.length > 0) {
        routeLine = L.polyline(coords, {
          color: 'green',
          weight: 6,
          opacity: 0.8
        }).addTo(map);

        map.fitBounds(routeLine.getBounds(), { padding: [50, 50] });
      }
    }
  });

  // Reactiviteit: Verander kleur als trafficIntensity verandert
  $effect(() => {
    const intensity = trafficIntensity;
    if (routeLine) {
      if (intensity > 50) {
        routeLine.setStyle({ color: 'red' });
      } else if (intensity > 20) {
        routeLine.setStyle({ color: 'orange' });
      } else {
        routeLine.setStyle({ color: 'green' });
      }
    }
  });
</script>

<svelte:head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
</svelte:head>

<div bind:this={mapElement} class="w-full h-[400px] rounded-xl overflow-hidden border shadow-sm"></div>