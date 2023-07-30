<script lang="ts">
	import type { ICellValue } from '../../types';
	import type { Dayjs } from 'dayjs';
	import Cell from './Cell.svelte';

	export let selected: Dayjs;
	export let stage: 0 | 1 | 2;
	export let currentYear: number;
	export let currentMonth: number;
	export let nohover: boolean | null;
	export let rows: (ICellValue[] | string[])[];
	export let onCellClick: null | ((value: number, isActive: boolean) => (e: MouseEvent) => void);
</script>

{#each rows as row}
	<div class="row">
		{#each row as data}
			<Cell
				{data}
				{stage}
				{nohover}
				{currentMonth}
				{currentYear}
				{selected}
				onClick={onCellClick}
			/>
		{/each}
	</div>
{/each}

<style lang="scss">
	$text-color-primary: #ebebeb;

	.row {
		width: 100%;
		display: flex;
		gap: 0.25rem;
	}
</style>
