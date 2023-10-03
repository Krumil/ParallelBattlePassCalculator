<template>
	<HelloWorld />
</template>

<script>
	import HelloWorld from './components/HelloWorld.vue';

	export default {
		name: 'App',
		components: {
			HelloWorld
		}
	};
</script>

<style>
	@font-face {
		font-family: 'Parallel'; /* You can name it whatever you like */
		src: url('~@/assets/font.woff2') format('woff2');
	}

	body {
		margin: 0;
		padding: 0;
	}

	* {
		font-family: 'Parallel', sans-serif;
	}

	#app {
		font-family: Avenir, Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		margin: 0;
		background-color: rgba(0, 0, 0, 0.9);
	}
</style>
