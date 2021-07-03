<script>
      import { Map, Geocoder, Marker, controls } from '@beyonk/svelte-mapbox';
      const { GeolocateControl, NavigationControl, ScaleControl } = controls;
      import {afterUpdate, getContext, onMount, setContext} from 'svelte';


      let center = [-74.5, 40];
      let mapComponent;
      let zoom = 10;
      $: lng = center[0];
      $: lat = center[1];

      onMount(async() => {

        if (navigator.geolocation) {

      const getCoords = () => {

          return new Promise((resolve, reject) => {

          navigator.geolocation.getCurrentPosition(resolve, reject)

          })

          }



          const getLocation = async () => {

          let pos = {};

          let position = await getCoords();

          pos.lat = position.coords.latitude;

          pos.lng = position.coords.longitude;

          center = [pos.lng, pos.lat];

          mapComponent.setCenter([pos.lng, pos.lat], 15)

          return pos

          };

          getLocation().then(res => {

          console.log(res);

          })
          }
      })


</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<div class="section-txt" id="map">
    <div class="map-wrap">
    <Map
      accessToken="pk.eyJ1IjoibGV0b3VycG93ZXJzY29tYmluZSIsImEiOiJjazFmN3N0eTUwb3JwM2JwYWk4ZXB1enNtIn0._UjpOqZIeiWqhscosubipw"
      bind:this={mapComponent}
      on:recentre={e => console.log(e.detail) }
      {center}
      bind:zoom
    >
    <Marker bind:lat bind:lng />
  </Map>
    </div>

</div>

<style>
    :global(.mapboxgl-map) {
        height: 100%;
    }

    .map-wrap {
		width: 100%;
    height: 80vh;
		/* height: 100vh; */
	}
</style>