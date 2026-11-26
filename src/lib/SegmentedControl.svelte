<script>
	/**
	 * Recessed pill selector (mail category / settings segs).
	 * Outer track has a subtle stroke; selected chip uses glass + --glass-shine.
	 *
	 * @typedef {{
	 *   value: string,
	 *   label: string,
	 *   badge?: number,
	 *   emoji?: string,
	 *   icon?: import('svelte').Component<any>,
	 *   labelFontSize?: string
	 * }} SegOption
	 */
	let {
		options = /** @type {SegOption[]} */ ([]),
		value = '',
		label = '',
		equal = true,
		onchange = /** @type {(next: string) => void} */ (() => {})
	} = $props();
</script>

<div class="seg-track" class:equal role="group" aria-label={label || undefined} data-h-scroll>
	{#each options as opt (opt.value)}
		<button
			type="button"
			class="seg-btn"
			class:on={value === opt.value}
			class:has-leading={Boolean(opt.icon || opt.emoji)}
			onclick={() => onchange(opt.value)}
		>
			{#if opt.icon}
				{@const Icon = opt.icon}
				<span class="seg-ico" aria-hidden="true"><Icon size={15} /></span>
			{:else if opt.emoji}
				<span class="seg-emoji" aria-hidden="true">{opt.emoji}</span>
			{/if}
			<span
				class="seg-label"
				style:font-size={opt.labelFontSize || undefined}
			>{opt.label}</span>
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
		gap: 0.125rem;
		min-height: 2.5rem;
		padding: 0.25rem;
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
		font-size: 0.75rem;
		font-weight: 600;
		min-height: 2.125rem;
		height: 100%;
		padding: 0 0.5rem;
		border-radius: 999px;
		cursor: pointer;
		white-space: nowrap;
		box-sizing: border-box;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 0.3125rem;
		transition:
			background 0.15s ease,
			color 0.15s ease;
	}

	/* Language / themed chips: bigger leading mark + more air before the label */
	.seg-btn.has-leading {
		gap: 0.5625rem;
	}

	.seg-ico {
		flex-shrink: 0;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		line-height: 0;
		color: inherit;
	}

	.seg-emoji {
		flex-shrink: 0;
		font-size: 1.125rem;
		line-height: 1;
	}

	.seg-label {
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		line-height: 1.2;
	}

	.seg-badge {
		flex-shrink: 0;
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.6875rem;
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
