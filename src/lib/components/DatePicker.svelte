<script lang="ts">
	import { dayOffset, days, months, yearsRange } from '$lib/config';
	import type { ICellValue, IDecade } from '../../types';
	import downIcon from '$lib/icons/down.svg';
	import upIcon from '$lib/icons/up.svg';
	import { onMount } from 'svelte';
	import Row from './Row.svelte';
	import dayjs from 'dayjs';

	/////////////////// Props below this ///////////////////

	export let date: string;
	export let datePickerOpen: boolean;

	/////////////////// Vars declared below this ///////////////////

	const d = dayjs();
	const weekdays = dayOffset ? [...days.slice(dayOffset), ...days.slice(0, dayOffset)] : days;

	let datePickerUI: HTMLDivElement;
	let slicedDateList: ICellValue[][];
	let slicedMonthList: ICellValue[][];
	let slicedYearList: ICellValue[][];

	let selectionStage: 0 | 1 | 2 = 0;
	let currentMonth = d.month();
	let currentDate = d.date();
	let selected = dayjs();

	/////////////////// Event listeners below this ///////////////////

	const clickAwayListener = (e: MouseEvent) => {
		const target = e.target as HTMLDivElement;
		if (target !== datePickerUI) {
			if (target.contains(datePickerUI) && target !== datePickerUI) {
				datePickerOpen = false;
			}
		}
	};

	onMount(() => {
		window.addEventListener('click', clickAwayListener);
		return () => {
			window.removeEventListener('click', clickAwayListener);
		};
	});

	/////////////////// Reactive values below this ///////////////////

	$: date = selected.format('YYYY-MM-DD');
	$: currentYear = d.month(currentMonth).year();
	$: farthestYearInPast = d.year() - yearsRange;

	$: decadeRange = ((): IDecade => {
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

	$: previousButtonDisabled = (() => {
		switch (selectionStage) {
			case 0:
				return false;
			case 1:
				return false;
			case 2:
				return farthestYearInPast === slicedYearList[0][0].value;
		}
	})();

	/////////////////// Methods below this ///////////////////

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

	const setCurrentDate = (value: number, isActive?: boolean) => (e: MouseEvent) => {
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
		datePickerOpen = false;
	};
</script>

{#if datePickerOpen}
	<div class="datepicker" bind:this={datePickerUI}>
		<div class="datepicker__actions">
			<button class="datepicker__actions__selector" on:click={updateSelectionStage}>
				{#if !selectionStage}
					{d.month(currentMonth).format('MMMM')}
				{/if}
				{selectionStage === 2 ? `${decadeRange.start} - ${decadeRange.end}` : currentYear}
			</button>

			<div class="datepicker__actions__buttons">
				<button on:click={previous} disabled={previousButtonDisabled}>
					<img src={upIcon} alt="<" width="24px" height="24px" />
				</button>
				<button on:click={next}>
					<img src={downIcon} alt=">" width="24px" height="24px" />
				</button>
			</div>
		</div>

		{#if !selectionStage}
			<Row
				{selected}
				{currentYear}
				{currentMonth}
				rows={[weekdays]}
				stage={selectionStage}
				onCellClick={null}
				nohover
			/>
		{/if}

		<Row
			{selected}
			{currentYear}
			{currentMonth}
			nohover={null}
			stage={selectionStage}
			rows={[slicedDateList, slicedMonthList, slicedYearList][selectionStage]}
			onCellClick={[setCurrentDate, setCurrentMonth, setCurrentYear][selectionStage]}
		/>
	</div>

	<style lang="scss">
		@use '$lib/styles/vars' as v;

		.datepicker {
			position: absolute;
			top: 100%;
			right: 0%;

			gap: 0.25rem;
			display: flex;
			flex-direction: column;

			max-width: max-content;
			max-height: max-content;

			color: v.$text-color-primary;
			padding: 0.5rem;
			border-radius: 0.5rem;
			background-color: darken(v.$background-primary, 3%);
			border: 2px solid lighten(v.$background-primary, 2.5%);
			border-top: 0;
			border-top-left-radius: 0;
			border-top-right-radius: 0;

			&__actions {
				gap: 0.5rem;
				display: flex;
				align-items: stretch;
				justify-content: space-between;

				&__selector {
					background-color: darken(v.$background-primary, 3%);
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
						background-color: lighten(v.$background-primary, 3%);
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
							background-color: lighten(v.$background-primary, 3%);
						}
					}
				}
			}
		}
	</style>
{/if}
