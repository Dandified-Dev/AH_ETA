<script lang="ts">
  import { Card, CardContent, CardHeader, CardTitle } from "$lib/components/ui/card";
  import { Slider } from "$lib/components/ui/slider/index.js";
  import { CloudRain, Clock, AlertTriangle, Truck } from "lucide-svelte";
  import '../app.css';
  import * as Select from "$lib/components/ui/select/index.js";
  import { Toaster, toast } from "svelte-sonner";
  import type { LatLngExpression } from 'leaflet';
  import Leaflet from '$lib/components/Leaflet.svelte';
  import Marker from '$lib/components/Marker.svelte';
  import Popup from '$lib/components/Popup.svelte';
  import Polyline from '$lib/components/Polyline.svelte';

  let weatherState = $state(30);
  let selectedRouteValue = $state("rit-1");
  let hasWarned = false;

  const routes = [
    {
      value: "rit-1",
      label: "Rit A: Zaandam DC -> Amsterdam",
      start: [52.435362, 4.816222] as LatLngExpression,
      end: [52.372879, 4.890579] as LatLngExpression,
      baseTime: 40
    },
    {
      value: "rit-2",
      label: "Rit B: Zwolle DC -> Groningen",
      start: [52.516963, 6.099787] as LatLngExpression,
      end: [53.2194, 6.5665] as LatLngExpression,
      baseTime: 55
    },
    {
      value: "rit-3",
      label: "Rit C: Geldermalsen -> Utrecht",
      start: [51.8833, 5.2833] as LatLngExpression,
      end: [52.0907, 5.1214] as LatLngExpression,
      baseTime: 30
    }
  ];

  let currentRoute = $derived(routes.find(r => r.value === selectedRouteValue) || routes[0]);

  let currentMarkers = $derived([
    { latLng: currentRoute.start, icon: '/icons/office.svg', description: 'Distributiecentrum' },
    { latLng: currentRoute.end, icon: '/icons/store.svg', description: 'Winkel Locatie' }
  ]);

  let delay = $derived(Math.floor((weatherState / 100) * 20)); 
  let totalTime = $derived(currentRoute.baseTime + delay);

  function formatTime(minutesToAdd: number) {
      const now = new Date();
      now.setHours(14, 0, 0); 
      now.setMinutes(now.getMinutes() + minutesToAdd);
      return now.toLocaleTimeString('nl-NL', { hour: '2-digit', minute: '2-digit' });
  }

  $effect(() => {
    if (weatherState> 80 && !hasWarned) {
      toast.error(`⚠️ Kritieke vertraging op ${currentRoute.label}`, {
        description: `De vertraging is opgelopen tot ${delay} minuten door noodweer.`,
        duration: 5000,
      });
      hasWarned = true;
    } else if (weatherState < 80) {
      hasWarned = false;
    }
  });

</script>

<Toaster position="top-center" richColors />

<div class="flex flex-1 flex-col gap-6 p-6 h-full bg-slate-50">
  
  <div class="flex flex-col md:flex-row justify-between items-center bg-white p-4 rounded-xl border shadow-sm gap-4">
    <div class="flex items-center gap-3 text-[#00AFE1] font-bold text-xl">
        <Truck class="h-8 w-8" /> 
        <span>AH Transport Control</span>
    </div>
    
    <div class="w-full md:w-[350px]">
        <Select.Root type="single" bind:value={selectedRouteValue}>
          <Select.Trigger>
              {currentRoute.label}
          </Select.Trigger>
          <Select.Content>
              {#each routes as route}
                  <Select.Item value={route.value} label={route.label}>{route.label}</Select.Item>
              {/each}
          </Select.Content>
        </Select.Root>
    </div>
  </div>

  <div class="grid auto-rows-min gap-4 md:grid-cols-3">
    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Weersomstandigheden</CardTitle>
        <CloudRain class="h-4 w-4 {weatherState > 50 ? 'text-blue-500' : 'text-gray-300'}" />
      </CardHeader>
      <CardContent>
        <div class="text-2xl font-bold mb-2">
          {weatherState > 60 ? 'Zware Regen' : weatherState > 20 ? 'Lichte Regen' : 'Zonnig'}
        </div>
        <Slider bind:value={weatherState} type="single" max={100} step={1} class="w-full py-2" />
        <p class="text-xs text-muted-foreground mt-1 text-right">{weatherState}% intensiteit</p>
      </CardContent>
    </Card>

    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Verwachte Aankomst</CardTitle>
        <Clock class="h-4 w-4 text-ah-blue" /> </CardHeader>
      <CardContent>
        <div class="text-4xl font-bold text-gray-900">{formatTime(totalTime)}</div>
        <p class="text-xs text-muted-foreground">Basisreistijd: {currentRoute.baseTime} min</p>
      </CardContent>
    </Card>

    <Card>
      <CardHeader class="flex flex-row items-center justify-between space-y-0 pb-2">
        <CardTitle class="text-sm font-medium text-muted-foreground">Status</CardTitle>
        <AlertTriangle class="h-4 w-4 {delay > 0 ? 'text-red-500' : 'text-green-500'}" />
      </CardHeader>
      <CardContent>
        <div class="text-2xl font-bold {delay > 0 ? 'text-red-600' : 'text-green-600'}">
            {delay > 0 ? `+${delay} min` : 'Op Tijd'}
        </div>
        <p class="text-xs text-muted-foreground">
            {delay > 0 ? 'Vertraging door weer' : 'Geen bijzonderheden'}
        </p>
      </CardContent>
    </Card>

  </div>

  <div class="w-full flex-1 min-h-[400px] rounded-xl overflow-hidden border shadow-sm bg-white">
    <Leaflet view={currentRoute.start} zoom={10} >     
      {#each currentMarkers as { latLng, icon, description }}
        <Marker {latLng} width={40} height={40}>
          <img class="h-10" src={icon} alt="Icon">
          <Popup>{description}</Popup>
        </Marker>
      {/each}
      <Polyline 
        latLngList={[currentRoute.start, currentRoute.end]} 
        weatherState={weatherState} 
      />
    </Leaflet>
  </div>
</div>