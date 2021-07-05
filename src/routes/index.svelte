<script>
      import { Map, Geocoder, controls } from '@beyonk/svelte-mapbox';
      import Marker from '$lib/Marker.svelte';
      const { GeolocateControl, NavigationControl, ScaleControl } = controls;
      import {afterUpdate, getContext, onMount, setContext} from 'svelte';
      import LoadingSpinner from '$lib/LoadingSpinner.svelte';


      export let sites;
      let center = [-74.5, 40];
      let mapComponent;
      let zoom = 3;
      let type = "plant_natives_here";
      let natives_list = "Species: \r\n \r\n - White clover (Trifolium repens) \r\n - Yarrow (Achillea millefolium ) \r\n - Nettle (Urtica dioica)";
      $: lng = center[0];
      $: lat = center[1];

      $: lng_display = center[0].toFixed(4);
      $: lat_display = center[1].toFixed(4);
      // $: style = 'mapbox://styles/mapbox/streets-v11'
      $: style = 'mapbox://styles/mapbox/satellite-v9'
      // $: style = 'mapbox://styles/mapbox/satellite-streets-v11'
      let menu_display = "plant";

      let display_notification = false;
      let notification_text = "You have local gardeners and seed shares nearby who can help.  Check the map to connect.";


      let opening_modal = true;


      let local_species_loaded = true;


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

          setTimeout(function() {
            if (zoom > 10) {
              local_species_loaded = true;
            }
      }, 750)          
      })


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

if (menu_display == "seed_share") {
  title = "Seeds and plants here";
  notification_text = "You have plant sites and local gardeners nearby.  Check the map to connect."
}

else if (menu_display == "peer") {
  title = "Willing to help others"
  notification_text = "You have seed shares and plant sites nearby.  Check the map to connect.";
}

else if (menu_display == "plant_learn") {
  title = "Learning to plant";
  notification_text = "You have seed shares and local gardeners nearby.  Check the map to connect.";
}

else if (menu_display == "plant_with_peers") {
  title = "Planting with peers";
  notification_text = "You have seed shares and local gardeners nearby.  Check the map to connect.";
}

else if (menu_display == "plant_here") {
  title = "Plant here!";
  notification_text = "You have seed shares and local gardeners nearby.  Check the map to connect.";
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
  // menu_display = "plant";

  display_notification = true;
  menu_display = "plant";

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

<!-- <h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p> -->

<!-- <button on:click={publishLocation}>Publish location</button> -->

{#if opening_modal == true}
<div class="opening_modal">
  <!-- <p>Welcome to</p> -->
  <div class="modal-content" style="position: absolute; top: 20%; text-align: center; padding: 20px;">
  <h1 style="margin-left: -20px;">🌱 Native</h1>
  <p style="font-size: 20px;">Walk to an outdoor place and share your location.</p>
  <button style="font-size: 18px; background: #d2f7a1;" on:click={function() {opening_modal = false;}}>🌿 I'm outside 🌿</button>
  </div>
</div>
{/if}

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

  <div class="info_panel">
    {#if local_species_loaded == true}
    <p style="margin-bottom: 5px;">[{lng.toFixed(4)}, {lat.toFixed(4)}]</p>
    <p style="margin-top: 10px;">Local Species:</p>
    <ul style="text-align: left; margin-left: -15px;">
      <li style="margin-bottom: 10px;">
        <!-- <img loading="lazy" style="width: 30px; height: auto;" src="https://www.gardenia.net/storage/app/public/uploads/images/detail/xeOghGISr6OZa8xAJ0wyWVyxzjNvLAbT8e81uo9i.jpeg" />  -->
        White clover (Trifolium repens)</li>
      <li style="margin-bottom: 10px;">Yarrow (Achillea millefolium)</li>
      <li>Nettle (Urtica dioica)</li>
    </ul>
    {/if}
    {#if local_species_loaded == false}
    <LoadingSpinner></LoadingSpinner>
    {/if}
  </div>

  {#if display_notification == true}
  <div class="notifications">
    <!-- <svg on:click={function() { display_notification = false; }} xmlns="http://www.w3.org/2000/svg" style="position: absolute; top: 10%; right: 5%;" class="icon icon-tabler icon-tabler-x" width="14" height="14" viewBox="0 0 24 24" stroke-width="1.5" stroke="#ffffff" fill="none" stroke-linecap="round" stroke-linejoin="round">
      <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
      <line x1="18" y1="6" x2="6" y2="18" />
      <line x1="6" y1="6" x2="18" y2="18" />
    </svg> -->
    <!-- <svg xmlns="http://www.w3.org/2000/svg" style="position: absolute; top: 5%; right: 5%;" class="icon icon-tabler icon-tabler-x" width="28" height="28" viewBox="0 0 24 24" stroke-width="1.5" stroke="#ffffff" fill="none" stroke-linecap="round" stroke-linejoin="round">
      <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
      <line x1="18" y1="6" x2="6" y2="18" />
      <line x1="6" y1="6" x2="18" y2="18" />
    </svg> -->
    <div class="update">
    <p style="padding-left: 3px;">
      <svg style="vertical-align: middle" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-circle-check" width="28" height="28" viewBox="0 0 24 24" stroke-width="1.5" stroke="#92de00" fill="none" stroke-linecap="round" stroke-linejoin="round">
        <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
        <circle cx="12" cy="12" r="9" />
        <path d="M9 12l2 2l4 -4" />
      </svg>
      <span style="color: #92de00; font-size: 16px;">Posted!</span>
    </p>
    </div>
    <div class="body">
      <p style="padding: 0px 15px; font-size: 16px;">
        {notification_text}
      </p>
      <button on:click={function() { display_notification = false; }} style="margin: auto; text-align: center; display: block; margin-bottom: 20px; background: #e6e150;">
        <svg style="vertical-align: text-bottom" xmlns="http://www.w3.org/2000/svg" class="icon icon-tabler icon-tabler-hand-rock" width="20" height="20" viewBox="0 0 24 24" stroke-width="1.5" stroke="#525252" fill="none" stroke-linecap="round" stroke-linejoin="round">
          <path stroke="none" d="M0 0h24v24H0z" fill="none"/>
          <path d="M11 11.5v-1a1.5 1.5 0 0 1 3 0v1.5" />
          <path d="M17 12v-6.5a1.5 1.5 0 0 1 3 0v10.5a6 6 0 0 1 -6 6h-2h.208a6 6 0 0 1 -5.012 -2.7a69.74 69.74 0 0 1 -.196 -.3c-.312 -.479 -1.407 -2.388 -3.286 -5.728a1.5 1.5 0 0 1 .536 -2.022a1.867 1.867 0 0 1 2.28 .28l1.47 1.47" />
          <path d="M14 10.5a1.5 1.5 0 0 1 3 0v1.5" />
          <path d="M8 13v-8.5a1.5 1.5 0 0 1 3 0v7.5" />
        </svg>
        <span style="color: #525252; font-size: 16px; display: inline-block; padding: 5px;">Got it!</span>
      </button>
    </div>
  </div>
  {/if}


  <div class="interaction" style="position: absolute; bottom: 15%">
    {#if menu_display.substr(0,5) == "plant"}
    <button style="display: block;" on:click={function() {if (menu_display == "plant_learn") { menu_display = "plant"} else { menu_display = "plant_learn"} }}>Learn to plant</button>
    {#if menu_display == "plant_learn"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">Below is a list of local natives.  Add any other information you wish to share with others.</label>
      <textarea name="content" bind:value={natives_list}></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {if (menu_display == "plant_with_peers") { menu_display = "plant"} else { menu_display = "plant_with_peers"} }}>Plant with peers</button>
    {#if menu_display == "plant_with_peers"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">Below is a list of local natives.  Add any other information you wish to share with others.</label>
      <textarea name="content" bind:value={natives_list}></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {if (menu_display == "plant_here") { menu_display = "plant"} else {menu_display = "plant_here"} }}>Someone, plant here!</button>
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
    <button style="display: block;" on:click={function() {if (menu_display == "seed_share") { menu_display = "offers"} else {menu_display = "seed_share"} }}>Seeds + plants to share.</button>
    {#if menu_display == "seed_share"}
    <form style="background: white;" on:submit|preventDefault={publishSite}>
      <label for="content">List the seeds and plants you're willing to share, and anything else you'd like others to know.</label>
      <textarea name="content"></textarea>
      <button>Submit</button>
    </form>
    {/if}
    <button style="display: block;" on:click={function() {if (menu_display == "peer") { menu_display = "offers"} else { menu_display = "peer"} }}>I'm a planter, willing to help others.</button>
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
    height: 100vh;
    /* height: 80vh; */
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

  .notifications {
    width: 80vw;
    margin: auto;
    color: white; 
    position: absolute; 
    top: 1%; 
    /* background:#50BFE6;  */
    background: #7c50e6;
    left: 50%; 
    transform: translate(-50%, 50%);
    border-radius: 10px;
  }

  .info_panel {
    /* width: 40vw;
    margin: auto; */
    text-align: center;
    width: 40vw;
    color: white; 
    position: absolute; 
    top: 5%; 
    /* background:#50BFE6;  */
    /* background: rgba(124,80,230, 0.3);
    border: solid 1px #7c50e6; */
    background: rgba(193,230,80, 0.2);
    border: solid 1px rgb(193 230 80);
    right: 5%; 
    /* transform: translate(-20%, 20%); */
    border-radius: 10px;
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

.opening_modal {
  position: absolute;
  height: 100vh;
  width: 100vw;
  background: black;
  z-index: 100;
  color: white;
}
</style>