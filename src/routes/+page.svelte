<script lang="ts">
	import { dayOffset, days, months, yearsRange } from '$lib/config';
	import calendarIcon from '$lib/icons/calendar.svg';
	import downIcon from '$lib/icons/down.svg';
	import upIcon from '$lib/icons/up.svg';
	import dayjs from 'dayjs';

	interface ICellValue {
		text: string | number;
		active: boolean;
		value: number;
	}

	const d = dayjs();

	let slicedDateList: ICellValue[][];
	let slicedMonthList: ICellValue[][];
	let slicedYearList: ICellValue[][];

	let selectionStage: 0 | 1 | 2 = 0;
	let currentMonth = d.month();
	let currentDate = d.date();
	let datePickerOpen = true;
	let selected = dayjs();

	/////////////////// Reactive values below this ///////////////////

	$: currentYear = d.month(currentMonth).year();
	$: farthestYearInPast = d.year() - yearsRange;
	$: isActiveMonth = !((currentMonth % 12) - selected.month());
	$: weekdays = dayOffset ? [...days.slice(dayOffset), ...days.slice(0, dayOffset)] : days;

	$: decadeRange = (() => {
		let year = currentYear % 100;
		year -= year % 10;
		let startOfDecade = +`${Math.floor(currentYear / 100)}${String(year).padEnd(2, '0')}`;
		if (startOfDecade < farthestYearInPast) startOfDecade = farthestYearInPast;
		return { start: startOfDecade, end: startOfDecade + 9 };
	})();

	$: {
		const start = currentYear - d.year();
		const monthsList: ICellValue[] = [
			...months.map((value, i) => ({
				active: true,
				value: i + start * 12,
				text: value
			})),
			...months.slice(0, 4).map((value, i) => ({
				active: false,
				value: i + start * 12 + 12,
				text: value
			}))
		];
		slicedMonthList = Array.from({ length: 4 }, (_, i) => monthsList.slice(i * 4, i * 4 + 4));
	}

	$: {
		const { start, end } = decadeRange;
		let startIndex = (start - farthestYearInPast) % 4;
		let yearsList: ICellValue[] = [
			...Array.from({ length: startIndex }, (_, i) => ({
				active: false,
				text: start - (i + 1),
				value: start - (i + 1)
			})).reverse(),
			...Array.from({ length: 10 }, (_, i) => ({ active: true, value: start + i, text: start + i }))
		];
		yearsList = [
			...yearsList,
			...Array.from({ length: 16 - yearsList.length }, (_, i) => ({
				active: false,
				text: end + (i + 1),
				value: end + (i + 1)
			}))
		];
		slicedYearList = Array.from({ length: 4 }, (_, i) => yearsList.slice(i * 4, i * 4 + 4));
	}

	$: {
		const daysInMonth = d.month(currentMonth).daysInMonth();
		const startAt = d.month(currentMonth).set('date', 1).day();
		const daysInPreviousMonth = d.month(currentMonth - 1).daysInMonth();
		let dateList: ICellValue[] = [
			...Array.from(
				{
					length: (() => {
						const x = startAt - dayOffset;
						return x > 0 ? x : 7 - dayOffset;
					})()
				},
				(_, i) => ({
					value: -i,
					text: daysInPreviousMonth - i,
					active: false
				})
			).reverse(),
			...Array.from({ length: daysInMonth }, (_, i) => ({
				value: i + 1,
				text: i + 1,
				active: true
			}))
		];
		dateList = [
			...dateList,
			...Array.from({ length: 42 - dateList.length }, (_, i) => ({
				value: daysInMonth + i + 1,
				text: i + 1,
				active: false
			}))
		];
		slicedDateList = Array.from({ length: 6 }, (_, i) => dateList.slice(i * 7, i * 7 + 7));
	}

	/////////////////// Methods below this ///////////////////

	const toggleDatePicker = () => (datePickerOpen = !datePickerOpen);

	const previous = () => {
		switch (selectionStage) {
			case 0:
				return (currentMonth -= 1);
			case 1:
				return (currentMonth -= 12);
			case 2:
				return (currentMonth -= 120);
		}
	};

	const next = () => {
		switch (selectionStage) {
			case 0:
				return (currentMonth += 1);
			case 1:
				return (currentMonth += 12);
			case 2:
				return (currentMonth += 120);
		}
	};

	const updateSelectionStage = () => {
		if (selectionStage < 2) {
			selectionStage += 1;
		}
	};

	const setCurrentYear = (value: number) => (e: MouseEvent) => {
		currentMonth += (value - currentYear) * 12;
		selectionStage -= 1;
	};

	const setCurrentMonth = (value: number) => (e: MouseEvent) => {
		currentMonth = value;
		selectionStage -= 1;
	};

	const setCurrentDate = (value: number, isActive: boolean) => (e: MouseEvent) => {
		if (isActive) {
			// Date selected is from current month
			currentDate = value;
		} else if (value > 0) {
			// Date selected is from next month
			const daysInMonth = d.month(currentMonth).daysInMonth();
			currentMonth += 1;
			currentDate = value - daysInMonth;
		} else {
			// Date selected is from previous month
			currentMonth -= 1;
			const daysInMonth = d.month(currentMonth).daysInMonth();
			currentDate = daysInMonth + value;
		}

		// No need to explicitly set years since months already have that difference accounted for
		selected = dayjs().month(currentMonth).date(currentDate);
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
		<label for="dateinput" class="date-input__container">
			<img src={calendarIcon} alt="i" width="24px" height="24px" />
			<input type="text" placeholder="dd/mm/yyyy" on:click={toggleDatePicker} class="date-input" />
		</label>

		{#if datePickerOpen}
			<div class="datepicker">
				<div class="datepicker__actions">
					<button class="datepicker__actions__selector" on:click={updateSelectionStage}>
						{#if !selectionStage}
							{d.month(currentMonth).format('MMMM')}
						{/if}
						{selectionStage === 2 ? `${decadeRange.start} - ${decadeRange.end}` : currentYear}
					</button>

					<div class="datepicker__actions__buttons">
						<button
							on:click={previous}
							disabled={farthestYearInPast === slicedYearList[0][0].value}
						>
							<img src={upIcon} alt="<" width="24px" height="24px" />
						</button>
						<button on:click={next}>
							<img src={downIcon} alt=">" width="24px" height="24px" />
						</button>
					</div>
				</div>

				{#if !selectionStage}
					<div class="datepicker__row">
						{#each weekdays as day}
							<div class="datepicker__cell nohover">
								{day}
							</div>
						{/each}
					</div>
					{#each slicedDateList as week}
						<div class="datepicker__row">
							{#each week as date}
								<div
									role="none"
									class="datepicker__cell"
									class:active={date.active}
									on:click={setCurrentDate(date.value, date.active)}
									class:selected={isActiveMonth && date.active && date.value === selected.date()}
								>
									{date.text}
								</div>
							{/each}
						</div>
					{/each}
				{:else if selectionStage === 1}
					{#each slicedMonthList as monthList}
						<div class="datepicker__row">
							{#each monthList as month}
								<div
									role="none"
									class="datepicker__cell lg"
									class:active={month.active}
									class:selected={month.value === selected.month() && month.active}
									on:click={setCurrentMonth(month.value)}
								>
									{month.text}
								</div>
							{/each}
						</div>
					{/each}
				{:else if selectionStage === 2}
					{#each slicedYearList as yearList}
						<div class="datepicker__row">
							{#each yearList as year}
								<div
									role="none"
									class="datepicker__cell lg"
									class:active={year.active}
									class:selected={year.value === selected.year() && year.active}
									on:click={setCurrentYear(year.value)}
								>
									{year.text}
								</div>
							{/each}
						</div>
					{/each}
				{/if}
			</div>
		{/if}
	</form>
</div>

<style lang="scss">
	$background-primary: #353535;
	$background-secondary: #49b3eb;

	$text-color-primary: #ebebeb;
	$text-color-secondary: #686868;

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
		max-width: max-content;
		font-size: 1.25rem;
		min-width: 17rem;
		color: white;
		border: none;
		outline: none;
		width: 100%;
		flex: 2;

		&__container {
			background-color: lighten($background-primary, 10%);
			border-radius: 0.35rem;
			align-items: center;
			padding: 0.5rem;
			display: flex;
			gap: 0.25rem;
		}
	}

	.datepicker {
		position: absolute;
		top: 100%;
		right: 0%;

		gap: 0.25rem;
		display: flex;
		flex-direction: column;

		max-width: max-content;
		max-height: max-content;

		color: $text-color-primary;
		padding: 0.5rem;
		border-radius: 0.5rem;
		border: 2px solid lighten($background-primary, 2.5%);
		border-top: 0;
		border-top-left-radius: 0;
		border-top-right-radius: 0;
		background-color: darken($background-primary, 3%);

		&__actions {
			gap: 0.5rem;
			display: flex;
			align-items: stretch;
			justify-content: space-between;

			&__selector {
				background-color: darken($background-primary, 3%);
				transition: all 75ms ease-in-out;
				border-radius: 0.35rem;
				font-family: inherit;
				font-size: medium;
				text-align: left;
				font-weight: 600;
				cursor: pointer;
				padding: 0.5rem;
				color: inherit;
				border: unset;
				flex: 1;

				&:hover {
					background-color: lighten($background-primary, 3%);
				}
			}

			&__buttons {
				align-items: stretch;
				display: flex;
				gap: 0.5rem;

				> button {
					border: none;
					padding: unset;
					cursor: pointer;
					background-color: transparent;
					transition: all 75ms ease-in-out;
					border-radius: 0.35rem;

					&:hover {
						background-color: lighten($background-primary, 3%);
					}
				}
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

			&.nohover {
				color: darken($text-color-primary, 25%);
				font-weight: 600;
			}

			&.lg {
				width: 3.688rem;
				height: 3.688rem;
			}

			&:not(&.nohover) {
				cursor: pointer;

				&:hover {
					background-color: lighten($background-primary, 5%);
					color: $text-color-primary !important;
					border-radius: 100rem;
				}

				&.selected {
					background-color: $background-secondary;
					color: $background-primary !important;
					border-radius: 100rem;
				}

				&:not(&.active) {
					color: $text-color-secondary;
				}
			}
		}
	}
</style>
