<script>
	import { onMount, onDestroy, getContext } from 'svelte';
	import { contextKey } from '@beyonk/svelte-mapbox';

	const { getMap, getMapbox } = getContext(contextKey);
	const map = getMap();
	const mapbox = getMapbox();

	function randomColour() {
		return Math.round(Math.random() * 255);
	}

	function move(lng, lat) {
		marker.setLngLat({ lng, lat });
	}

	export let lat;
	export let lng;
	export let label = 'Marker';
	export let popupClassName = 'beyonk-mapbox-popup';
	export let markerOffset = [0, 0];
	export let popupOffset = 10;
	export let color = randomColour();
	export let popup = true;
	export let popupOptions = {};
	export let markerOptions = {};

	let marker;
	let element;
	let elementPopup;

	$: marker && move(lng, lat);

	onMount(() => {
		const namedParams = Object.assign(
			{
				offset: markerOffset
			},
			element.hasChildNodes() ? { element } : { color }
		);
		marker = new mapbox.Marker(Object.assign(namedParams, markerOptions));

		if (popup) {
			const namedPopupParams = { offset: popupOffset, className: popupClassName };
			const popupEl = new mapbox.Popup(Object.assign(namedPopupParams, popupOptions));
			if (elementPopup.hasChildNodes()) {
				popupEl.setDOMContent(elementPopup);
			} else {
				popupEl.setText(label);
			}

			marker.setPopup(popupEl);
		}

		marker.setLngLat({ lng, lat }).addTo(map);
	});

	onDestroy(() => {
		// Asynchronously remove the marker so Svelte does not complain about the component being gone
		setTimeout(() => marker.remove(), 1);
	});

	export function getMarker() {
		return marker;
	}
</script>

<div bind:this={element}>
	<slot />
</div>

<div class="popup" bind:this={elementPopup}>
	<slot name="popup" />
</div>
