<script>
	/**
	 * Presentation-only detail column frame (list + detail chrome parity).
	 * Apps own body content; this owns layout, mobile slide-over, and top bars.
	 */
	import { attachChromeScrollFade } from './chrome-scroll-fade.js';
	import ChromeMobileBar from './ChromeMobileBar.svelte';
	import MetaballLoader from './MetaballLoader.svelte';

	/**
	 * @type {{
	 *   open?: boolean,
	 *   empty?: boolean,
	 *   label?: string,
	 *   title?: string,
	 *   syncing?: boolean,
	 *   onClose?: () => void,
	 *   mobileTrailing?: import('svelte').Snippet,
	 *   desktopActions?: import('svelte').Snippet,
	 *   desktopLeading?: import('svelte').Snippet,
	 *   children?: import('svelte').Snippet
	 * }}
	 */
	let {
		open = false,
		empty = false,
		label = 'Detail',
		title = '',
		syncing = false,
		onClose = () => {},
		mobileTrailing,
		desktopActions,
		desktopLeading,
		children
	} = $props();

	/** @type {HTMLElement | undefined} */
	let detailRoot = $state();
	let detailFade = $state(0);

	$effect(() => {
		open;
		empty;
		const root = detailRoot;
		return attachChromeScrollFade(root, (v) => {
			detailFade = v;
		});
	});
</script>

<aside
	class="detail"
	class:open
	class:empty
	aria-label={label}
	bind:this={detailRoot}
	style="--chrome-scroll-fade: {detailFade}"
	class:titles-syncing={syncing}
	aria-busy={syncing || undefined}
>
	{#if !empty}
		<ChromeMobileBar title={title || label} {syncing} onBack={onClose} trailing={mobileTrailing} />

		<header class="detail-top desktop-head">
			<div class="detail-top-start">
				{#if desktopLeading}
					{@render desktopLeading()}
				{:else}
					<button type="button" class="icon-hit" aria-label="Back" onclick={onClose}>
						<svg width="16" height="16" viewBox="0 0 16 16" fill="none" aria-hidden="true">
							<path
								d="M10 3.5 5.5 8 10 12.5"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								stroke-linejoin="round"
							/>
						</svg>
					</button>
				{/if}
				<span class="detail-top-label">{title || label}</span>
				{#if syncing}
					<span class="title-loader"><MetaballLoader size="sm" label="Refreshing" /></span>
				{/if}
			</div>
			<div class="detail-top-actions">
				{#if desktopActions}
					{@render desktopActions()}
				{/if}
			</div>
		</header>

		<div class="body">
			{@render children?.()}
		</div>
	{:else}
		<div class="empty">
			<p class="empty-title">Nothing selected</p>
			<p class="empty-body">Pick an item from the list.</p>
		</div>
	{/if}
</aside>

<style>
	.detail {
		position: relative;
		flex: 1;
		min-width: 0;
		min-height: 0;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		background: var(--bg);
		color: var(--fg);
	}

	.desktop-head {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		z-index: 6;
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 0.625rem;
		height: 3rem;
		padding: 0 0.75rem 0 0.625rem;
		pointer-events: none;
		box-sizing: border-box;
	}

	.desktop-head > * {
		pointer-events: auto;
	}

	.detail-top-start {
		display: flex;
		align-items: center;
		/* Match ListTitleChrome / Admin detail-top back ↔ title */
		gap: 0.75rem;
		min-width: 0;
	}

	.detail-top-label {
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.8125rem;
		font-weight: 500;
		color: var(--fg-33);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.title-loader {
		display: inline-flex;
		align-items: center;
		align-self: center;
		flex-shrink: 0;
		line-height: 0;
		--loader-ink: var(--fg-66);
		color: var(--fg-66);
		opacity: 0.85;
	}

	.detail-top-actions {
		display: flex;
		align-items: center;
		gap: 0.25rem;
	}

	.icon-hit {
		display: grid;
		place-items: center;
		width: 1.75rem;
		height: 1.75rem;
		border: 1px solid transparent;
		border-radius: 0.625rem;
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		color: var(--fg-66);
		cursor: pointer;
		padding: 0;
		box-sizing: border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
	}

	.icon-hit:hover {
		color: var(--fg);
	}

	.body {
		flex: 1;
		min-height: 0;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		padding-top: 3rem;
	}

	.empty {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 2rem 1.5rem;
		text-align: center;
	}

	.empty-title {
		margin: 0;
		font-family: var(--font);
		font-size: 1.65rem;
		font-weight: 650;
		letter-spacing: -0.02em;
		color: var(--fg-33);
	}

	.empty-body {
		margin: 0.55rem 0 0;
		max-width: 16rem;
		font-size: 0.875rem;
		line-height: 1.5;
		color: var(--fg-16);
	}

	@media (max-width: 767px) {
		.detail {
			position: fixed;
			inset: 0;
			z-index: 55;
			width: 100%;
			background: var(--shell);
			transform: translate3d(calc((1 - var(--detail-progress, 0)) * 100%), 0, 0)
				scale(calc(1.02 - 0.02 * var(--detail-progress, 0)));
			transform-origin: right center;
			transition:
				transform 0.4s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				box-shadow 0.4s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
			pointer-events: none;
			box-shadow: -12px 0 40px
				color-mix(in srgb, var(--fg-raw, #000) calc(22% * var(--detail-progress, 0)), transparent);
			will-change: transform;
		}

		.detail.open {
			pointer-events: auto;
		}

		.detail.dragging {
			transition: none;
		}

		.detail.empty {
			display: flex;
		}

		.desktop-head {
			display: none;
		}

		.body {
			padding-top: var(--detail-scroll-pad-top-mobile, calc(3.5rem + env(safe-area-inset-top, 0px)));
			padding-left: 0.875rem;
			padding-right: 0.875rem;
			padding-bottom: calc(1.5rem + env(safe-area-inset-bottom, 0px));
		}
	}
</style>
