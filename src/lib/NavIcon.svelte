<script>
	/**
	 * Your glyphs keep native viewBoxes. Stroke is --icon-stroke in screen px
	 * (vector-effect: non-scaling-stroke) so Drive/Mail/Chat match visually.
	 * Tune --icon-stroke in @alpha/ui tokens.
	 */
	import BaseIcon from './icons/BaseIcon.svelte';
	import { ENTERPRISE_STAR_18 } from './star-path.js';

	/** @type {{ name?: string, size?: number }} */
	let { name = 'dot', size = 22 } = $props();
</script>

<BaseIcon {size}>
	{#if name === 'songs' || name === 'music'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<path
				d="M6 12.5V4.2l7-1.4v8.2"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
			<circle cx="4.5" cy="12.5" r="1.5" stroke="currentColor" />
			<circle cx="11.5" cy="11" r="1.5" stroke="currentColor" />
		</svg>
	{:else if name === 'albums' || name === 'album'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<circle cx="8" cy="8" r="5.5" stroke="currentColor" />
			<circle cx="8" cy="8" r="1.6" stroke="currentColor" />
		</svg>
	{:else if name === 'playlist' || name === 'playlists'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<path d="M3 4.5h10M3 8h10M3 11.5h6" stroke="currentColor" stroke-linecap="round" />
		</svg>
	{:else if name === 'home'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<path
				d="M2.5 7.2 8 2.5l5.5 4.7V13a.5.5 0 0 1-.5.5H9.2V9.2H6.8V13.5H3a.5.5 0 0 1-.5-.5V7.2Z"
				stroke="currentColor"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'artists' || name === 'artist'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<circle cx="8" cy="5" r="2.25" stroke="currentColor" />
			<path
				d="M3.5 13c.6-2.2 2.2-3.5 4.5-3.5s3.9 1.3 4.5 3.5"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else if name === 'recent' || name === 'played' || name === 'clock'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<circle cx="8" cy="8" r="5.5" stroke="currentColor" />
			<path
				d="M8 5v3.2l2 1.3"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'upload'}
		<svg viewBox="0 0 16 16" fill="none" aria-hidden="true">
			<path
				d="M8 11.5V3.5M4.5 6.5 8 3l3.5 3.5M3.5 12.5h9"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'drive'}
		<svg viewBox="0 0 41 33" fill="none" aria-hidden="true">
			<path
				d="M0.5 24.5L6.2056 4.75303C6.9745 2.22664 9.30483 0.5 11.9456 0.5H29.0544C31.6952 0.5 34.0255 2.22664 34.7944 4.75303L40.5 24.5M6.5 25.5H10.5M7.5 32.5H33.5C37.366 32.5 40.5 29.366 40.5 25.5C40.5 21.634 37.366 18.5 33.5 18.5H7.5C3.63401 18.5 0.5 21.634 0.5 25.5C0.5 29.366 3.63401 32.5 7.5 32.5Z"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else if name === 'mail'}
		<svg viewBox="0 0 26 22" fill="none" aria-hidden="true">
			<path
				d="M24.7002 5.75293L14.7451 11.5605C13.5718 12.2449 12.1385 12.2872 10.9326 11.6885L10.6943 11.5605L0.739258 5.75293V16.7188C0.739258 18.9168 2.52163 20.6992 4.71973 20.6992H20.7197C22.9178 20.6992 24.7002 18.9168 24.7002 16.7188V5.75293ZM24.7002 4.71875C24.7002 2.52066 22.9178 0.738281 20.7197 0.738281H4.71973C2.52163 0.738281 0.739258 2.52066 0.739258 4.71875V5.70605L10.7139 11.5254L10.9502 11.6533C12.1441 12.2461 13.5639 12.203 14.7256 11.5254L24.7002 5.70605V4.71875ZM24.7393 16.7188C24.7393 18.9389 22.9399 20.7383 20.7197 20.7383H4.71973C2.49954 20.7383 0.700195 18.9389 0.700195 16.7188V4.71875C0.700195 2.49857 2.49954 0.699219 4.71973 0.699219H20.7197C22.9399 0.699219 24.7393 2.49857 24.7393 4.71875V16.7188Z"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'chat'}
		<svg viewBox="0 0 18 19" fill="none" aria-hidden="true">
			<path
				d="M9 0.5C13.6944 0.5 17.5 4.30558 17.5 9C17.5 13.6944 13.6944 17.5 9 17.5C7.77335 17.5 6.60815 17.2386 5.55507 16.7706C5.3323 16.6716 5.08291 16.6441 4.84737 16.7069L1.79379 17.5216C1.02463 17.7269 0.335703 16.9948 0.587211 16.2395L1.4629 13.6098C1.55241 13.341 1.52084 13.0482 1.39409 12.7948C0.822691 11.6525 0.5 10.3641 0.5 9C0.5 4.30563 4.30565 0.500084 9 0.5Z"
				stroke="currentColor"
			/>
		</svg>
	{:else if name === 'tasks'}
		<!-- Checkmark (Bley Check glyph, stroke for nav weight) -->
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<path
				d="M3.2 9.4 L7.1 13.1 L14.8 4.4"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'projects'}
		<svg viewBox="0 0 24 24" fill="none" aria-hidden="true">
			<path
				d="M13.299 1.75L20.227 5.75Q21.526 6.5 21.526 8L21.526 16Q21.526 17.5 20.227 18.25L13.299 22.25Q12 23 10.701 22.25L3.773 18.25Q2.474 17.5 2.474 16L2.474 8Q2.474 6.5 3.773 5.75L10.701 1.75Q12 1 13.299 1.75Z"
				stroke="currentColor"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'notes' || name === 'note'}
		<!-- alpha-ui/icons/note.svg verbatim -->
		<svg viewBox="0 0 17 17" fill="none" aria-hidden="true">
			<path
				d="M4.5 0.5H12.5C14.7091 0.5 16.5 2.29086 16.5 4.5V9.5C16.5 10.7318 14.9075 12.0746 13.5 13.5C12.051 14.9674 10.7682 16.5 9.5 16.5H4.5C2.29086 16.5 0.5 14.7091 0.5 12.5V4.5C0.5 2.29086 2.29086 0.5 4.5 0.5Z"
				stroke="currentColor"
			/>
			<path
				d="M16.5 8.5C16.5 11 12.5 8.5 10.5 10.5C8.5 12.5 11 16.5 8.5 16.5"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else if name === 'calendar' || name === 'planning'}
		<!-- alpha-ui/icons/planning.svg — optically slightly smaller than peer glyphs -->
		<svg viewBox="0 0 10 10" fill="none" aria-hidden="true" class="planning-glyph">
			<path d="M1 1H7M4 5H9M2 9H6" stroke="currentColor" stroke-linecap="round" />
		</svg>
	{:else if name === 'dashboard'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<rect x="1.5" y="1.5" width="15" height="15" rx="1" stroke="currentColor" />
			<path d="M9 1.5v15M1.5 9h15" stroke="currentColor" stroke-linecap="round" />
		</svg>
	{:else if name === 'player'}
		<!-- Bley play glyph — stroke; optical size matched to peer icons -->
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<g transform="translate(2.1, 1.85) scale(0.92)">
				<path
					d="M13.439 5.88263C13.8097 6.08696 14.1199 6.39198 14.3361 6.76502C14.5524 7.13805 14.6667 7.56504 14.6667 8.00022C14.6667 8.43541 14.5524 8.8624 14.3361 9.23543C14.1199 9.60847 13.8097 9.91349 13.439 10.1178L3.54868 15.6914C1.95613 16.589 0 15.421 0 13.5746V2.42664C0 0.578645 1.95613 -0.588552 3.54868 0.308245L13.439 5.88263Z"
					stroke="currentColor"
					stroke-linejoin="round"
				/>
			</g>
		</svg>
	{:else if name === 'website' || name === 'link' || name === 'globe'}
		<!-- alpha-ui/icons/website.svg verbatim -->
		<svg viewBox="0 0 14 14" fill="none" aria-hidden="true">
			<path
				d="M7 13.5C10.5899 13.5 13.5 10.5899 13.5 7C13.5 3.41015 10.5899 0.5 7 0.5C3.41015 0.5 0.5 3.41015 0.5 7C0.5 10.5899 3.41015 13.5 7 13.5Z"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
			<path
				d="M0.5 7H13.5"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
			<path
				d="M9.5 7C9.3772 9.37699 8.50168 11.6533 7 13.5C5.49832 11.6533 4.6228 9.37699 4.5 7C4.6228 4.62301 5.49832 2.34665 7 0.5C8.50168 2.34665 9.3772 4.62301 9.5 7Z"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'publications'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<path
				d="M3.5 2.5h8L15.5 6v9a1.5 1.5 0 0 1-1.5 1.5h-10A1.5 1.5 0 0 1 2.5 15V4A1.5 1.5 0 0 1 4 2.5h-.5Z"
				stroke="currentColor"
				stroke-linejoin="round"
			/>
			<path
				d="M11.5 2.5V5.5H14.5M5.5 9h7M5.5 11.5h5"
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'profile' || name === 'accounts' || name === 'people' || name === 'members'}
		<!-- alpha-ui / Profile silhouette (stroke for nav) -->
		<svg viewBox="0 0 16 20" fill="none" aria-hidden="true">
			<path
				d="M12.8 4.70588C12.8 7.30487 10.651 9.41177 8 9.41177C5.34903 9.41177 3.2 7.30487 3.2 4.70588C3.2 2.1069 5.34903 0 8 0C10.651 0 12.8 2.1069 12.8 4.70588Z"
				stroke="currentColor"
				stroke-width="1.35"
				stroke-linejoin="round"
			/>
			<path
				d="M16 16.2353C16 18.3145 12.4183 20 8 20C3.58172 20 0 18.3145 0 16.2353C0 14.1561 3.58172 12.4706 8 12.4706C12.4183 12.4706 16 14.1561 16 16.2353Z"
				stroke="currentColor"
				stroke-width="1.35"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'settings'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<circle cx="9" cy="9" r="2.25" stroke="currentColor" />
			<path
				d="M9 1.75v1.5M9 14.75v1.5M1.75 9h1.5M14.75 9h1.5M3.4 3.4l1.06 1.06M13.54 13.54l1.06 1.06M14.6 3.4l-1.06 1.06M4.46 13.54l-1.06 1.06"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else if name === 'accounting'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<rect x="3.5" y="1.5" width="11" height="15" rx="1.5" stroke="currentColor" />
			<path
				d="M6.5 5.5h5M6.5 8.5h5M6.5 11.5h3"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else if name === 'enterprise' || name === 'entreprises'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<path
				d={ENTERPRISE_STAR_18}
				stroke="currentColor"
				stroke-linecap="round"
				stroke-linejoin="round"
			/>
		</svg>
	{:else if name === 'shop'}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<path
				d="M3.5 5.5h11l-1 9.5H4.5l-1-9.5Z"
				stroke="currentColor"
				stroke-linejoin="round"
			/>
			<path
				d="M6.5 5.5V4a2.5 2.5 0 0 1 5 0v1.5"
				stroke="currentColor"
				stroke-linecap="round"
			/>
		</svg>
	{:else}
		<svg viewBox="0 0 18 18" fill="none" aria-hidden="true">
			<circle cx="9" cy="9" r="2" stroke="currentColor" />
		</svg>
	{/if}
</BaseIcon>

<style>
	/* Planning bars read large in a square slot — shrink optically vs peer icons */
	:global(.planning-glyph) {
		width: 78% !important;
		height: 78% !important;
	}
</style>
