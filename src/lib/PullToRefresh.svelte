<script>
	/**
	 * Mobile pull-to-refresh over a nested scroller (.feed / .fill / .body).
	 * Visual = empty space only while pulling / refreshing (no spinner/label in the gap).
	 * Title-chrome spinner is owned by ListTitleChrome / section sync.
	 * @type {{
	 *   onRefresh: () => Promise<void> | void,
	 *   disabled?: boolean,
	 *   children: import('svelte').Snippet
	 * }}
	 */
	let { onRefresh, disabled = false, children } = $props();

	/** @type {HTMLDivElement | undefined} */
	let root = $state();
	let pulling = $state(false);
	let refreshing = $state(false);
	let pullPx = $state(0);

	let startY = 0;
	let tracking = false;

	const THRESHOLD = 64;

	/** @param {HTMLElement | null | undefined} el */
	function findScroller(el) {
		if (!el) return null;
		const candidates = el.querySelectorAll('.feed, .fill, .body, .ptr-scroll');
		for (const node of candidates) {
			const style = getComputedStyle(node);
			const oy = style.overflowY;
			if ((oy === 'auto' || oy === 'scroll' || oy === 'overlay') && node.scrollHeight > node.clientHeight + 1) {
				return /** @type {HTMLElement} */ (node);
			}
		}
		for (const node of candidates) {
			return /** @type {HTMLElement} */ (node);
		}
		return el;
	}

	function scrollTopNow() {
		const scroller = findScroller(root);
		return scroller?.scrollTop ?? 0;
	}

	/** @param {TouchEvent} e */
	function onTouchStart(e) {
		if (disabled || refreshing) return;
		if (scrollTopNow() > 1) return;
		tracking = true;
		startY = e.touches[0]?.clientY ?? 0;
		pulling = false;
		pullPx = 0;
	}

	/** @param {TouchEvent} e */
	function onTouchMove(e) {
		if (!tracking || disabled || refreshing) return;
		if (scrollTopNow() > 1) {
			tracking = false;
			pullPx = 0;
			pulling = false;
			return;
		}
		const y = e.touches[0]?.clientY ?? 0;
		const dy = y - startY;
		if (dy <= 0) {
			pullPx = 0;
			pulling = false;
			return;
		}
		pulling = true;
		pullPx = Math.min(96, dy * 0.45);
		if (pullPx > 8) e.preventDefault();
	}

	async function onTouchEnd() {
		if (!tracking) return;
		tracking = false;
		const should = pullPx >= THRESHOLD && !disabled;
		pulling = false;
		pullPx = 0;
		if (!should) return;
		refreshing = true;
		try {
			await onRefresh();
		} finally {
			refreshing = false;
		}
	}
</script>

<div
	class="ptr"
	bind:this={root}
	ontouchstart={onTouchStart}
	ontouchmove={onTouchMove}
	ontouchend={onTouchEnd}
	ontouchcancel={onTouchEnd}
>
	<div
		class="ptr-indicator"
		class:on={pulling || refreshing}
		style:height="{refreshing ? 36 : pullPx}px"
		aria-hidden="true"
	></div>
	{@render children()}
</div>

<style>
	.ptr {
		position: relative;
		display: flex;
		flex-direction: column;
		flex: 1;
		min-height: 0;
		overflow: hidden;
	}

	.ptr-indicator {
		flex-shrink: 0;
		overflow: hidden;
		transition: height 0.15s ease;
		pointer-events: none;
	}

	@media (min-width: 768px) {
		.ptr-indicator {
			display: none;
		}
	}
</style>
