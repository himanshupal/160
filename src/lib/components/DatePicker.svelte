<script lang="ts">
	import { dayOffset, days, months, yearsRange } from '$lib/config';
	import type { ICellValue } from '../../types';
	import Row from './Row.svelte';
	import dayjs from 'dayjs';

	/////////////////// Props below this ///////////////////

	export let date: string;

	/////////////////// Vars declared below this ///////////////////

	let selected = dayjs();
	let selectedMonth = selected.month();
	let selectedYear = selected.year();

	const weekdays = dayOffset ? [...days.slice(dayOffset), ...days.slice(0, dayOffset)] : days;

	const monthsList: ICellValue[] = months.map((value, i) => ({
		text: value,
		value: i
	}));

	const yearsList: ICellValue[] = Array.from({ length: yearsRange * 2 }, (_, i) => {
		const value = selected.year() - yearsRange + i;
		return { value, text: value };
	});

	/////////////////// Reactive values below this ///////////////////

	$: date = selected.format('YYYY-MM-DD');

	$: slicedDateList = ((): ICellValue[][] => {
		const daysInMonth = selected.month(selectedMonth).daysInMonth();
		const startAt = selected.month(selectedMonth).set('date', 1).day();
		const daysInPreviousMonth = selected.month(selectedMonth - 1).daysInMonth();
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
					text: daysInPreviousMonth - i
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
				text: i + 1
			}))
		];
		return Array.from({ length: 6 }, (_, i) => dateList.slice(i * 7, i * 7 + 7));
	})();

	/////////////////// Methods below this ///////////////////

	const setCurrentMonth = (value: number) => () => {
		selected = selected.month(value);
	};

	const setCurrentYear = (value: number) => () => {
		selected = selected.year(value);
	};

	const setCurrentDate = (value: number, isActive?: boolean) => (e: MouseEvent) => {
		if (isActive) {
			// Date selected is from current month
			selected = selected.date(value);
		} else if (value > 0) {
			// Date selected is from next month
			const daysInMonth = selected.month(selectedMonth).daysInMonth();
			selected = selected.month(selectedMonth + 1).date(value - daysInMonth);
			if (selectedMonth === 11) {
				selectedYear += 1;
				selectedMonth = 0;
			} else {
				selectedMonth += 1;
			}
		} else {
			// Date selected is from previous month
			const daysInMonth = selected.month(selectedMonth - 1).daysInMonth();
			selected = selected.month(selectedMonth - 1).date(daysInMonth + value);
			if (selectedMonth === 0) {
				selectedYear -= 1;
				selectedMonth = 11;
			} else {
				selectedMonth -= 1;
			}
		}
	};
</script>

<div class="datepicker">
	<div class="selection">
		<select
			class="selection__select"
			bind:value={selectedMonth}
			on:change={setCurrentMonth(selectedMonth)}
		>
			{#each monthsList as month}
				<option class="selection__select__option" value={month.value}>
					{month.text}
				</option>
			{/each}
		</select>

		<select
			class="selection__select"
			bind:value={selectedYear}
			on:change={setCurrentYear(selectedYear)}
		>
			{#each yearsList as year}
				<option class="selection__select__option" value={year.value}>
					{year.text}
				</option>
			{/each}
		</select>
	</div>

	<Row {selected} rows={[weekdays]} onCellClick={null} nohover />
	<Row {selected} rows={slicedDateList} onCellClick={setCurrentDate} nohover={null} />
</div>

<style lang="scss">
	@use '$lib/styles/vars' as v;

	.selection {
		gap: 1rem;
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-between;

		&__select {
			background-color: v.$background-primary;
			border: 2px solid v.$background-secondary;
			color: v.$highlight-primary;
			border-radius: 0.65rem;
			font-size: 1.15rem;
			text-align: center;
			min-width: 150px;
			font-weight: 600;
			padding: 0.5rem;
			outline: none;

			&__option {
				background-color: v.$background-primary;
				text-align: left;
				padding: 0.25rem;
			}
		}
	}

	.datepicker {
		gap: 1rem;
		display: flex;
		flex-direction: column;
		max-width: max-content;
		max-height: max-content;
		color: v.$text-color-primary;
	}
</style>
