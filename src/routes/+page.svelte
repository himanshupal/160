<script lang="ts">
	import DatePicker from '$lib/components/DatePicker.svelte';
	import calendarIcon from '$lib/icons/calendar.svg';

	let date: string;
	let datePickerOpen = false;

	const toggleDatePicker = (e: MouseEvent) => {
		e.preventDefault();
		datePickerOpen = !datePickerOpen;
	};
</script>

<svelte:head>
	<title>Date Picker</title>
	<meta name="description" content="Custom Svelte Calendar" />
	<style lang="scss">
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
			background-color: #353535;
		}
	</style>
</svelte:head>

<div class="container">
	<form class="content">
		<span style="color: white">Select a date</span>
		<label for="dateinput" class="date-input__container">
			<img src={calendarIcon} alt="i" width="24px" height="24px" />
			<input
				type="date"
				placeholder="dd/mm/yyyy"
				on:click={toggleDatePicker}
				class="date-input"
				bind:value={date}
			/>
		</label>

		<DatePicker bind:date bind:datePickerOpen />
	</form>
</div>

<style lang="scss">
	$background-primary: #353535;

	.container {
		width: 100dvw;
		height: 65dvh;

		display: flex;
		align-items: center;
		justify-content: center;
	}

	.content {
		display: flex;
		flex-direction: column;
		position: relative;
	}

	.date-input {
		background-color: transparent;
		font-size: 1.25rem;
		color: white;
		border: none;
		outline: none;
		width: 100%;
		flex: 2;

		&__container {
			background-color: lighten($background-primary, 10%);
			width: calc(300px - 1rem);
			border-radius: 0.35rem;
			align-items: center;
			padding: 0.5rem;
			display: flex;
			gap: 0.25rem;
		}
	}
</style>
