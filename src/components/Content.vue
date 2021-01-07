<template>
	<section class="main-container">
		<el-container>
			<el-row>
					<ContentData
							:is-optimizing="isOptimizing"
							:jobs="jobs"
							@add-job="addJob"
							@remove-job="removeJob"
							@optimize="calculateSequence"
							:here-api-key="hereApiKey"
					/>

				<el-col :span="16">
					<ContentMap
							ref="map"
							class="map"
							v-loading="isOptimizing"
							:latitude="startPoint.latitude"
							:longitude="startPoint.longitude"
							:here-api-key="hereApiKey"
					/>
				</el-col>
			</el-row>
		</el-container>

		<Footer />

	</section>
</template>

<script>
import ContentData from "@/components/ContentData";
import ContentMap from "@/components/ContentMap";
import Footer from "@/components/Footer";
import axios from "axios";

export default {
	name: "Content",

	components: {
		ContentMap,
		Footer,
		ContentData
	},

	mounted() {
		this.jobs = []
	},

	beforeDestroy() {
		this.jobs = []
	},

	data() {
		return {
			jobs: [],
			isOptimizing: false,
			startPoint: {
				latitude: 53.3227003,
				longitude: -6.2666459
			}
		}
	},

	methods: {
		async addJob(job){
			if(!this.jobs.some(address => address.id === job.id)){
				job.position = await this.getJobLatLng(job.id)
				this.jobs.push(job)
				this.$refs.map.dropMarker(job.position, job.id)
			}else {
				this.$message({
					message: 'address has been included',
					type: 'error'
				})
			}
		},

		getJobLatLng(id) {
			let endpoint = 'https://lookup.search.hereapi.com/v1/lookup'
			let config = {
				method: 'get',
				url: `${endpoint}?id=${id}&apiKey=${this.hereApiKey}`,
				headers: { }
			};
			return axios(config)
					.then(function (response) {
						return response.data.position
					})
					.catch(function (error) {
						console.log(error);
					});
		},

		async calculateSequence(){
			this.isOptimizing = true
			let sequence = await this.$refs.map.calculateRouteSequence(this.jobs);
			this.$refs.map.updateMarkerLabel(sequence)
			this.updateJobsOrder(sequence)
			this.$refs.map.drawRoute(sequence.map(e => {
				return {
					lng: e.location[0],
					lat: e.location[1]
				}
			}))
			this.isOptimizing = false
		},

		removeJob(job) {
			this.jobs.splice(this.jobs.indexOf(job), 1);
		},

		updateJobsOrder(optimizedJobs){
				let newArray = []
				for(let job of this.jobs){
					if(job.id){
						let orderedIndex = optimizedJobs.findIndex(e => e.id === job.id)
						newArray.splice(orderedIndex - 1, 0, job)
					}
				}
				this.jobs = newArray
		}
	},

	computed: {
		hereApiKey(){
			return process.env.VUE_APP_HERE_API_KEY
		}
	}
}
</script>

<style scoped>
.main-container {
	padding-left: 2rem;
	padding-right: 2rem;
	margin-top: 3rem;
}

.el-container {
	background-color: white;
	border: 1px solid #e4e4e7;
	--tw-shadow: 0 10px 15px -3px rgba(0,0,0,0.1),0 4px 6px -2px rgba(0,0,0,0.05);
	box-shadow: var(--tw-ring-offset-shadow,0 0 #0000),var(--tw-ring-shadow,0 0 #0000),var(--tw-shadow);
}

.map {
	width: 100%;
	height: 600px;
 }
</style>
