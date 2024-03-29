<script>
	import { onMount } from 'svelte';
	import { Map } from '@beyonk/svelte-mapbox';
	import { page, navigating } from '$app/stores';

	import SatelliteImagery from '$lib/components/map/layers/SatelliteImagery.svelte';
	import LocationLines from '$lib/components/map/layers/LocationLines.svelte';
	import Head from '$lib/components/Head.svelte';
	import SafetyDisclaimerModal from '$lib/components/SafetyDisclaimerModal.svelte';

	import { elsa } from '$lib/simulation/elsa';
	import { aircraftID } from '$lib/stores';

	onMount(async () => {
		await elsa.startup;
	});

	let innerWidth = 0;

	// Since child components are only mounted once the map is rendered, we have to kinda cheese our way into setting the correct header :(
	function calculateHead(page) {
		const constants = {
			'/tool': {},
			'/tool/reachability': {
				title: 'Reachability analysis tool',
				description:
					'A map that displays all known, viable emergency landing sites and reachability ranges for each of them. It may aid in planning departure / arrival routes so that you do not pass through areas with no fields in range.'
			},
			'/tool/location': {
				title: 'Location details',
				description:
					'A detailed analysis of an emergency landing site in the area — includes available length, directions, surface type, imagery, chance of human presence and other details.'
			},
			'/tool/route': {
				title: 'Secret alpha tool',
				description: "How did you find this?! Either way, it is pre-alpha so don't you dare use it!"
			}
		};

		if (page.path.startsWith('/tool/location')) {
			// TODO Fetch social card images based on the location ID
			return constants['/tool/location'];
		} else {
			return constants[page.path];
		}
	}

	$: headProperties = calculateHead($page);
</script>

<svelte:window bind:innerWidth />
{#if headProperties}
	<Head {...headProperties} />
{/if}

<Map
	accessToken="pk.eyJ1IjoidGlsYmxlY2hzY2htaWR0IiwiYSI6ImNqczYxZXplZjA3bnM0M3A5djB1cDl3azUifQ.MEU9Fe4JHD1_3U1BLNJWbg"
	style="mapbox://styles/tilblechschmidt/ckraoako74wms18mx5xv38zea/draft"
	center={[9.99, 53.95]}
	zoom={7.5}
	options={{
		customAttribution: ['Til Blechschmidt', 'LGV Hamburg'],
		maxPitch: 0,
		bearingSnap: 180
	}}
>
	<SatelliteImagery />
	<LocationLines aircraft={$aircraftID} />
	<slot />
</Map>

<SafetyDisclaimerModal />

{#if innerWidth < 650}
	<div class="absolute flex items-center justify-center h-full w-full z-50 top-0 left-0 bg-white">
		<div class="p-8 text-center mx-auto">
			<h1 class="text-2xl">Uh oh.</h1>
			Your screen is too narrow to use E.L.S.A.!<br />
			Either rotate your device or use another one.
		</div>
	</div>
{/if}

<style global>
	body {
		overflow: hidden;
	}
</style>
