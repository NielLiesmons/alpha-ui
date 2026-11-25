<script>
	/**
	 * Storage — fill paths from alpha-ui/icons/storage.svg (currentColor).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 18, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 32 25" width={size} height={size} fill="none" aria-hidden="true">
		<path
			d="M26.4711 0.897091C27.2726 1.47891 27.8484 2.2946 28.1088 3.21742L30.984 13.4031C31.0971 13.8036 30.5754 14.1405 30.1911 13.9404C29.0121 13.3267 27.681 13.0006 26.3199 13.0026H5.67821C4.31756 13.0008 2.98695 13.327 1.80826 13.9406C1.42398 14.1406 0.90238 13.8038 1.0154 13.4033L3.88927 3.21742C4.14978 2.29436 4.72575 1.4785 5.52762 0.896651C6.3295 0.314806 7.31237 -0.000412217 8.32341 4.0458e-07H23.6762C24.687 -9.37779e-05 25.6695 0.315271 26.4711 0.897091Z"
			fill="currentColor"
		/>
		<path
			fill-rule="evenodd"
			clip-rule="evenodd"
			d="M26.7822 14.5625C29.6638 14.5625 32 16.8987 32 19.7803C32 22.6618 29.6638 24.998 26.7822 24.998H5.2168C2.33547 24.9978 0 22.6617 0 19.7803C1.42149e-05 16.8987 2.33623 14.5625 5.21777 14.5625H26.7822ZM5.2168 18.3867C4.44858 18.3869 3.82617 19.01 3.82617 19.7783C3.82629 20.5465 4.44865 21.1697 5.2168 21.1699C5.98513 21.1699 6.60828 20.5466 6.6084 19.7783C6.6084 19.0099 5.9852 18.3867 5.2168 18.3867Z"
			fill="currentColor"
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
</style>
