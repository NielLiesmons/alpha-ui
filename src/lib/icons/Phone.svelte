<script>
	/**
	 * Phone / audio call — fill path from alpha-ui/icons/phone.svg (currentColor).
	 * @type {{ size?: number, className?: string }}
	 */
	let { size = 16, className = '' } = $props();
</script>

<span class="icon {className}" style:width="{(size / 16)}rem" style:height="{(size / 16)}rem">
	<svg viewBox="0 0 12 16" width={size} height={size} fill="none" aria-hidden="true">
		<path
			d="M1.88953 0.351052L2.75114 0.0758487C3.55755 -0.181754 4.41996 0.235051 4.76556 1.04946L5.45357 2.67188C5.75357 3.37829 5.58717 4.2095 5.04237 4.7263L3.88679 5.82389C3.66065 6.03868 3.54235 6.34779 3.60234 6.65385C3.73818 7.34694 4.00246 8.03111 4.39516 8.70635C4.76835 9.35904 5.24423 9.94592 5.8035 10.4449C6.05083 10.6656 6.40025 10.7195 6.71466 10.6145L8.0328 10.1744C8.72241 9.94477 9.47362 10.2096 9.89682 10.8312L10.8824 12.2792C11.3752 13.0024 11.2864 13.9992 10.676 14.612L10.0216 15.2688C9.37042 15.9224 8.44081 16.16 7.58 15.8912C5.54904 15.2576 3.68155 13.3768 1.97753 10.2488C0.271377 7.1154 -0.330763 4.45723 0.171109 2.27427C0.382312 1.35586 1.03672 0.623855 1.89113 0.351052"
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
