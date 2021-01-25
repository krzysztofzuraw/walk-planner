<script lang="ts">
  import mapboxgl from "mapbox-gl";
  import { onMount, onDestroy } from "svelte";
  import Geolocation from "svelte-geolocation";
  import { coordAll, featureCollection, point } from "@turf/turf";

  const { SNOWPACK_PUBLIC_MAPBOX_ACCESS_TOKEN } = import.meta.env;

  let mapRef: mapboxgl.Map;
  let getPosition = false;
  let userLocation: number[] = [];
  const waypoints = featureCollection([]);
  const nothing = featureCollection([]);

  const createWaypoint = (coords: any) => {
    const pt = point([coords.lng, coords.lat], {
      orderTime: Date.now(),
      key: Math.random(),
    });
    waypoints.features.push(pt);
  };

  const updateWaypoints = (geojson: any) => {
    //@ts-ignore
    mapRef.getSource("waypoints").setData(geojson);
  };

  const generateQueryURL = () => {
    const coordinates = [userLocation, coordAll(waypoints), userLocation];
    const distributions = [];
    const end = [userLocation];

    return `https://api.mapbox.com/optimized-trips/v1/mapbox/walking/${coordinates.join(
      ";"
    )}&overview=full&steps=true&geometries=geojson&source=first&access_token=${SNOWPACK_PUBLIC_MAPBOX_ACCESS_TOKEN}`;
  };

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
          //@ts-ignore
          data: waypoints,
        },
        layout: {
          "icon-allow-overlap": true,
          "icon-ignore-placement": true,
          "icon-image": "marker-15",
        },
      });

      mapRef.addSource("route", {
        type: "geojson",
        //@ts-ignore
        data: nothing,
      });

      mapRef.addLayer(
        {
          id: "routeline-active",
          type: "line",
          source: "route",
          layout: {
            "line-join": "round",
            "line-cap": "round",
          },
          paint: {
            "line-color": "#3887be",
            "line-width": ["interpolate", ["linear"], ["zoom"], 12, 3, 22, 12],
          },
        },
        "waterway-label"
      );
    });

    mapRef.on("click", (event) => {
      createWaypoint(mapRef.unproject(event.point));
      updateWaypoints(waypoints);
    });
  });

  const handleGenerateClick = () => {
    fetch(generateQueryURL(), {
      method: "GET",
    }).then((res) => console.log(res));
  };
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

        userLocation = [e.detail.coords.latitude, e.detail.coords.longitude];
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
  <button on:click={handleGenerateClick}>Generate</button>
</div>
