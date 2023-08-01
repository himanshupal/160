<script lang="ts">
	import DatePicker from '$lib/components/DatePicker.svelte';

	let date: string;
</script>

<svelte:head>
	<title>Date Picker</title>
	<meta name="description" content="Custom Svelte Calendar" />
	<style lang="scss">
		@use '$lib/styles/vars' as v;

		:root {
			font-family: system-ui, consolas;
			input[type='date']::-webkit-inner-spin-button,
			input[type='date']::-webkit-calendar-picker-indicator {
				display: none;
				-webkit-appearance: none;
			}
		}

		body {
			margin: unset;
			background-color: v.$background-primary;
		}
	</style>
</svelte:head>

<div class="container">
	<form class="content">
		<DatePicker bind:date />
		<input type="date" placeholder="dd/mm/yyyy" class="input" bind:value={date} />
	</form>
</div>

<style lang="scss">
	@use '$lib/styles/vars' as v;

	.container {
		width: 100dvw;
		height: 100dvh;

		display: flex;
		align-items: center;
		justify-content: center;
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.input {
		border: 2px solid v.$highlight-primary;
		background-color: v.$background-secondary;
		color: v.$highlight-primary;
		border-radius: 0.5rem;
		width: min-content;
		text-align: center;
		font-size: 1.5rem;
		padding: 0.35rem;
		outline: none;
		flex: 2;
	}
</style>
