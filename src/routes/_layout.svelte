<script>
	import Header from '../components/UI/Header.svelte'
</script>

<Header />

<main>
	<slot></slot>
</main>

<style>
	main {
		margin-top: 4rem;
	}
</style>