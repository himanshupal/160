<script lang="ts">
	import type { ICellValue } from '../../types';
	import type { Dayjs } from 'dayjs';

	export let stage: 0 | 1 | 2;
	export let selected: Dayjs;
	export let currentYear: number;
	export let currentMonth: number;
	export let nohover: boolean | null;
	export let data: ICellValue | string;
	export let onClick: null | ((value: number, isActive: boolean) => (e: MouseEvent) => void);

	$: isActiveYear = currentYear === selected.year();
	$: isActiveMonth = currentMonth % 12 === selected.month();

	$: isSelected = (() => {
		if (typeof data === 'string') return false;
		switch (stage) {
			case 0:
				return isActiveMonth && data.active && data.value === selected.date();
			case 1:
				return isActiveYear && data.active && data.value === currentMonth;
			case 2:
				return data.active && data.value === selected.year();
		}
	})();
</script>

<div
	role="none"
	class="cell"
	class:nohover
	class:lg={!!stage}
	class:selected={isSelected}
	class:active={typeof data === 'string' ? undefined : data.active}
	on:click={typeof data === 'string' ? null : onClick?.(data.value, data.active)}
>
	{typeof data === 'string' ? data : data.text}
</div>

<style lang="scss">
	$background-primary: #353535;
	$background-secondary: #49b3eb;

	$text-color-primary: #ebebeb;
	$text-color-secondary: #686868;

	.cell {
		flex: 1;
		width: 2rem;
		height: 2rem;
		display: grid;
		cursor: pointer;
		place-content: center;
		transition: all 75ms ease-in-out;

		&.nohover {
			color: darken($text-color-primary, 25%);
			font-weight: 600;
			cursor: default;
		}

		&.lg {
			width: 3.688rem;
			height: 3.688rem;
		}

		&:not(.nohover) {
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
</style>
