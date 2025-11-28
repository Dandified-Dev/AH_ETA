<script lang="ts">
  import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
  import { CloudRain, Clock, AlertTriangle } from "lucide-svelte";
  import '../app.css';
  import type { LatLngExpression } from 'leaflet';
  import Leaflet from '$lib/components/Leaflet.svelte';
  import Marker from '$lib/components/Marker.svelte';
	import Popup from '$lib/components/Popup.svelte';
	import Polyline from '$lib/components/Polyline.svelte';

  const initialView: LatLngExpression = [52.435362, 4.816222]; // Ahold Delhaize Head Office
  const markerIcons: Array<string> = [
    'src/lib/assets/office.svg',
    'src/lib/assets/store.svg'
  ]; 
  const markerLocations: Array<LatLngExpression> = [
    [52.435362, 4.816222],
    [52.694118, 5.177879],
    [52.698253, 5.234271],
    [52.516963, 6.099787],
    [52.171251, 5.003938]
  ];

  const markerDescriptions: Array<string> = [
    'Ahold Delhaize Head Office',
    'Albert Heijn Hoogkarspel',
    'Albert Heijn Bovenkarspel',
    'Albert Heijn Zwolle',
    'Albert Heijn Breukelen'
  ];

  const markers = markerLocations.map((location, index) => ({
    latLng: location,
    icon: index === 0 ? markerIcons[0] : markerIcons[1],
    description: markerDescriptions[index % markerDescriptions.length]
  }));
</script>

<div class="flex flex-1 flex-col gap-4 p-4 h-full">
  <div class="grid auto-rows-min gap-4 md:grid-cols-3 mb-4">
    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Weersomstandigheden</CardTitle>
        <CloudRain class="h-4 w-4 text-blue-500" />
      </CardHeader>
      <CardContent>
        <div class="text-2xl font-bold">Regenachtig</div>
        <p class="text-xs text-muted-foreground">80% intensiteit</p>
      </CardContent>
    </Card>

    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Verwachte Aankomst</CardTitle>
        <Clock class="h-4 w-4 text-ah-blue" /> </CardHeader>
      <CardContent>
        <div class="text-2xl font-bold">12:40</div>
        <p class="text-xs text-muted-foreground">Rit ID: #38291</p>
      </CardContent>
    </Card>

    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Status</CardTitle>
        <AlertTriangle class="h-4 w-4 text-red-500" />
      </CardHeader>
      <CardContent>
        <div class="text-2xl font-bold text-red-600">+5 min</div>
        <p class="text-xs text-muted-foreground">Vertraging door weer</p>
      </CardContent>
    </Card>

  </div>
    <div class="w-full flex-1">
      <Leaflet view={initialView} zoom={10} >
        {#each markers as { latLng, icon, description }}
          <Marker {latLng} width={40} height={40}>
            <img class="h-10" src={icon} alt="Icon">
            <Popup>{description}</Popup>
          </Marker>
        {/each}
        <Polyline latLngList={markers.map(marker => marker.latLng)} />
      </Leaflet>
    </div>
</div>
