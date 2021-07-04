<script>
      import { Map, Geocoder, Marker, controls } from '@beyonk/svelte-mapbox';
      const { GeolocateControl, NavigationControl, ScaleControl } = controls;
      import {afterUpdate, getContext, onMount, setContext} from 'svelte';


      export let sites;
      let center = [-74.5, 40];
      let mapComponent;
      let zoom = 3;
      let type = "plant_natives_here";
      let natives_list = "Species: \r\n \r\n - White clover (Trifolium repens) \r\n - Yarrow (Achillea millefolium ) \r\n - Nettle (Urtica dioica)";
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

// async function publishLocation() {

//         var formData = new FormData();

//         formData.append('lng', lng);
//         formData.append('lat', lat);
//         formData.append('type', type);
//         formData.append('lng_lat', JSON.stringify(center));


//         console.log(formData);

//         const response = await fetch('/publish_location', {
//           method: 'post',
//           body: formData
//         })

//         if (response.ok) {

//           let response_json = await response.json();
//           console.log(response_json);
//           console.log(response_json[0].id);

//           // let redirect = '/' + response_json[0].id;
          
//           // window.location = redirect;
//         }
//         else {
//           let response_json = await response.json();
//           console.log(response_json);
//           console.log(response_json.status);
//           console.log(response.body);
//           // alert(await response.text())
//         }

// }

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

  let title;

var formData = new FormData(e.target);

if (menu_display == "share_seeds_plants") {
  title = "Seeds and plants here"
}

else if (menu_display == "help") {
  title = "Willing to help others"
}

else if (menu_display == "plant_learn") {
  title = "Learning to plant"
}

else if (menu_display == "plant_with_peers") {
  title = "Planting with peers"
}

else if (menu_display == "plant_here") {
  title = "Plant here!"
}

formData.append('lng', lng);
formData.append('lat', lat);
formData.append('type', menu_display);
formData.append('title', title);
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

<!-- <button on:click={publishLocation}>Publish location</button> -->

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

    <Marker bind:lat bind:lng color="skyblue">
      <!-- <div style="font-size: 18px">
        📍
      </div> -->
    </Marker>

    {#if sites}
    {#each sites as site}
      <Marker lng={site.lng} lat={site.lat} label={site.title} type={site.title} content={site.content} color="blue">
        <div style="font-size: 26px;">
          {#if site.type == "plant_here"}
          ❗
          {:else if site.type == "plant_with_peers"}
          🤝🏼
          {:else if site.type == "plant_learn"}
          🍎
          <!-- 🌿👋 -->
          {:else if site.type == "seed_share"}
          🌱
          <!-- 🌱🌿 -->
          {:else if site.type == "peer"}
          🖖
          {/if}
        </div>
      </Marker>
    {/each}
    {/if}
  </Map>


  <div class="interaction" style="position: absolute; bottom: 15%">
    {#if menu_display.substr(0,5) == "plant"}
    <button style="display: block;" on:click={function() {menu_display = "plant_learn"}}>Learn to plant</button>
    {#if menu_display == "plant_learn"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">Below is a list of local natives.  Add any other information you wish to share with others.</label>
      <textarea name="content" bind:value={natives_list}></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {menu_display = "plant_with_peers"}}>Plant with a peer</button>
    {#if menu_display == "plant_with_peer"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">Below is a list of local natives.  Add any other information you wish to share with others.</label>
      <textarea name="content" bind:value={natives_list}></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {menu_display = "plant_here"}}>Someone, plant here!</button>
    {#if menu_display == "plant_here"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">Below is a list of local natives.  Add any other information you wish to share with others.</label>
      <textarea name="content" bind:value={natives_list}></textarea>
      <button>Submit</button>
    </form>
    {/if}
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
    <button style="display: block;" on:click={function() {menu_display = "peer"}}>I'm a planter, willing to help others.</button>
    {#if menu_display == "peer"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
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

  .myMarker {
  background-color: currentColor;
  background-size: cover;
  box-sizing: border-box;
  width: 50px;
  height: 50px;
  /* border-radius: 50%; */
  cursor: pointer;
  /* transition: all .2s; */
  /* border: 12px solid currentColor; */
  /* opacity:.8; */
  z-index: 994;
  /* box-shadow: rgba(0, 0, 0, 0.3) 0px 19px 38px, rgba(0, 0, 0, 0.22) 0px 15px 12px; */
}
.myMarker:hover, .myMarker.active {
  width: 100px;
  height: 100px;
  border-width: 0px;
  opacity:1;
  z-index: 995;
}
</style>