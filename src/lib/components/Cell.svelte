<script lang="ts">
	import type { ICellValue } from '../../types';
	import type { Dayjs } from 'dayjs';

	export let selected: Dayjs;
	export let nohover: boolean | null;
	export let data: ICellValue | string;
	export let onClick: null | ((value: number, isActive?: boolean) => (e: MouseEvent) => void);
</script>

<div
	role="none"
	class="cell"
	class:nohover
	class:active={typeof data === 'string' ? undefined : data.active}
	class:selected={typeof data !== 'string' && selected.date() === data.value}
	on:click={typeof data === 'string' ? null : onClick?.(data.value, data?.active)}
>
	{typeof data === 'string' ? data : data.text}
</div>

<style lang="scss">
	@use '$lib/styles/vars' as v;

	.cell {
		flex: 1;
		width: 2.5rem;
		height: 2.5rem;
		display: grid;
		cursor: pointer;
		place-content: center;
		transition: all 75ms ease-in-out;
		border-radius: 0.65rem;
		border: 2px solid;

		&.nohover {
			color: darken(v.$highlight-primary, 5%);
			font-weight: 600;
			cursor: default;
		}

		&:not(.nohover) {
			&:hover {
				background-color: lighten(v.$background-secondary, 5%);
				color: v.$text-color-primary !important;
			}

			&.selected {
				background-color: v.$background-secondary;
				color: v.$highlight-primary !important;
			}

			&:not(&.active) {
				color: v.$text-color-secondary;
			}
		}
	}
</style>
