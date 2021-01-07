<template>
	<el-col :span="8">
		<el-row class="jobs-data">
			<section>
				<el-row class="job-item">
					<label>Add Job</label>
					<el-select
							v-model="newJob"
							filterable
							remote
							reserve-keyword
							placeholder="Add a location"
							:remote-method="remoteMethod"
							:loading="loading"
							@change="addJob"
					>
						<el-option
								v-for="item in options"
								:key="item.id"
								:label="item.title"
								:value="item">
						</el-option>
					</el-select>
				</el-row>

				<el-row class="job-item" v-if="jobs.length > 0">
					<label>Locations</label>
					<div class="jobs-locations" v-loading="loading">
						<el-tag
								v-for="(job, index) in jobs"
								type="info"
								:key="index"
								closable
								:disable-transitions="false"
								@close="removeJob(job)">
							<div>
								<sup class="el-badge__content el-badge__content--primary">{{index + 1}}</sup>
								{{job.title}}
							</div>
						</el-tag>
					</div>
				</el-row>

			</section>
		</el-row>

		<el-row class="search-btn-container">
			<el-col>
				<el-button @click="runOptimization" class="search-btn" type="primary" :loading="isOptimizing" :disabled="jobs.length < 1">Optimize route</el-button>
			</el-col>
		</el-row>
	</el-col>
</template>

<script>
import axios from "axios";

export default {
	name: "ContentData",

	props: {
		hereApiKey: {
			type: String,
			required: true
		},
		jobs: {
			type: Array,
			required: true
		},
		isOptimizing: {
			type: Boolean,
			required: true
		}
	},

	data(){
		return {
			newJob: '',
			options: [],
			loading: false
		}
	},

	methods: {
		async remoteMethod(query) {
			let endpoint = 'https://autocomplete.search.hereapi.com/v1/autocomplete'
			if (query !== '') {
				this.loading = true;
				var config = {
					method: 'get',
					url: `${endpoint}?q=${query}&resultType=place&apiKey=${this.hereApiKey}`
				};
				this.options = await axios(config)
						.then(function (response) {
							return response.data.items
						})
						.catch(function (error) {
							console.log(error);
						});
				this.loading = false;
			} else {
				this.options = [];
			}
		},

		async addJob(job){
			await this.$emit('add-job', job)
			this.newJob = ''
			this.options = []
		},

		removeJob(job) {
			this.$emit('remove-job', job)
		},

		runOptimization(){
			this.$emit('optimize')
		}
	}
}
</script>

<style scoped>
.jobs-data {
	max-height: 500px;
	overflow-y: auto;
}

.jobs-data section {
	padding: 30px;
}

.job-item {
	display: flex;
	flex-direction: column;
	margin-bottom: 20px;
}

.job-item label {
	margin-bottom: 1rem;
	font-weight: 600;
	font-size: 12px;
}

.jobs-locations {
	display: flex;
	flex-direction: column;
	align-items: baseline;
}

.search-btn-container {
	position: absolute;
	bottom: 0px;
	left: 0px;
	padding: 1.5rem 2rem;
	z-index: 10;
	width: 33.3333%;
	--tw-shadow: 0 10px 15px -3px rgba(0,0,0,0.1),0 4px 6px -2px rgba(0,0,0,0.05);
	box-shadow: var(--tw-ring-offset-shadow,0 0 #0000),var(--tw-ring-shadow,0 0 #0000),var(--tw-shadow);
}

.search-btn {
	width: 100%;
}

>>> .el-tag.el-tag--info {
	background: #f4f4f5;
	outline: none;
	box-shadow: none;
	border: none;
	font-weight: 400;
}

>>> .el-tag.el-tag--info:hover {
	background-color: rgb(242, 244, 248);
}

>>> .el-tag.el-tag--info .el-tag__close:hover {
	background-color: #e8e8e8;
	color: #909399;
}

>>> .el-input__inner {
	background: #f4f4f5;
	outline: none;
	box-shadow: none;
	border: none;
	font-size: 13px;
	font-weight: 400;
}

.el-tag {
	display: flex;
	align-items: center;
}

.jobs-locations .el-tag div {
	display: flex;
	align-items: center;
}

.el-badge__content--primary {
	margin-right: 5px;
}
</style>
