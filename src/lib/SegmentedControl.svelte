<script>
	/**
	 * Recessed pill selector (mail category / settings segs).
	 * Outer track has a subtle stroke; selected chip uses glass + --glass-shine.
	 *
	 * @typedef {{ value: string, label: string, badge?: number }} SegOption
	 */
	let {
		options = /** @type {SegOption[]} */ ([]),
		value = '',
		label = '',
		equal = true,
		onchange = /** @type {(next: string) => void} */ (() => {})
	} = $props();
</script>

<div class="seg-track" class:equal role="group" aria-label={label || undefined}>
	{#each options as opt (opt.value)}
		<button
			type="button"
			class="seg-btn"
			class:on={value === opt.value}
			onclick={() => onchange(opt.value)}
		>
			<span class="seg-label">{opt.label}</span>
			{#if (opt.badge ?? 0) > 0}
				<span class="seg-badge">{opt.badge > 9 ? '9+' : opt.badge}</span>
			{/if}
		</button>
	{/each}
</div>

<style>
	.seg-track {
		display: grid;
		grid-auto-flow: column;
		align-items: stretch;
		gap: 2px;
		min-height: 40px;
		padding: 4px;
		border-radius: 999px;
		border: 1px solid color-mix(in srgb, var(--fg, var(--ink)) 10%, transparent);
		background: color-mix(in srgb, var(--fg, var(--ink)) 4%, transparent);
		overflow-x: auto;
		scrollbar-width: none;
		box-sizing: border-box;
	}

	.seg-track.equal {
		grid-auto-columns: 1fr;
	}

	.seg-track:not(.equal) {
		grid-auto-columns: max-content;
	}

	.seg-track::-webkit-scrollbar {
		display: none;
	}

	.seg-btn {
		appearance: none;
		border: 1px solid transparent;
		background: transparent;
		color: var(--fg-66, var(--ink66, color-mix(in srgb, var(--fg, var(--ink)) 66%, transparent)));
		font: inherit;
		font-size: 12px;
		font-weight: 600;
		min-height: 34px;
		height: 100%;
		padding: 0 8px;
		border-radius: 999px;
		cursor: pointer;
		white-space: nowrap;
		box-sizing: border-box;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 5px;
		transition:
			background 0.15s ease,
			color 0.15s ease;
	}

	.seg-label {
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.seg-badge {
		flex-shrink: 0;
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 11px;
		font-weight: 600;
		font-variant-numeric: lining-nums;
		font-feature-settings: 'lnum' 1, 'tnum' 0, 'onum' 0;
		color: var(--fg-33, color-mix(in srgb, var(--fg, var(--ink)) 33%, transparent));
		letter-spacing: 0;
		line-height: 1;
	}

	.seg-btn.on .seg-badge {
		color: var(--fg-66, color-mix(in srgb, var(--fg, var(--ink)) 66%, transparent));
	}

	.seg-btn:hover {
		color: var(--fg, var(--ink));
	}

	.seg-btn.on {
		color: var(--fg, var(--ink));
		background:
			linear-gradient(var(--glass, var(--popover, transparent)), var(--glass, var(--popover, transparent)))
				padding-box,
			var(--glass-shine) border-box;
		backdrop-filter: blur(var(--glass-blur, 22px));
		-webkit-backdrop-filter: blur(var(--glass-blur, 22px));
		box-shadow: none;
	}
</style>
