<script lang="ts">
	import { dayOffset, days } from '$lib/config';
	import dayjs from 'dayjs';

	interface IDateValue {
		active: boolean;
		value: number;
	}

	const d = dayjs();

	let slicedDateList: IDateValue[][];

	let currentMonth = d.month();
	let currentYear = d.year();
	let datePickerOpen = true;

	$: {
		const yearDiff = Math.floor((currentMonth - 12) / 12) + 1;
		currentYear = d.year(d.year() + yearDiff).year();
	}

	$: displayMonth = d.month(currentMonth).month() + 1;
	$: daysInMonth = d.month(currentMonth).daysInMonth();
	$: weekdays = dayOffset ? [...days.slice(dayOffset), ...days.slice(0, dayOffset)] : days;

	$: {
		const startAt = d.month(currentMonth).set('date', 1).day();
		const daysInPreviousMonth = d.month(currentMonth - 1).daysInMonth();
		let dateList: IDateValue[] = [
			...Array.from(
				{
					length: (() => {
						const x = startAt - dayOffset;
						return x > 0 ? x : 7 - dayOffset;
					})()
				},
				(_, i) => ({
					value: daysInPreviousMonth - i,
					active: false
				})
			).reverse(),
			...Array.from({ length: daysInMonth }, (_, i) => ({ value: i + 1, active: true }))
		];
		dateList = [
			...dateList,
			...Array.from({ length: 42 - dateList.length }, (_, i) => ({
				value: i + 1,
				active: false
			}))
		];
		slicedDateList = Array.from({ length: 6 }, (_, i) => dateList.slice(i * 7, i * 7 + 7));
	}

	const toggleDatePicker = () => {
		datePickerOpen = !datePickerOpen;
	};

	const previous = () => {
		currentMonth -= 1;
	};

	const next = () => {
		currentMonth += 1;
	};
</script>

<svelte:head>
	<title>Calendar</title>
	<meta name="description" content="Custom Svelte Calendar" />
	<style lang="scss">
		:root {
			font-family: system-ui, consolas;
		}
		body {
			margin: unset;
			background-color: #353535;
		}
	</style>
</svelte:head>

<div class="container">
	<form class="content">
		<input type="text" placeholder="dd/mm/yyyy" on:click={toggleDatePicker} class="date-input" />

		{#if datePickerOpen}
			<div class="datepicker">
				<div class="datepicker__actions">
					<span>{displayMonth} {currentYear}</span>
					<div class="datepicker__actions__buttons">
						<button on:click={previous}>&lt;</button>
						<button on:click={next}>&gt;</button>
					</div>
				</div>
				<div class="datepicker__row">
					{#each weekdays as day}
						<div class="datepicker__cell week">
							{day}
						</div>
					{/each}
				</div>
				{#each slicedDateList as week}
					<div class="datepicker__row">
						{#each week as date}
							<div class="datepicker__cell" class:active={date.active}>
								{date.value}
							</div>
						{/each}
					</div>
				{/each}
			</div>
		{/if}
	</form>
</div>

<style lang="scss">
	$background-primary: #353535;
	$text-color-primary: #ebebeb;
	$text-color-secondary: #686868;

	.container {
		display: flex;
		width: 100dvw;
		min-height: 100dvh;

		display: grid;
		place-content: center;
	}

	.content {
		display: flex;
		flex-direction: column;
		position: relative;
	}

	.date-input {
		background-color: lighten($background-primary, 10%);
		border-radius: 0.35rem;
		min-width: 17rem;
		padding: 0.5rem;
		color: white;
		border: none;
		outline: none;
		width: 100%;
	}

	.datepicker {
		position: absolute;
		top: calc(100% + 1rem);
		left: 50%;
		transform: translateX(-50%);

		display: flex;
		flex-direction: column;

		max-width: max-content;
		max-height: max-content;

		color: $text-color-primary;
		padding: 0.5rem;
		border-radius: 0.25rem;
		border: 2px solid lighten($background-primary, 2.5%);
		background-color: darken($background-primary, 3%);

		&__actions {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 0.5rem;

			&__buttons {
				display: flex;
				gap: 1rem;
			}
		}

		&__row {
			width: 100%;
			display: flex;
			gap: 0.25rem;
		}

		&__cell {
			flex: 1;
			width: 2rem;
			height: 2rem;
			display: grid;
			place-content: center;
			transition: all 75ms ease-in-out;

			&:not(&.week) {
				cursor: pointer;

				&:hover {
					background-color: lighten($background-primary, 5%);
					color: $text-color-primary !important;
					border-radius: 100rem;
				}

				&:not(&.active) {
					color: $text-color-secondary;
				}
			}
		}
	}
</style>
