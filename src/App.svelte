<script lang="ts">
  import mapboxgl from "mapbox-gl";
  import { onMount, onDestroy } from "svelte";
  import Geolocation from "svelte-geolocation";
  const { SNOWPACK_PUBLIC_MAPBOX_ACCESS_TOKEN } = import.meta.env;

  let mapRef: mapboxgl.Map;
  let getPosition = false;

  onMount(async () => {
    mapboxgl.accessToken = SNOWPACK_PUBLIC_MAPBOX_ACCESS_TOKEN;

    mapRef = new mapboxgl.Map({
      container: "map",
      style: "mapbox://styles/kzuraw/ckk4chfz70i8h17oc4hg96ho5",
      center: { lat: 51.1, lng: 17.03 },
      zoom: 13,
    });

    mapRef.on("load", () => {
      mapRef.addLayer({
        id: "waypoints",
        type: "symbol",
        source: {
          type: "geojson",
          data: {
            type: "FeatureCollection",
            features: [],
          },
        },
      });
    });

    mapRef.on("click", (event) => {
      new mapboxgl.Marker().setLngLat(event.lngLat).addTo(mapRef);
    });
  });
</script>

<div id="map" class="h-screen w-screen" />
<div
  class="absolute top-40 left-40 bg-white p-4 min-w-max rounded-lg shadow-lg flex gap-4 flex-col items-center"
>
  <h1 class="text-5xl font-medium">Walk planner</h1>
  <p
    class="text-normal font-thin gray-500 underline cursor-pointer"
    on:click={() => (getPosition = true)}
  >Use your location to start</p>
  <Geolocation
    {getPosition}
    on:position={(e) => {
      console.log(e.detail); // GeolocationPosition
      if (mapRef) {
        mapRef.flyTo({
          center: {
            lat: e.detail.coords.latitude,
            lng: e.detail.coords.longitude,
          },
        });

        new mapboxgl.Marker()
          .setLngLat({
            lat: e.detail.coords.latitude,
            lng: e.detail.coords.longitude,
          })
          .addTo(mapRef);
      }
    }}
    let:coords
    let:loading
    let:success
    let:error
    let:notSupported
  >
    {#if notSupported}
      Your browser does not support the Geolocation API.
    {:else}
      {#if loading}
        Loading...
      {/if}

      {#if success}
        {JSON.stringify(coords)}
      {/if}

      {#if error}
        An error occurred. {error.code} {error.message}
      {/if}
    {/if}
  </Geolocation>
</div>
