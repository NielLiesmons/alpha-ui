<script>
	/**
	 * Product loader — metaball (legacy Spinner API).
	 * Prefer importing MetaballLoader; size numbers map to sm (≤14) / default.
	 * @type {{ size?: number | 'sm' | 'default', label?: string }}
	 */
	import MetaballLoader from './MetaballLoader.svelte';

	let { size = 22, label = 'Loading' } = $props();

	/** @param {number | 'sm' | 'default'} s */
	function variant(s) {
		if (s === 'sm' || s === 'default') return s;
		return typeof s === 'number' && s <= 14 ? 'sm' : 'default';
	}
</script>

<MetaballLoader size={variant(size)} {label} />
