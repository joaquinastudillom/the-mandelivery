<template>
	<div ref="map"></div>
</template>

<script>
import axios from "axios"

export default {
	name: "LeafletMap",

	data() {
		return {
			platform: {},
			map: {},
			sequenceMarkerGroup: {},
			routesGroup: {}
		}
	},

	props: {
		hereApiKey: {
			type: String,
			required: true
		},
		latitude: {
			type: Number,
			required: true
		},
		longitude: {
			type: Number,
			required: true
		}
	},

	created() {
		this.sequenceMarkerGroup = new window.H.map.Group();
		this.routesGroup = new window.H.map.Group();
	},

	async mounted() {
		const platform = new window.H.service.Platform({
			'apikey': this.hereApiKey
		});

		const layers = platform.createDefaultLayers();

		const map = new window.H.Map(
				this.$refs.map,
				layers.vector.normal.map,
				{
					center: {lat: this.latitude, lng: this.longitude},
					zoom: 10,
					pixelRatio: window.devicePixelRatio || 1
				},
		);

		window.H.ui.UI.createDefault(map, layers);

		this.map = map;

		this.dropStartingPointMarker()
	},

	methods: {
		dropMarker(position, jobId) {
			let coords = {lat: position.lat, lng: position.lng}
			this.map.addObject(this.sequenceMarkerGroup);
			let marker = new window.H.map.Marker(coords, {
				data: {
					jobId: jobId
				}
			});
			this.sequenceMarkerGroup.addObject(marker)
		},

		dropStartingPointMarker(){
			let coords = {lat: this.latitude, lng: this.longitude}
			let svgMarkup = `<svg xmlns="http://www.w3.org/2000/svg" width="50" height="50" font-family="Arial, Helvetica, sans-serif">
                <circle cx="25" cy="25" r="15" fill="#7e2c36" />
                <text x="50%" y="50%" text-anchor="middle" fill="white" font-size="12px" dy=".3em">TH</text>
            </svg>`

			let newIcon = new window.H.map.Icon(svgMarkup)
			let marker = new window.H.map.Marker(coords, {icon: newIcon});
			this.map.addObject(marker);
		},

		async calculateRouteSequence(places) {
			let jobPlaces = {
				jobs: []
			}
			let vehicles = {
				vehicles: [
					{
						id: 1,
						profile: 'driving-car',
						start: [this.longitude, this.latitude],
						capacity: [
							7
						],
						skills: [
							1
						]
					}
				]
			}

			let id = 1
			for (let job of places) {
				jobPlaces.jobs.push({
					id: id,
					jobId: job.id,
					amount: [1],
					location: [job.position.lng, job.position.lat],
					skills: [1]
				})
				id++
			}

			let data = JSON.stringify({
				...jobPlaces,
				...vehicles
			});

			let config = {
				method: 'post',
				url: 'https://api.openrouteservice.org/optimization',
				headers: {
					'Authorization': this.openServiceApiKey,
					'Content-Type': 'application/json'
				},
				data: data
			};

			return axios(config).then(response => {
				let steps = response.data.routes[0].steps
				for (let step of steps) {
					if (jobPlaces.jobs.find(element => element.id === step.id)) {
						step.id = jobPlaces.jobs.find(element => element.id === step.id).jobId
					}
				}
				return steps
			});
		},

		drawRoute(sequence) {
			this.map.addObject(this.routesGroup);
			this.map.getViewModel().setLookAtData({
				bounds: this.sequenceMarkerGroup.getBoundingBox()
			});

			let viaString = ''
			let startPoint = sequence.shift()
			let endPoint = sequence.pop()
			for (let i = 0; i < sequence.length; i++) {
				viaString = viaString + '&via=' + `${sequence[i].lat},${sequence[i].lng}`;
			}

			let endpoint = 'https://router.hereapi.com/v8/routes', origin = `${startPoint.lat},${startPoint.lng}`,
					destination = `${endPoint.lat},${endPoint.lng}`

			let url = `${endpoint}?origin=${origin}&transportMode=car&destination=${destination}${viaString}&return=polyline&apiKey=${this.hereApiKey}`

			axios({
				"method": "GET",
				"url": url
			}).then(result => {
				this.routesGroup.removeAll()
				result.data.routes[0].sections.forEach(section => {
					let linestring = window.H.geo.LineString.fromFlexiblePolyline(section.polyline);
					let polyline = new window.H.map.Polyline(linestring, {
						style: {
							lineWidth: 4,
							strokeColor: 'rgba(0, 128, 255, 0.7)'
						}
					});
					this.routesGroup.addObject(polyline)
				})
			}, error => {
				console.error(error);
			});
		},

		updateMarkerLabel(jobs) {
			let markers = this.sequenceMarkerGroup.getObjects()
			for (let marker of markers) {
				let index = jobs.findIndex(element => element.id === marker.data.jobId)
				var svgMarkup = `<svg xmlns="http://www.w3.org/2000/svg" width="50" height="50" font-family="Arial, Helvetica, sans-serif">
															<circle cx="25" cy="25" r="10" fill="#7e2c36" />
															<text x="50%" y="50%" text-anchor="middle" fill="white" font-size="12px" dy=".3em">${index}</text>
													</svg>`

				var newIcon = new window.H.map.Icon(
						svgMarkup.replace('${FILL}', 'blue').replace('${STROKE}', 'red'))
				marker.setIcon(newIcon)
			}
		}
	},

	computed: {
		openServiceApiKey(){
			return process.env.VUE_APP_OPEN_SERVICE_API_KEY
		}
	}
}
</script>

<style scoped>
</style>
