<template>
	<nav>
		<img alt="logo" src="@/assets/mandelivery.png" />
		<el-tag class="beta-label" type="warning" effect="dark" size="mini">Beta</el-tag>
	</nav>
</template>

<script>
export default {
	name: "Header"
}
</script>

<style scoped>
nav {
	padding-left: 2rem;
	--tw-bg-opacity: 1;
	background-color: rgba(255,255,255,var(--tw-bg-opacity));
	display: flex;
	height: 4.5rem;
	-webkit-box-align: center;
	align-items: center;
	-webkit-box-pack: justify;
	--tw-shadow: 0 10px 15px -3px rgba(0,0,0,0.1),0 4px 6px -2px rgba(0,0,0,0.05);
	box-shadow: var(--tw-ring-offset-shadow,0 0 #0000),var(--tw-ring-shadow,0 0 #0000),var(--tw-shadow);
	width: 100%;
	color: #a8a8b1;
	font-size: 18px;
}

img{
	height: 60%;
}

.beta-label {
	align-self: flex-end;
	font-weight: 400;
}

</style>
