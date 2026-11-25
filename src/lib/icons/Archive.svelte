<script>
	/**
	 * Archive — box + lid from alpha-ui/icons/archive.svg.
	 * Default stroked for chrome / popovers; `filled` for solid mark.
	 * @type {{ size?: number, className?: string, filled?: boolean, strokeWidth?: number | string }}
	 */
	let { size = 14, className = '', filled = false, strokeWidth = 1.35 } = $props();
	const h = $derived(Math.round((size * 11) / 12));
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(h / 16)}rem">
	<svg viewBox="0 0 12 11" width={size} height={h} fill="none" aria-hidden="true">
		<path
			d="M0.75 0H11.25C11.6648 0 12 0.335156 12 0.75V1.5C12 1.91484 11.6648 2.25 11.25 2.25H0.75C0.335156 2.25 0 1.91484 0 1.5V0.75C0 0.335156 0.335156 0 0.75 0ZM0.75 3.6C0.75 3.26863 1.01863 3 1.35 3H10.65C10.9814 3 11.25 3.26863 11.25 3.6V9C11.25 9.82734 10.5773 10.5 9.75 10.5H2.25C1.42266 10.5 0.75 9.82734 0.75 9V3.6ZM3.75 4.875C3.75 5.08125 3.91875 5.25 4.125 5.25H7.875C8.08125 5.25 8.25 5.08125 8.25 4.875C8.25 4.66875 8.08125 4.5 7.875 4.5H4.125C3.91875 4.5 3.75 4.66875 3.75 4.875Z"
			fill={filled ? 'currentColor' : 'none'}
			stroke={filled ? 'none' : 'currentColor'}
			stroke-width={filled ? 0 : strokeWidth}
			stroke-linejoin="round"
		/>
	</svg>
</span>

<style>
	.icon {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		flex-shrink: 0;
		line-height: 0;
		color: inherit;
	}

	.icon :global(svg) {
		display: block;
		width: 100%;
		height: 100%;
		overflow: visible;
	}

	.icon :global(path) {
		vector-effect: non-scaling-stroke;
	}
</style>
