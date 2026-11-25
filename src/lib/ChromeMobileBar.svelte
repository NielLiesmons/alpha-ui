<script>
	/**
	 * Shared mobile detail top bar: round glass back, centered muted title, trailing slot.
	 * Matches Admin DetailColumn mobile chrome — use from DetailChrome / nested list titles.
	 * @type {{
	 *   title?: string,
	 *   syncing?: boolean,
	 *   onBack?: () => void,
	 *   trailing?: import('svelte').Snippet
	 * }}
	 */
	import MetaballLoader from './MetaballLoader.svelte';

	let { title = '', syncing = false, onBack = () => {}, trailing } = $props();
</script>

<header class="mobile-bar">
	<button type="button" class="chrome-btn chev-back" aria-label="Back" onclick={onBack}>
		<svg width="22" height="22" viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<path
				d="M11.5 3.5 6 9l5.5 5.5"
				stroke="currentColor"
				stroke-width="1.7"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	</button>
	<span class="mobile-title-stack" class:titles-syncing={syncing} aria-busy={syncing || undefined}>
		<span class="mobile-type">{title}</span>
		{#if syncing}
			<span class="title-loader"><MetaballLoader size="sm" label="Refreshing" /></span>
		{/if}
	</span>
	<span class="trail-slot">
		{#if trailing}
			{@render trailing()}
		{:else}
			<span class="menu-spacer" aria-hidden="true"></span>
		{/if}
	</span>
</header>

<style>
	.mobile-bar {
		display: none;
	}

	.menu-spacer {
		width: var(--chrome-mobile, 2.75rem);
		height: var(--chrome-mobile, 2.75rem);
		flex-shrink: 0;
	}

	.trail-slot {
		display: contents;
	}

	@media (max-width: 767px) {
		.mobile-bar {
			position: absolute;
			top: 0;
			left: 0;
			right: 0;
			z-index: 8;
			display: flex;
			align-items: center;
			justify-content: space-between;
			gap: 0.75rem;
			height: calc(5.5rem + env(safe-area-inset-top, 0px));
			padding: calc(0.375rem + env(safe-area-inset-top, 0px)) 0.875rem 2.125rem;
			border-bottom: none;
			flex-shrink: 0;
			box-sizing: border-box;
			pointer-events: none;
			background: transparent;
		}

		.mobile-bar::after {
			content: '';
			position: absolute;
			top: 0;
			left: 0;
			right: 0;
			height: 100%;
			background: linear-gradient(
				to bottom,
				var(--shell, var(--bg)) 0%,
				color-mix(in srgb, var(--shell, var(--bg)) 80%, transparent) 50%,
				transparent 100%
			);
			opacity: var(--chrome-scroll-fade, 0);
			pointer-events: none;
			z-index: -1;
		}

		.mobile-bar > :global(*),
		.trail-slot > :global(*) {
			pointer-events: auto;
		}

		.mobile-title-stack {
			position: relative;
			flex: 1;
			min-width: 0;
			height: var(--chrome-mobile, 2.75rem);
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			gap: 0.375rem;
		}

		.mobile-type {
			min-width: 0;
			max-width: 100%;
			text-align: center;
			font-family: 'Inter', var(--font), system-ui, sans-serif;
			font-size: 1.0625rem;
			font-weight: 600;
			letter-spacing: -0.01em;
			line-height: 1.15;
			color: var(--fg-66);
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
		}

		.title-loader {
			display: inline-flex;
			align-items: center;
			justify-content: center;
			align-self: center;
			flex: 0 0 auto;
			margin-left: 2px;
			line-height: 0;
			overflow: visible;
			padding-top: 2px;
			box-sizing: content-box;
			--loader-ink: var(--fg-66);
			color: var(--fg-66);
			opacity: 0.85;
		}

		.trail-slot {
			display: flex;
			align-items: center;
			justify-content: flex-end;
			gap: 0.375rem;
			width: auto;
			min-width: var(--chrome-mobile, 2.75rem);
			max-width: min(58vw, 13.75rem);
			flex-shrink: 0;
		}

		.chrome-btn,
		.trail-slot > :global(.chrome-btn) {
			width: var(--chrome-mobile, 2.75rem);
			height: var(--chrome-mobile, 2.75rem);
			display: grid;
			place-items: center;
			border: 1px solid transparent;
			border-radius: 9999px;
			background:
				linear-gradient(var(--glass), var(--glass)) padding-box,
				var(--glass-shine) border-box;
			color: var(--fg-66);
			padding: 0;
			cursor: pointer;
			flex-shrink: 0;
			box-sizing: border-box;
			backdrop-filter: blur(var(--glass-blur));
			-webkit-backdrop-filter: blur(var(--glass-blur));
			-webkit-tap-highlight-color: transparent;
			transition: transform 0.15s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.chrome-btn:hover,
		.chrome-btn:global([aria-expanded='true']),
		.trail-slot > :global(.chrome-btn:hover),
		.trail-slot > :global(.chrome-btn[aria-expanded='true']) {
			transform: scale(1.04);
		}

		.chrome-btn :global(svg),
		.trail-slot > :global(.chrome-btn svg) {
			width: var(--chrome-icon, 1.375rem);
			height: var(--chrome-icon, 1.375rem);
		}

		.trail-slot > :global(.chrome-btn.more-btn svg) {
			width: 1.625rem;
			height: 1.625rem;
		}

		.chrome-btn.chev-back :global(svg) {
			padding-right: 0.125rem;
			box-sizing: content-box;
		}
	}
</style>
