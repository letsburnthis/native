<script>
      import { Map, Geocoder, Marker, controls } from '@beyonk/svelte-mapbox';
      const { GeolocateControl, NavigationControl, ScaleControl } = controls;
      import {afterUpdate, getContext, onMount, setContext} from 'svelte';


      export let sites;
      let center = [-74.5, 40];
      let mapComponent;
      let zoom = 3;
      let type = "plant_natives_here";
      $: lng = center[0];
      $: lat = center[1];
      $: style = 'mapbox://styles/mapbox/satellite-v9'
      // $: style = 'mapbox://styles/mapbox/satellite-streets-v11'
      let menu_display = "plant";


      let projectsArrayGeojson = [];


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

          mapComponent.setCenter([pos.lng, pos.lat], 17)

          zoom = 17;

          return pos

          };

          getLocation().then(res => {

          console.log(res);

          })
          }
          console.log(sites);
          
          // for (var i=0; i < sites.length; i++) {

          // let new_feature =   {
          //                       "type": "Feature",
          //                     "geometry": {
          //                       "type": "Point",
          //                       "coordinates": [
          //                         sites[i].lng,
          //                         sites[i].lat
          //                       ]
          //                     }
          //                   }

          // projectsArrayGeojson.push(new_feature);

          // }
          
          // mapComponent.addSource('projects', {
          //   'type': 'geojson',
          //   'data': projectsArrayGeojson
          //   });

          // mapComponent.addLayer({
          //   id: 'projectsLayer',
          //   type: 'circle',
          //   source: 'projects',
          //   layout: {
          //       // make layer visible by default
          //       visibility: "visible",
          //       },
          //   paint: {
          //   "circle-color": "blue",
          //   "circle-radius": 30
          //   // 'circle-radius':{
          //   //         'base': 5,
          //   //         'stops': [
          //   //         [12, 2],
          //   //         [22, 180]
          //   //         ]
          //   //       }          
          //     }
          //   });
      })

async function publishLocation() {

        var formData = new FormData();

        formData.append('lng', lng);
        formData.append('lat', lat);
        formData.append('type', type);
        formData.append('lng_lat', JSON.stringify(center));


        console.log(formData);

        const response = await fetch('/publish_location', {
          method: 'post',
          body: formData
        })

        if (response.ok) {

          let response_json = await response.json();
          console.log(response_json);
          console.log(response_json[0].id);

          // let redirect = '/' + response_json[0].id;
          
          // window.location = redirect;
        }
        else {
          let response_json = await response.json();
          console.log(response_json);
          console.log(response_json.status);
          console.log(response.body);
          // alert(await response.text())
        }

}

function handleMapClick(e) {

  console.log(e.detail);
  setTimeout( function () {
    let popups = document.getElementsByClassName('mapboxgl-popup beyonk-mapbox-popup mapboxgl-popup-anchor-bottom');
    console.log(popups.length);

    if (popups.length == 0) {
      lng = e.detail.lng; 
      lat = e.detail.lat; 
  }
  }, 50);

}

async function publishSite(e) {

var formData = new FormData(e.target);

if (menu_display == "share_seeds_plants") {
  type = "Seeds and plants here"
}

else if (menu_display == "help") {
  type = "Willing to help others"
}

formData.append('lng', lng);
formData.append('lat', lat);
formData.append('type', type);
formData.append('lng_lat', JSON.stringify(center));


console.log(formData);

const response = await fetch('/publish_location', {
  method: 'post',
  body: formData
})

if (response.ok) {

  let response_json = await response.json();
  console.log(response_json);
  console.log(response_json[0].id);
  sites.push(response_json[0]);
  sites = sites;
  menu_display = "offers";

  // let redirect = '/' + response_json[0].id;
  
  // window.location = redirect;
}
else {
  let response_json = await response.json();
  console.log(response_json);
  console.log(response_json.status);
  console.log(response.body);
  // alert(await response.text())
}

}

function handleMessage(event) {
		alert(event.detail.text);
    console.log(event.detail.text);
	}


</script>

<script context="module">

  let sites;


  export const load = async ({ fetch }) => {

    const res = await fetch('/load_sites',{
      method: 'get'
    });

		if (res.ok) {
			const sites = await res.json();

			return {
				props: { 
          sites: sites.table
        }
			};
		}

		const { message } = await res.json();

		return {
			error: new Error(message)
		};
	};
  
</script>

<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<button on:click={publishLocation}>Publish location</button>

<div class="section-txt" id="map">
    <div class="map-wrap">
    <Map
      accessToken="pk.eyJ1IjoibGV0b3VycG93ZXJzY29tYmluZSIsImEiOiJjazFmN3N0eTUwb3JwM2JwYWk4ZXB1enNtIn0._UjpOqZIeiWqhscosubipw"
      bind:this={mapComponent}
      on:recentre={e => { console.log(e.detail); } }
      {center}
      on:click={handleMapClick}
      bind:zoom
      style={style}
    >
    <!-- on:click={e => { console.log(e.detail); lng = e.detail.lng; lat = e.detail.lat; if (zoom < 10) { zoom = 17; }} } -->

    <Marker bind:lat bind:lng color="blue" />

    {#if sites}
    {#each sites as site}
      <Marker lng={site.lng} lat={site.lat} label={site.type} color="blue" />
    {/each}
    {/if}
  </Map>


  <div class="interaction" style="position: absolute; bottom: 15%">
    {#if menu_display == "plant"}
    <button style="display: block;">Learn myself</button>
    <button style="display: block;">Plant with a peer</button>
    <button style="display: block;">Someone, plant here!</button>
    <hr>
    <button style="display: block;" on:click={function() {menu_display = "offers"}}>Have something to offer?</button>
    {:else}
    <button style="display: block;" on:click={function() {menu_display = "share_seeds_plants"}}>Seeds / plants to share.</button>
    {#if menu_display == "share_seeds_plants"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">List the seeds and plants you're willing to share, and anything else you'd like others to know.</label>
      <textarea name="content"></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {menu_display = "help"}}>I'm a planter, willing to help others.</button>
    {#if menu_display == "help"}
    <form on:submit|preventDefault={publishSite}>
      <label for="content">Anything you want to add or others should know?</label>
      <textarea name="content"></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <hr>
    <button style="display: block;" on:click={function() {menu_display = "plant"}}>Want to plant something?</button>
    {/if}
  </div>
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

 .interaction > button {
   text-align: center;
   margin: auto;
   margin-bottom: 15px;
   width: 60vw;
   /* height: 24px; */
   font-size: 18px;
    /* left: 50%;
    transform: translate(-50%, 50%); */
  }

  .interaction {
    left: 50%;
    transform: translate(-50%, 50%);
  }
</style>