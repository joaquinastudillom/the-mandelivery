<template>
  <div id="app">

		<Header />

		<AppContent />

  </div>
</template>

<script>
import Header from "@/components/Header";
import AppContent from "@/components/Content";

export default {
  name: 'App',

  components: {
		Header,
		AppContent
  }
}
</script>

<style>
body {
	margin: 0;
	background: rgb(242, 244, 248);
	color: rgba(24,24,27,var(--tw-text-opacity));
	font-family: "Inter var", ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
}

.el-row {
	width: 100%;
}

.el-tag {
	margin-bottom: 10px;
}
</style>
