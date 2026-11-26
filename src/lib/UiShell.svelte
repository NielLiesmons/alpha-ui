<script>
	/**
	 * Configurable Alpha app shell — sidebar + list/detail + mobile drawer nav.
	 * Apps pass sections and own domain logic; this owns chrome presentation.
	 */
	import { onDestroy } from 'svelte';
	import { attachChromeScrollFade } from './chrome-scroll-fade.js';
	import { navPngUrl } from './nav-png.js';
	import MetaballLoader from './MetaballLoader.svelte';
	import Modal from './Modal.svelte';
	import NavIcon from './NavIcon.svelte';
	import SheetTopbar from './SheetTopbar.svelte';
	import UnreadMark from './UnreadMark.svelte';
	import Search from './icons/Search.svelte';
	import ChevronRight from './icons/ChevronRight.svelte';
	import PullToRefresh from './PullToRefresh.svelte';

	/**
	 * @typedef {{
	 *   id: string,
	 *   label: string,
	 *   active?: boolean
	 * }} ShellSectionChild
	 */

	/**
	 * @typedef {{
	 *   href: string,
	 *   label: string,
	 *   icon?: string,
	 *   faviconUrl?: string,
	 *   spaceBrand?: boolean,
	 *   id?: string,
	 *   countKey?: string,
	 *   createKind?: string,
	 *   active?: boolean,
	 *   children?: ShellSectionChild[]
	 * }} ShellSection
	 */

	/**
	 * @typedef {{
	 *   type: 'group',
	 *   id: string,
	 *   label: string,
	 *   creatable?: boolean,
	 *   createKind?: string,
	 *   items: ShellSection[]
	 * }} ShellNavGroup
	 */

	/**
	 * @type {{
	 *   sections?: (ShellSection | ShellNavGroup)[],
	 *   mobileSections?: (ShellSection | ShellNavGroup)[] | null,
	 *   activeHref?: string,
	 *   title?: string,
	 *   titleIcon?: string | false,
	 *   counts?: Record<string, number>,
	 *   unreadCounts?: Record<string, number>,
	 *   wideList?: boolean,
	 *   listSize?: 'default' | 'chat',
	 *   listCompact?: boolean,
	 *   commsFeed?: boolean,
	 *   syncing?: boolean,
	 *   searchEnabled?: boolean,
	 *   feedPadClass?: string,
	 *   filterOptions?: { id: string, label: string, icon?: import('svelte').Component<any> }[] | null,
	 *   filterValue?: string,
	 *   createMenuItems?: { kind: string, label: string, icon?: string }[],
	 *   createKind?: string | undefined,
	 *   onNavigate?: (href: string) => void,
	 *   onNavChild?: (parentHref: string, childId: string) => void,
	 *   onCreate?: (kind: string) => void,
	 *   onNavPlus?: (kind: string | undefined) => void,
	 *   onGroupPlus?: ((groupId: string) => void) | null,
	 *   onFilterChange?: (id: string) => void,
	 *   onRefresh?: (() => Promise<void> | void) | null,
	 *   isSectionActive?: ((href: string) => boolean) | null,
	 *   accountLabel?: string,
	 *   sidebarLead?: import('svelte').Snippet,
	 *   sidebarTrail?: import('svelte').Snippet,
	 *   defaultListActions?: boolean,
	 *   listTitleSlot?: import('svelte').Snippet,
	 *   listActionsSlot?: import('svelte').Snippet,
	 *   detail?: import('svelte').Snippet,
	 *   sheets?: import('svelte').Snippet,
	 *   banners?: import('svelte').Snippet,
	 *   children?: import('svelte').Snippet,
	 *   listBlank?: boolean,
	 *   listTopHidden?: boolean
	 * }}
	 */
	let {
		sections = [],
		/** @deprecated Bottom pill removed; kept for call-site compat. */
		mobileSections: _mobileSections = null,
		activeHref = '',
		title = '',
		/** Mobile title-pill icon: false = text-only; string = force nav key; omit = matched PNG/favicon. @type {string | false | undefined} */
		titleIcon = undefined,
		counts = {},
		/** Targeted-at-you unread (accent UnreadMark). `counts` = muted “new” totals. */
		unreadCounts = {},
		wideList = false,
		/** Wider list column (Chat feed on desktop). */
		listSize = 'default',
		/** Chat: shrink feed to φ `--list` when a deeper detail page is open. */
		listCompact = false,
		/** Chat/mail: slightly recessed feed canvas in light mode. */
		commsFeed = false,
		syncing = false,
		searchEnabled = true,
		feedPadClass = '',
		filterOptions = null,
		filterValue = 'all',
		createMenuItems = [],
		createKind = undefined,
		/** When false, only listActionsSlot is shown (Admin mail/planning chrome). */
		defaultListActions = true,
		/**
		 * Mobile section switch: blank list pane so the previous page never flashes.
		 * AppShell sets true before goto and clears afterNavigate.
		 */
		listBlank = false,
		/** Hide floating list-top (e.g. full-page call setup owns SheetTopbar). */
		listTopHidden = false,
		onNavigate = () => {},
		onNavChild = () => {},
		onCreate = () => {},
		onNavPlus = () => {},
		onGroupPlus = null,
		onFilterChange = () => {},
		onRefresh = null,
		isSectionActive = null,
		/** Display name next to the profile pic in the menu header. */
		accountLabel = '',
		sidebarLead,
		sidebarTrail,
		listTitleSlot,
		listActionsSlot,
		detail,
		sheets,
		banners,
		children
	} = $props();

	/** @type {HTMLElement | undefined} */
	let listColEl = $state();
	/** @type {HTMLElement | undefined} */
	let shellEl = $state();
	/** @type {HTMLElement | undefined} */
	let workspaceWrapEl = $state();
	let listFade = $state(0);
	let search = $state('');
	let searchOpen = $state(false);
	let createOpen = $state(false);
	let filterOpen = $state(false);
	/** @type {HTMLInputElement | undefined} */
	let searchInputEl = $state();
	/** Mobile expanding nav panel. Desktop sidebar stays in-flow. */
	let mobileNavOpen = $state(false);
	/** 0..1 while dragging the menu open/closed; null when idle. */
	let menuDrag = $state(/** @type {number | null} */ (null));
	let isMobileViewport = $state(false);
	/** Ignore our own pushState/back while syncing the Android-back trap. */
	let ignoreNavPop = false;
	/** True while history.back() is closing the menu — popstate must not reopen it. */
	let navClosePending = false;
	/** Last path we armed — popstate with a path change is section back, not nav. */
	let navArmedPath = '';
	/** Only close the menu when the section href actually changes. */
	let lastNavHref = '';
	/** Detail pane covering the list — hide the menu chrome so back stays usable. */
	let detailCovering = $state(false);
	/**
	 * Chat width morph only after a compact↔wide toggle on Chat —
	 * never when first landing on the section (wide is the default).
	 */
	let chatWasCompact = $state(false);

	/*
	 * Compact↔wide morph only while staying on Chat. Leaving/entering the
	 * section must snap widths — never animate listSize across routes.
	 */
	let prevListSize = /** @type {'default' | 'chat'} */ (listSize);
	$effect(() => {
		const size = listSize;
		const compact = listCompact;
		const leftOrEnteredChat = prevListSize !== size;
		prevListSize = size;
		if (size !== 'chat') {
			chatWasCompact = false;
			return;
		}
		if (leftOrEnteredChat) {
			/* Fresh Chat land — wide default, no morph from prior section. */
			chatWasCompact = false;
			return;
		}
		if (compact) chatWasCompact = true;
	});

	$effect(() => {
		if (typeof window === 'undefined') return;
		const mq = window.matchMedia('(max-width: 767px)');
		const sync = () => {
			isMobileViewport = mq.matches;
			if (!mq.matches) closeMobileNav({ history: false });
		};
		sync();
		mq.addEventListener('change', sync);
		return () => mq.removeEventListener('change', sync);
	});

	/* Watch detail open/progress so the fixed menu circle never sits on the back button. */
	$effect(() => {
		const ws = listColEl?.parentElement;
		if (!ws || typeof window === 'undefined') return;

		const sync = () => {
			const detail = ws.querySelector('.detail');
			const open = Boolean(detail?.classList.contains('open'));
			const raw = getComputedStyle(ws).getPropertyValue('--detail-progress').trim();
			const p = raw ? Number.parseFloat(raw) : open ? 1 : 0;
			const covering = open || (Number.isFinite(p) && p > 0.02);
			detailCovering = covering;
			if (covering && mobileNavOpen) closeMobileNav({ history: false });
		};

		sync();
		const mo = new MutationObserver(sync);
		mo.observe(ws, {
			attributes: true,
			subtree: true,
			attributeFilter: ['class', 'style']
		});
		return () => mo.disconnect();
	});

	/** @param {string} href */
	function active(href) {
		if (isSectionActive) return isSectionActive(href);
		return activeHref === href || activeHref.startsWith(`${href}/`);
	}

	/** @param {ShellSection | ShellNavGroup} entry */
	function isNavGroup(entry) {
		return Boolean(entry && /** @type {ShellNavGroup} */ (entry).type === 'group');
	}

	const activeItem = $derived.by(() => {
		for (const entry of sections) {
			if (isNavGroup(entry)) {
				const group = /** @type {ShellNavGroup} */ (entry);
				const hit = group.items.find((item) => active(item.href));
				if (hit) return hit;
				continue;
			}
			const item = /** @type {ShellSection} */ (entry);
			if (active(item.href)) return item;
		}
		const first = sections[0];
		if (first && isNavGroup(first)) {
			const g = /** @type {ShellNavGroup} */ (first);
			return g.items[0] ?? { href: '', label: title || 'Alpha', icon: 'dot' };
		}
		return /** @type {ShellSection} */ (first) ?? { href: '', label: title || 'Alpha', icon: 'dot' };
	});

	const resolvedTitle = $derived(title || activeItem.label);
	const resolvedCreateKind = $derived(
		createKind !== undefined
			? createKind
			: 'createKind' in activeItem
				? activeItem.createKind
				: undefined
	);
	const filterActive = $derived(Boolean(filterOptions) && filterValue !== 'all');

	/** Flat nav hits for the Search modal (sidebar always shows full `sections`). */
	const searchHits = $derived.by(() => {
		const q = search.trim().toLowerCase();
		/** @type {{ href: string, label: string, icon?: string, faviconUrl?: string, spaceBrand?: boolean, group?: string }[]} */
		const out = [];
		for (const entry of sections) {
			if (isNavGroup(entry)) {
				const group = /** @type {ShellNavGroup} */ (entry);
				for (const item of group.items) {
					const hay = `${item.label} ${group.label}`.toLowerCase();
					if (!q || hay.includes(q)) {
						out.push({
							href: item.href,
							label: item.label,
							icon: item.icon,
							faviconUrl: item.faviconUrl,
							spaceBrand: item.spaceBrand,
							group: group.label
						});
					}
				}
				continue;
			}
			const item = /** @type {ShellSection} */ (entry);
			if (!q || item.label.toLowerCase().includes(q)) {
				out.push({
					href: item.href,
					label: item.label,
					icon: item.icon,
					faviconUrl: item.faviconUrl,
					spaceBrand: item.spaceBrand
				});
			}
		}
		return out;
	});

	/** Bundled image fallback when a space-brand blob/favicon fails — never NavIcon/globe. */
	const SPACE_BRAND_IMG_FALLBACK = '/icons/icon-192.png';

	/**
	 * Favicon / space-brand rows stay on <img>. Space brand never reveals stroke NavIcon.
	 * @param {Event} e
	 * @param {boolean} brand
	 */
	function onNavFaviconError(e, brand) {
		const el = /** @type {HTMLImageElement} */ (e.currentTarget);
		if (brand) {
			const cur = String(el.getAttribute('src') || '');
			if (el.dataset.fallback !== '1' && !cur.endsWith(SPACE_BRAND_IMG_FALLBACK)) {
				el.dataset.fallback = '1';
				el.style.display = '';
				el.src = SPACE_BRAND_IMG_FALLBACK;
				return;
			}
			el.style.display = 'none';
			el.dataset.failed = '1';
			return;
		}
		el.style.display = 'none';
		el.dataset.failed = '1';
		const fallback = el.nextElementSibling;
		if (fallback instanceof HTMLElement) fallback.hidden = false;
	}

	/** @param {string | undefined} kind */
	function createIconFor(kind) {
		const k = String(kind || '').toLowerCase();
		/** @type {Record<string, string>} */
		const map = {
			task: 'tasks',
			project: 'projects',
			note: 'notes',
			newsletter: 'publications',
			ledger: 'accounting',
			cost: 'accounting',
			sales: 'accounting',
			enterprise: 'enterprise',
			mail: 'mail',
			file: 'folder',
			photo: 'photos',
			poll: 'chat',
			website: 'website'
		};
		return map[k] || k;
	}

	function isMobileShell() {
		return typeof window !== 'undefined' && window.matchMedia('(max-width: 767px)').matches;
	}

	function detailOpen() {
		const workspace = listColEl?.parentElement;
		return Boolean(workspace?.querySelector('.detail.open'));
	}

	/** Drop the alphaNav flag without walking history (section changes). */
	function clearNavHistoryFlag() {
		if (typeof history === 'undefined' || !history.state?.alphaNav) return;
		ignoreNavPop = true;
		const next = { ...(history.state || {}) };
		delete next.alphaNav;
		history.replaceState(next, '');
		queueMicrotask(() => {
			ignoreNavPop = false;
		});
	}

	/**
	 * @param {{ history?: boolean }} [opts]
	 */
	function closeMobileNav(opts = {}) {
		const useHistory = opts.history !== false;
		menuDrag = null;
		if (!mobileNavOpen) {
			if (!useHistory) clearNavHistoryFlag();
			return;
		}
		if (
			useHistory &&
			typeof history !== 'undefined' &&
			history.state?.alphaNav &&
			isMobileShell()
		) {
			navClosePending = true;
			ignoreNavPop = true;
			history.back();
			mobileNavOpen = false;
			/* Popstate may arrive after the microtask — keep ignore until it runs. */
			requestAnimationFrame(() => {
				ignoreNavPop = false;
			});
			return;
		}
		mobileNavOpen = false;
		if (!useHistory) clearNavHistoryFlag();
	}

	/**
	 * @param {{ history?: boolean }} [opts]
	 */
	function openMobileNav(opts = {}) {
		if (detailOpen()) return;
		if (!isMobileShell()) return;
		const useHistory = opts.history !== false;
		menuDrag = null;
		mobileNavOpen = true;
		/* Never pushState for the drawer — it raced SvelteKit section goto. */
		void useHistory;
	}

	function toggleMobileNav() {
		/*
		 * Pill / peek toggle must NOT pushState — an alphaNav history entry per
		 * open left a duplicate same-URL frame that raced SvelteKit goto and
		 * emptied the list pane every other section switch.
		 * Android-back open still uses openMobileNav({ history: true }) in popstate.
		 */
		if (mobileNavOpen) closeMobileNav({ history: false });
		else openMobileNav({ history: false });
	}

	/** @param {string} href */
	function go(href) {
		/* Navigating away — don't history.back() into the previous section. */
		closeMobileNav({ history: false });
		onNavigate(href);
	}

	/** @param {string} kind */
	function create(kind) {
		createOpen = false;
		onCreate(kind);
	}

	function openSearch() {
		createOpen = false;
		searchOpen = true;
		closeMobileNav({ history: false });
		queueMicrotask(() => searchInputEl?.focus());
	}

	function closeSearch() {
		searchOpen = false;
		search = '';
	}

	function openCreate() {
		searchOpen = false;
		search = '';
		createOpen = true;
		closeMobileNav({ history: false });
	}

	/** @param {string} href */
	function pickSearchHit(href) {
		closeSearch();
		go(href);
	}

	function onListPlus() {
		if (resolvedCreateKind) onNavPlus(resolvedCreateKind);
		else if (createMenuItems.length) openCreate();
	}

	$effect(() => {
		activeHref;
		resolvedTitle;
		const root = listColEl;
		return attachChromeScrollFade(root, (v) => {
			listFade = v;
		});
	});

	/* Close drawer when the section href actually changes (not on every effect). */
	$effect(() => {
		const href = activeHref;
		if (lastNavHref && lastNavHref !== href) {
			closeMobileNav({ history: false });
		}
		lastNavHref = href;
	});

	/**
	 * Android-back “trap” pushState is DISABLED — it raced SvelteKit goto and
	 * left the list pane empty ~50% of the time on section switches (live repro).
	 * Same-URL back still opens the drawer via popstate without pushing.
	 */
	function rearmNavTrap() {
		/* no-op — see comment above */
	}

	$effect(() => {
		if (typeof window === 'undefined') return;
		if (!isMobileViewport) return;
		activeHref;
		navArmedPath = window.location.pathname;
	});

	/* Android / browser back: sync drawer to history, or open when still on this list. */
	$effect(() => {
		if (typeof window === 'undefined') return;

		/** @param {PopStateEvent} _e */
		function onPop(_e) {
			/* Honor in-flight dismiss even when ignoreNavPop is still set. */
			if (navClosePending) {
				navClosePending = false;
				mobileNavOpen = false;
				menuDrag = null;
				rearmNavTrap();
				return;
			}
			if (ignoreNavPop) return;
			if (!isMobileShell()) {
				mobileNavOpen = false;
				return;
			}
			const path = window.location.pathname;
			const pathChanged = Boolean(navArmedPath) && path !== navArmedPath;
			navArmedPath = path;

			/* Detail / comments overlays own their history — don't steal that back. */
			if (
				detailOpen() ||
				detailCovering ||
				history.state?.alphaLayer ||
				history.state?.alphaComments
			) {
				if (mobileNavOpen) mobileNavOpen = false;
				return;
			}
			if (pathChanged) {
				mobileNavOpen = false;
				menuDrag = null;
				return;
			}
			if (history.state?.alphaNav) {
				mobileNavOpen = true;
				menuDrag = null;
				return;
			}
			if (mobileNavOpen) {
				mobileNavOpen = false;
				menuDrag = null;
				/* Trap was consumed — re-arm so the next back opens nav again. */
				rearmNavTrap();
				return;
			}
			/* Same-URL back on a main list page opens the nav instead of leaving. */
			openMobileNav({ history: true });
		}

		window.addEventListener('popstate', onPop);
		return () => window.removeEventListener('popstate', onPop);
	});

	const menuProgress = $derived(
		menuDrag != null ? menuDrag : mobileNavOpen ? 1 : 0
	);

	/*
	 * Mobile nav open: paint html/body + theme-color with the drawer surface (--bg)
	 * so status-bar / home-indicator safe areas match the lighter nav panel (top + bottom).
	 * Closed → restore page surface (--shell). app.css: html.nav-menu-open { background: var(--bg) }.
	 *
	 * Do NOT remove nav-menu-open in $effect cleanup — Svelte re-runs cleanup before every
	 * re-execution, which flashed html/body back to --shell (wrong home-indicator strip).
	 * Toggle only in the effect body; clear on destroy.
	 */
	function syncNavMenuSurface(on) {
		if (typeof document === 'undefined') return;
		const root = document.documentElement;
		const body = document.body;
		root.classList.toggle('nav-menu-open', on);
		const probe = document.createElement('div');
		probe.style.cssText = `position:absolute;visibility:hidden;pointer-events:none;background:var(${on ? '--bg' : '--shell'})`;
		root.appendChild(probe);
		const color = getComputedStyle(probe).backgroundColor;
		probe.remove();
		if (!color) return;
		const meta = document.querySelector('meta[name="theme-color"]');
		if (meta) meta.setAttribute('content', color);
		/*
		 * Inline paint beats app.html first-paint / light-theme boot inline styles,
		 * which otherwise leave the home-indicator strip on --shell while the drawer
		 * is --bg. Clear on close so CSS tokens (and boot) own the surface again.
		 */
		if (on) {
			root.style.background = color;
			body.style.background = color;
		} else {
			root.style.removeProperty('background');
			body.style.removeProperty('background');
		}
	}

	$effect(() => {
		syncNavMenuSurface(isMobileViewport && menuProgress > 0.02);
	});

	onDestroy(() => {
		syncNavMenuSurface(false);
	});

	/*
	 * Touch swipe on the slid page:
	 * - Level-one lists → open/close the nav menu
	 * - Deeper pages (detail covering / alphaLayer / nested title) → history.back
	 * Detail chrome owns the visual underlay when `.detail.open`; shell stays hands-off then.
	 * Listen on the wrap (capture) so nav-peek / feed / PTR children still bubble here.
	 * Avoid shell-level touch-action: pan-y — it lets the browser cancel horizontal claims.
	 */
	$effect(() => {
		const root = workspaceWrapEl;
		if (!root || typeof window === 'undefined') return;

		const CLAIM = 10;
		const OPEN_AT = 0.2;
		const CLOSE_AT = 0.28;
		const BACK_AT = 0.28;

		function drawerSpan() {
			const shell = shellEl;
			const raw = shell
				? getComputedStyle(shell).getPropertyValue('--mobile-drawer').trim()
				: '67vw';
			if (raw.endsWith('vw')) return (parseFloat(raw) / 100) * window.innerWidth;
			if (raw.endsWith('px')) return parseFloat(raw);
			const n = parseFloat(raw);
			return Number.isFinite(n) && n > 0 ? n : window.innerWidth * 0.67;
		}

		/** Nested list / history layer — swipe-back instead of opening nav. */
		function deeperPage() {
			if (detailOpen() || detailCovering) return true;
			if (typeof history !== 'undefined' && history.state?.alphaLayer) return true;
			return Boolean(
				listColEl?.querySelector('.mobile-page-title.nested, [data-swipe-back]')
			);
		}

		let startX = 0;
		let startY = 0;
		/** @type {'open' | 'close' | 'back' | null} */
		let mode = null;
		let claimed = false;
		let startOpen = false;
		let tracking = false;
		let suppressToggleClick = false;
		/** Horizontal progress 0..1 while mode === 'back'. */
		let backProg = 0;

		/** @param {TouchEvent} e */
		function onStart(e) {
			if (!isMobileShell()) return;
			/* Detail column owns swipe-back + underlay when the pane is open. */
			if (detailOpen() || detailCovering) return;
			if (e.touches.length !== 1) return;
			const t = e.target;
			if (
				t instanceof Element &&
				t.closest(
					'input, textarea, select, [contenteditable="true"], [data-no-nav-swipe], [data-h-scroll]'
				)
			) {
				return;
			}

			const touch = e.touches[0];
			startX = touch.clientX;
			startY = touch.clientY;
			startOpen = mobileNavOpen || menuProgress > 0.5;
			claimed = false;
			tracking = true;
			backProg = 0;
			menuDrag = null;
			if (startOpen) mode = 'close';
			else if (deeperPage()) mode = 'back';
			else mode = 'open';
		}

		/** @param {TouchEvent} e */
		function onMove(e) {
			if (!tracking || !mode || e.touches.length !== 1) return;
			const touch = e.touches[0];
			const dx = touch.clientX - startX;
			const dy = touch.clientY - startY;
			const span = drawerSpan();
			if (!claimed) {
				if (Math.abs(dx) < CLAIM && Math.abs(dy) < CLAIM) return;
				/* Vertical wins — abandon (scroll / PTR). */
				if (Math.abs(dy) >= Math.abs(dx)) {
					tracking = false;
					mode = null;
					menuDrag = null;
					backProg = 0;
					return;
				}
				if ((mode === 'open' || mode === 'back') && dx < 0) {
					tracking = false;
					mode = null;
					return;
				}
				if (mode === 'close' && dx > 0) {
					tracking = false;
					mode = null;
					return;
				}
				claimed = true;
				suppressToggleClick = true;
			}
			e.preventDefault();
			if (mode === 'back') {
				backProg = Math.max(0, Math.min(1, dx / span));
				return;
			}
			if (mode === 'open') {
				menuDrag = Math.max(0, Math.min(1, dx / span));
			} else {
				menuDrag = Math.max(0, Math.min(1, 1 + dx / span));
			}
		}

		function onEnd() {
			if (!tracking) {
				suppressToggleClick = false;
				return;
			}
			const p = mode === 'back' ? backProg : menuDrag;
			const wasClaimed = claimed;
			const endMode = mode;
			tracking = false;
			mode = null;
			claimed = false;
			backProg = 0;
			if (!wasClaimed || p == null) {
				menuDrag = null;
				queueMicrotask(() => {
					suppressToggleClick = false;
				});
				return;
			}
			if (endMode === 'back') {
				menuDrag = null;
				if (p >= BACK_AT && typeof history !== 'undefined') history.back();
				queueMicrotask(() => {
					suppressToggleClick = false;
				});
				return;
			}
			const nextOpen = startOpen ? p > 1 - CLOSE_AT : p >= OPEN_AT;
			menuDrag = null;
			/* history:false — same as pill toggle; avoid duplicate same-URL frames. */
			if (nextOpen) openMobileNav({ history: false });
			else closeMobileNav({ history: false });
			queueMicrotask(() => {
				suppressToggleClick = false;
			});
		}

		function onClickCapture(e) {
			if (!suppressToggleClick) return;
			const t = e.target;
			if (t instanceof Element && t.closest('.mobile-title-pill, .nav-peek')) {
				e.preventDefault();
				e.stopPropagation();
			}
		}

		root.addEventListener('touchstart', onStart, { passive: true, capture: true });
		root.addEventListener('touchmove', onMove, { passive: false, capture: true });
		root.addEventListener('touchend', onEnd, { capture: true });
		root.addEventListener('touchcancel', onEnd, { capture: true });
		root.addEventListener('click', onClickCapture, true);
		return () => {
			root.removeEventListener('touchstart', onStart, true);
			root.removeEventListener('touchmove', onMove, true);
			root.removeEventListener('touchend', onEnd, true);
			root.removeEventListener('touchcancel', onEnd, true);
			root.removeEventListener('click', onClickCapture, true);
		};
	});
</script>

<div
	class="shell {feedPadClass}"
	class:nav-open={mobileNavOpen}
	class:nav-dragging={menuDrag != null}
	class:detail-covering={detailCovering}
	class:list-chat={listSize === 'chat'}
	class:chat-compact={listSize === 'chat' && listCompact}
	class:chat-was-compact={listSize === 'chat' && chatWasCompact}
	class:comms-feed={commsFeed}
	bind:this={shellEl}
	style="--menu-progress: {menuProgress}"
>
	{#if banners}
		{@render banners()}
	{/if}

	{#snippet sidebarBody()}
		<header class="side-top">
			{#if sidebarLead || accountLabel}
				<button
					type="button"
					class="account-pill"
					class:on={active('/settings')}
					aria-label="Settings"
					aria-current={active('/settings') ? 'page' : undefined}
					onclick={() => go('/settings')}
				>
					{#if sidebarLead}
						<span class="account-pic">{@render sidebarLead()}</span>
					{/if}
					{#if accountLabel}
						<span class="account-name">{accountLabel}</span>
					{/if}
				</button>
			{:else}
				<span class="side-spacer"></span>
			{/if}
			<span class="side-spacer"></span>
			{#if searchEnabled}
				<button type="button" class="top-search" aria-label="Search" onclick={openSearch}>
					<Search size={isMobileViewport ? 18 : 13} />
				</button>
			{/if}
			{#if createMenuItems.length}
				<button
					type="button"
					class="top-plus"
					aria-label="Create"
					aria-expanded={createOpen}
					onclick={openCreate}
				>
					<svg class="chrome-ico" width="16" height="16" viewBox="0 0 16 16" fill="none" aria-hidden="true">
						<path
							d="M8 3.25v9.5M3.25 8h9.5"
							stroke="currentColor"
							stroke-width="1.5"
							stroke-linecap="round"
						/>
					</svg>
				</button>
			{/if}
		</header>

		<nav class="sidebar-nav" aria-label="Sections">
			{#each sections as entry (isNavGroup(entry) ? `g:${/** @type {ShellNavGroup} */ (entry).id}` : /** @type {ShellSection} */ (entry).href)}
				{#if isNavGroup(entry)}
					{@const group = /** @type {ShellNavGroup} */ (entry)}
					<div class="nav-group" role="group" aria-label={group.label}>
						<div class="nav-group-head">
							<span class="nav-group-eyebrow eyebrow-label-xs">{group.label}</span>
							{#if group.creatable}
								<button
									type="button"
									class="row-plus"
									aria-label={`Add ${group.label}`}
									title={`Add ${group.label}`}
									onclick={() => {
										if (onGroupPlus) onGroupPlus(group.id);
										else if (group.createKind) onNavPlus(group.createKind);
									}}
								>
									<svg width="12" height="12" viewBox="0 0 12 12" fill="none" aria-hidden="true">
										<path
											d="M6 1.5v9M1.5 6h9"
											stroke="currentColor"
											stroke-width="1.6"
											stroke-linecap="round"
										/>
									</svg>
								</button>
							{/if}
						</div>
						{#each group.items as item (item.id || item.href)}
							{@const rowActive = active(item.href)}
							{@const fav = String(item.faviconUrl || '').trim()}
							{@const brand = Boolean(item.spaceBrand)}
							<div class="nav-block" class:active={rowActive}>
								<div class="nav-row">
									<button
										type="button"
										class="nav-item"
										class:active={rowActive}
										onclick={() => go(item.href)}
									>
										<span
											class="icon-wrap"
											class:favicon={Boolean(fav)}
											class:space-brand={brand}
											class:empty={rowActive && !fav}
										>
											{#if fav}
												<img
													class="nav-favicon"
													class:space-brand={brand}
													src={fav}
													alt=""
													width="18"
													height="18"
													loading="lazy"
													onerror={(e) => onNavFaviconError(e, brand)}
												/>
												{#if !brand && item.icon}
													<span class="nav-ico-fallback" hidden>
														<NavIcon name={item.icon} size={18} />
													</span>
												{/if}
											{:else if item.icon}
												<NavIcon name={item.icon} size={18} />
											{/if}
										</span>
										<span class="nav-label">{item.label}</span>
									</button>
								</div>
							</div>
						{/each}
					</div>
				{:else}
					{@const item = /** @type {ShellSection} */ (entry)}
					{@const count = item.countKey ? counts[item.countKey] : null}
					{@const unread =
						item.countKey && unreadCounts[item.countKey] != null
							? Number(unreadCounts[item.countKey]) || 0
							: 0}
					{@const rowActive = active(item.href)}
					{@const kids = item.children || []}
					{@const expanded = rowActive && kids.length > 0}
					{@const fav = String(item.faviconUrl || '').trim()}
					{@const brand = Boolean(item.spaceBrand)}
					{@const png = !fav && rowActive && item.icon ? navPngUrl(item.icon) : null}
					<div
						class="nav-block"
						class:active={rowActive}
						class:expanded={expanded}
					>
						<div
							class="nav-row"
							class:creatable={Boolean(item.createKind)}
						>
							<button
								type="button"
								class="nav-item"
								class:active={rowActive}
								onclick={() => go(item.href)}
							>
								<span
									class="icon-wrap"
									class:favicon={Boolean(fav)}
									class:space-brand={brand}
									class:png={Boolean(png)}
									class:empty={rowActive && !fav && !png}
								>
									{#if fav}
										<img
											class="nav-favicon"
											class:space-brand={brand}
											src={fav}
											alt=""
											width="18"
											height="18"
											loading="lazy"
											onerror={(e) => onNavFaviconError(e, brand)}
										/>
										{#if !brand && item.icon}
											<span class="nav-ico-fallback" hidden>
												<NavIcon name={item.icon} size={18} />
											</span>
										{/if}
									{:else if rowActive}
										{#if png}
											<img class="nav-png" src={png} alt="" width="18" height="18" />
										{/if}
									{:else if item.icon}
										<NavIcon name={item.icon} size={18} />
									{/if}
								</span>
								<span class="nav-label">{item.label}</span>
							</button>
							{#if item.createKind || (count != null && count > 0) || unread > 0}
								<span class="row-end">
									{#if unread > 0}
										<UnreadMark count={unread} />
									{/if}
									{#if (count != null && count > 0) || item.createKind}
										<span class="row-end-stack">
											{#if count != null && count > 0}
												<span class="count">{count}</span>
											{/if}
											{#if item.createKind}
												<button
													type="button"
													class="row-plus"
													aria-label={`New ${item.createKind}`}
													onclick={() => onNavPlus(item.createKind)}
												>
													<svg width="12" height="12" viewBox="0 0 12 12" fill="none" aria-hidden="true">
														<path
															d="M6 1.5v9M1.5 6h9"
															stroke="currentColor"
															stroke-width="1.6"
															stroke-linecap="round"
														/>
													</svg>
												</button>
											{/if}
										</span>
									{/if}
								</span>
							{/if}
						</div>
						{#if expanded}
							<div class="nav-sub" role="group" aria-label={item.label}>
								{#each kids as child (child.id)}
									<button
										type="button"
										class="nav-sub-item"
										class:active={Boolean(child.active)}
										onclick={() => {
											onNavChild(item.href, child.id);
											closeMobileNav({ history: false });
										}}
									>
										{child.label}
									</button>
								{/each}
							</div>
						{/if}
					</div>
				{/if}
			{/each}
		</nav>
		{#if sidebarTrail}
			<div class="sidebar-trail">
				{@render sidebarTrail()}
			</div>
		{/if}
	{/snippet}

	<aside class="sidebar" id="alpha-shell-sidebar">
		<div
			class="sidebar-inner"
			aria-hidden={isMobileViewport && menuProgress < 0.02 ? true : undefined}
		>
			{@render sidebarBody()}
		</div>
	</aside>

	<div class="workspace-wrap" bind:this={workspaceWrapEl}>
		{#if isMobileViewport}
			<button
				type="button"
				class="nav-peek"
				class:on={menuProgress > 0.02}
				aria-label="Close menu"
				tabindex={menuProgress > 0.02 ? 0 : -1}
				onclick={() => {
					if (menuProgress <= 0.02) return;
					closeMobileNav();
				}}
			></button>
		{/if}
		<div class="workspace">
			<section
				class="list-col"
				class:wide={wideList}
				class:chat={listSize === 'chat'}
				class:list-top-hidden={listTopHidden}
				aria-label={resolvedTitle}
				bind:this={listColEl}
			>
				<header
					class="list-top"
					class:hidden={listTopHidden}
					style="--chrome-scroll-fade: {listFade}"
					aria-hidden={listTopHidden || undefined}
				>
					{#if isMobileViewport}
						{@const pillMatched =
							Boolean(activeHref) &&
							(activeItem.href === activeHref ||
								(activeItem.href
									? activeHref.startsWith(`${activeItem.href}/`)
									: false) ||
								active(activeItem.href))}
						{@const pillFav =
							titleIcon === false
								? ''
								: typeof titleIcon === 'string'
									? ''
									: pillMatched
										? String(activeItem.faviconUrl || '').trim()
										: ''}
						{@const pillBrand = Boolean(pillFav && activeItem.spaceBrand)}
						{@const pillPng =
							titleIcon === false || pillFav
								? null
								: typeof titleIcon === 'string'
									? navPngUrl(titleIcon)
									: /* Off-nav roots (Settings) fall back to Chat in activeItem — never paint Chat’s PNG on a foreign title. */
										pillMatched
										? navPngUrl(activeItem.icon)
										: null}
						{@const pillCompact = menuProgress > 0.55}
						<button
							type="button"
							class="mobile-title-pill"
							class:compact={pillCompact}
							aria-label={mobileNavOpen ? 'Close menu' : 'Open menu'}
							aria-expanded={mobileNavOpen}
							aria-controls="alpha-shell-sidebar"
							onclick={toggleMobileNav}
						>
							<span class="pill-chev" aria-hidden="true">
								{#if pillCompact}
									<!-- Shared chrome chevron, mirrored → (round dismiss control) -->
									<svg width="22" height="22" viewBox="0 0 18 18" fill="none">
										<path
											d="M6.5 3.5 12 9l-5.5 5.5"
											stroke="currentColor"
											stroke-width="1.7"
											stroke-linecap="round"
											stroke-linejoin="round"
										/>
									</svg>
								{:else}
									<!-- Same back chevron as ChromeMobileBar / ListTitleChrome -->
									<svg width="22" height="22" viewBox="0 0 18 18" fill="none">
										<path
											d="M11.5 3.5 6 9l5.5 5.5"
											stroke="currentColor"
											stroke-width="1.7"
											stroke-linecap="round"
											stroke-linejoin="round"
										/>
									</svg>
								{/if}
							</span>
							{#if !pillCompact && pillFav}
								<span class="pill-ico favicon" class:space-brand={pillBrand} aria-hidden="true">
									<img
										class="nav-favicon"
										class:space-brand={pillBrand}
										src={pillFav}
										alt=""
										width="22"
										height="22"
										onerror={(e) => onNavFaviconError(e, pillBrand)}
									/>
								</span>
							{:else if !pillCompact && pillPng}
								<span class="pill-ico png" aria-hidden="true">
									<img class="nav-png" src={pillPng} alt="" width="22" height="22" />
								</span>
							{/if}
							{#if !pillCompact}
								<span class="pill-title-stack" class:titles-syncing={syncing} aria-busy={syncing || undefined}>
									<span class="pill-label">{resolvedTitle}</span>
									{#if syncing}
										<span class="pill-loader"><MetaballLoader size="sm" label="Refreshing" /></span>
									{/if}
								</span>
							{/if}
						</button>
					{/if}
					{#if listTitleSlot}
						<div class="list-title-slot" class:mobile-pill-owns-title={isMobileViewport}>
							{@render listTitleSlot()}
						</div>
					{:else}
						<div
							class="list-title-row desktop-only-title"
							class:titles-syncing={syncing}
							aria-busy={syncing || undefined}
						>
							<h1 class="list-title">{resolvedTitle}</h1>
							{#if syncing}
								<span class="title-loader"><MetaballLoader size="sm" label="Refreshing" /></span>
							{/if}
						</div>
						{#if !isMobileViewport}
							<div class="mobile-page-title">
								<span
									class="mobile-title-stack"
									class:titles-syncing={syncing}
									aria-busy={syncing || undefined}
								>
									<h1 class="mobile-page-name">{resolvedTitle}</h1>
									{#if syncing}
										<span class="title-loader"><MetaballLoader size="sm" label="Refreshing" /></span>
									{/if}
								</span>
							</div>
						{/if}
					{/if}
					<div class="list-actions">
						{#if listActionsSlot}
							{@render listActionsSlot()}
						{/if}
						{#if defaultListActions}
							{#if filterOptions}
								<div class="filter-wrap">
									<button
										type="button"
										class="chrome-btn"
										class:on={filterActive}
										aria-label="Filter"
										aria-expanded={filterOpen}
										aria-haspopup="listbox"
										onclick={() => (filterOpen = !filterOpen)}
									>
										<svg class="chrome-ico" width="16" height="16" viewBox="0 0 16 16" fill="none" aria-hidden="true">
											<path
												d="M2.5 4h11M4.5 8h7M6.5 12h3"
												stroke="currentColor"
												stroke-width="1.5"
												stroke-linecap="round"
											/>
										</svg>
									</button>
									{#if filterOpen}
										<button
											type="button"
											class="menu-scrim"
											aria-label="Close filter"
											onclick={() => (filterOpen = false)}
										></button>
										<div class="filter-menu alpha-popover" role="listbox">
											<div class="alpha-popover-list">
												{#each filterOptions as opt (opt.id)}
													{@const Icon = opt.icon}
													<button
														type="button"
														class="alpha-popover-item"
														class:on={filterValue === opt.id}
														role="option"
														aria-selected={filterValue === opt.id}
														onclick={() => {
															onFilterChange(opt.id);
															filterOpen = false;
														}}
													>
														<span class="alpha-popover-ico" aria-hidden="true">
															{#if Icon}
																<Icon size={16} />
															{:else}
																<ChevronRight size={14} />
															{/if}
														</span>
														<span class="alpha-popover-label">{opt.label}</span>
													</button>
												{/each}
											</div>
										</div>
									{/if}
								</div>
							{/if}
							{#if resolvedCreateKind}
								<button
									type="button"
									class="chrome-btn list-plus"
									aria-label={`New ${resolvedCreateKind}`}
									onclick={onListPlus}
								>
									<svg class="chrome-ico" width="16" height="16" viewBox="0 0 16 16" fill="none" aria-hidden="true">
										<path
											d="M8 3.25v9.5M3.25 8h9.5"
											stroke="currentColor"
											stroke-width="1.5"
											stroke-linecap="round"
										/>
									</svg>
								</button>
							{/if}
						{/if}
					</div>
				</header>
				{#if onRefresh}
					<PullToRefresh onRefresh={onRefresh}>
						<!--
							Never {#if}-unmount children while blanking — that tore out
							SvelteKit’s page outlet mid-goto and left the pane empty.
							CSS-hide only; SK can swap the route under the blank.
						-->
						<div class="pane-inner" class:list-blank={listBlank}>
							{@render children?.()}
						</div>
					</PullToRefresh>
				{:else}
					<div class="pane-inner" class:list-blank={listBlank}>
						{@render children?.()}
					</div>
				{/if}
				<div class="list-bottom-fade" aria-hidden="true"></div>
			</section>
			{#if !wideList && detail}
				{@render detail()}
			{/if}
		</div>
	</div>

	{#if sheets}
		{@render sheets()}
	{/if}

	{#if searchEnabled}
		<Modal bind:open={searchOpen} label="Search">
			<div class="shell-modal">
				<SheetTopbar title="Search" trailing="none" onclose={closeSearch} />
				<div class="shell-modal-body">
					<label class="shell-search-field">
						<span class="sr-only">Search</span>
						<span class="shell-search-ico" aria-hidden="true"><Search size={16} /></span>
						<input
							bind:this={searchInputEl}
							type="search"
							placeholder="Search"
							bind:value={search}
							autocomplete="off"
							onkeydown={(e) => {
								if (e.key === 'Escape') closeSearch();
							}}
						/>
					</label>
					<div class="shell-search-hits" role="listbox" aria-label="Results">
						{#each searchHits as hit (hit.href + (hit.group || '') + hit.label)}
							{@const fav = String(hit.faviconUrl || '').trim()}
							{@const brand = Boolean(hit.spaceBrand)}
							{@const png = !fav && hit.icon ? navPngUrl(hit.icon) : null}
							<button
								type="button"
								class="shell-search-hit"
								role="option"
								onclick={() => pickSearchHit(hit.href)}
							>
								<span class="shell-search-hit-ico" aria-hidden="true">
									{#if fav}
										<img
											class="nav-favicon"
											class:space-brand={brand}
											src={fav}
											alt=""
											width="18"
											height="18"
											onerror={(e) => onNavFaviconError(e, brand)}
										/>
									{:else if png}
										<img class="nav-png" src={png} alt="" width="18" height="18" />
									{:else}
										<NavIcon name={hit.icon || 'dot'} size={18} />
									{/if}
								</span>
								<span class="shell-search-hit-text">
									<span class="shell-search-hit-label">{hit.label}</span>
									{#if hit.group}
										<span class="shell-search-hit-group">{hit.group}</span>
									{/if}
								</span>
							</button>
						{:else}
							<p class="shell-search-empty">No matches</p>
						{/each}
					</div>
				</div>
			</div>
		</Modal>
	{/if}

	{#if createMenuItems.length}
		<Modal bind:open={createOpen} label="Create">
			<div class="shell-modal">
				<SheetTopbar title="Create" trailing="none" onclose={() => (createOpen = false)} />
				<div class="shell-modal-body">
					<div class="create-grid" role="menu">
						{#each createMenuItems as item (item.kind)}
							{@const icon = item.icon || createIconFor(item.kind)}
							{@const png = navPngUrl(icon)}
							<button
								type="button"
								class="create-tile"
								role="menuitem"
								onclick={() => create(item.kind)}
							>
								<span class="create-tile-ico" aria-hidden="true">
									{#if png}
										<img class="create-tile-png" src={png} alt="" />
									{:else}
										<span class="create-tile-nav"><NavIcon name={icon} size={28} /></span>
									{/if}
								</span>
								<span class="create-tile-label">{item.label}</span>
							</button>
						{/each}
					</div>
				</div>
			</div>
		</Modal>
	{/if}
</div>

<style>
	.shell {
		/* Pad ≈ control row; fade is short/soft so content stays readable until the bar */
		/* Clears the solid control row; fade veil is scroll-driven */
		/* Default = Chat / reverse-scroll; `.feed-pad` aligns Tasks / Mail / … */
		--feed-pad-top: var(--chrome-mobile, 2.75rem);
		--feed-pad-bottom: 2.25rem;
		--chrome-fade: var(--bg);
		--chrome-scroll-fade: 0;
		display: flex;
		/* --app-height tracks visualViewport so the keyboard shrinks the app */
		height: var(--app-height, 100dvh);
		max-height: var(--app-height, 100dvh);
		width: 100%;
		overflow: hidden;
		background: var(--shell);
	}

	.sidebar {
		width: var(--side);
		flex-shrink: 0;
		padding: 1.0625rem 1rem 0.75rem;
		display: flex;
		flex-direction: column;
		min-height: 0;
		overflow: hidden;
		gap: 0.625rem;
		background: var(--shell);
		color: var(--white);
		box-sizing: border-box;
	}

	.sidebar-inner {
		display: flex;
		flex-direction: column;
		min-height: 0;
		flex: 1;
		gap: 0.625rem;
		overflow: hidden;
	}

	.mobile-title-pill,
	.nav-peek,
	.mobile-nav-slot {
		display: none;
	}

	/* Wrapper must not alter desktop title flex layout */
	.list-title-slot {
		display: contents;
	}

	.side-top {
		display: flex;
		align-items: center;
		gap: 0.625rem;
		flex-shrink: 0;
		min-height: 2.75rem;
		padding: 0;
	}

	/* Pfp + name — glass pill when Settings is selected; chrome invisible otherwise */
	.account-pill {
		display: inline-flex;
		align-items: center;
		gap: 0.625rem;
		flex: 1 1 auto;
		min-width: 0;
		height: 2.75rem;
		margin: 0;
		/* 5px around 34px pfp → 44px outer, same as round chrome */
		padding: 0.3125rem 0.75rem 0.3125rem 0.3125rem;
		border: 1px solid transparent;
		border-radius: 9999px;
		background: transparent;
		color: var(--fg-66);
		cursor: pointer;
		box-sizing: border-box;
		-webkit-tap-highlight-color: transparent;
		transition:
			background 0.18s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
			color 0.18s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
	}

	.account-pill.on {
		background:
			linear-gradient(var(--nav-selected, var(--glass)), var(--nav-selected, var(--glass)))
				padding-box,
			var(--glass-shine) border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
		color: var(--fg);
	}

	.account-pic {
		display: grid;
		place-items: center;
		flex-shrink: 0;
		line-height: 0;
	}

	.account-name {
		flex: 1 1 auto;
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: 0.8125rem;
		font-weight: 600;
		letter-spacing: -0.01em;
		color: inherit;
		text-align: left;
	}

	.search {
		flex: 1;
		min-width: 0;
		height: 1.75rem;
		display: flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0 0.625rem;
		border: 1px solid transparent;
		border-radius: 0.625rem;
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		color: var(--fg-66);
		box-sizing: border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
	}

	.search-icon {
		flex-shrink: 0;
	}

	.search input {
		flex: 1;
		min-width: 0;
		height: 100%;
		border: 0;
		background: transparent;
		color: var(--fg);
		font: inherit;
		font-size: 0.8125rem;
		outline: none;
	}

	.search input::placeholder {
		color: var(--fg-33);
	}

	.shell-modal {
		display: flex;
		flex-direction: column;
		max-height: min(78vh, 640px);
		box-sizing: border-box;
	}

	.shell-modal-body {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		padding: 0.25rem 0.75rem 1rem;
		box-sizing: border-box;
		min-height: 0;
	}

	.shell-search-field {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		min-height: 2.75rem;
		padding: 0 0.75rem;
		border-radius: 0.875rem;
		border: 1px solid var(--fg-8);
		background: var(--fg-4);
		color: var(--fg-66);
		box-sizing: border-box;
	}

	.shell-search-field:focus-within {
		border-color: var(--fg-16, color-mix(in srgb, var(--fg) 16%, transparent));
		color: var(--fg);
	}

	.shell-search-ico {
		flex-shrink: 0;
		display: grid;
		place-items: center;
	}

	.shell-search-field input {
		flex: 1;
		min-width: 0;
		height: 2.5rem;
		border: 0;
		background: transparent;
		color: var(--fg);
		font: inherit;
		font-size: 0.9375rem;
		font-weight: 500;
		outline: none;
	}

	.shell-search-field input::placeholder {
		color: var(--fg-33);
		font-weight: 500;
	}

	.shell-search-hits {
		display: flex;
		flex-direction: column;
		gap: 0.125rem;
		min-height: 0;
		overflow-y: auto;
		margin: 0 -0.25rem;
		padding: 0 0.25rem;
	}

	.shell-search-hit {
		display: flex;
		align-items: center;
		gap: 0.625rem;
		width: 100%;
		padding: 0.625rem 0.625rem;
		border: 0;
		border-radius: 0.75rem;
		background: transparent;
		color: var(--fg);
		font: inherit;
		text-align: left;
		cursor: pointer;
		box-sizing: border-box;
	}

	.shell-search-hit:hover {
		background: var(--fg-4);
	}

	.shell-search-hit-ico {
		flex-shrink: 0;
		width: 1.375rem;
		height: 1.375rem;
		display: grid;
		place-items: center;
		color: var(--fg-66);
	}

	.shell-search-hit-text {
		flex: 1;
		min-width: 0;
		display: flex;
		flex-direction: column;
		gap: 1px;
	}

	.shell-search-hit-label {
		font-size: 0.875rem;
		font-weight: 600;
		color: var(--fg);
	}

	.shell-search-hit-group {
		font-size: 0.75rem;
		font-weight: 500;
		color: var(--fg-33);
	}

	.shell-search-empty {
		margin: 0.75rem 0.25rem;
		font-size: 0.8125rem;
		font-weight: 500;
		color: var(--fg-33);
		text-align: center;
	}

	.create-grid {
		display: grid;
		grid-template-columns: repeat(3, minmax(0, 1fr));
		gap: 0.625rem;
	}

	.create-tile {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 0.5rem;
		min-height: 96px;
		padding: 0.875rem 0.5rem 0.75rem;
		border: 1px solid var(--fg-8);
		border-radius: 1.125rem;
		background: var(--fg-4);
		color: var(--fg);
		font: inherit;
		cursor: pointer;
		box-sizing: border-box;
		transition: transform 0.15s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
	}

	.create-tile:hover {
		background: var(--fg-8);
		transform: scale(1.02);
	}

	.create-tile:active {
		transform: scale(0.97);
	}

	.create-tile-ico {
		width: 2.25rem;
		height: 2.25rem;
		display: grid;
		place-items: center;
		color: var(--fg-66);
	}

	.create-tile-png {
		display: block;
		width: 2rem;
		height: 2rem;
		object-fit: contain;
		border-radius: 0.5rem;
	}

	.create-tile-nav {
		display: grid;
		place-items: center;
		color: var(--fg-66);
	}

	.create-tile-label {
		font-size: 0.75rem;
		font-weight: 600;
		letter-spacing: -0.01em;
		text-align: center;
		line-height: 1.25;
		color: var(--fg);
	}

	@media (min-width: 480px) {
		.create-grid {
			grid-template-columns: repeat(4, minmax(0, 1fr));
		}
	}

	.filter-wrap {
		position: relative;
		flex-shrink: 0;
	}

	/* Match desktop list-top chrome-btn (rounded square) */
	.top-search,
	.top-plus {
		width: 1.75rem;
		height: 1.75rem;
		display: grid;
		place-items: center;
		border: 1px solid transparent;
		border-radius: 0.625rem;
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		color: var(--fg-66);
		cursor: pointer;
		padding: 0;
		box-sizing: border-box;
		flex-shrink: 0;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
		transition: transform 0.15s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
	}

	.top-search:hover,
	.top-plus:hover {
		transform: scale(1.06);
	}

	.top-search:active,
	.top-plus:active {
		transform: scale(0.96);
	}

	.icon-wrap.png,
	.icon-wrap.empty {
		width: 1.125rem;
		height: 1.125rem;
	}

	.nav-png {
		display: block;
		width: 1.125rem;
		height: 1.125rem;
		object-fit: contain;
		border-radius: 0.25rem;
	}

	.menu-scrim {
		position: fixed;
		inset: 0;
		z-index: 40;
		border: 0;
		background: transparent;
		cursor: default;
	}

	.create-menu,
	.filter-menu {
		position: absolute;
		top: calc(100% + 0.375rem);
		right: 0;
		z-index: 50;
		min-width: 168px;
		padding: 0;
		overflow: hidden;
	}

	/* Legacy create-menu rows (pre-popover). Filter uses .alpha-popover-* */
	.create-menu {
		border: 1px solid var(--glass-border);
		border-radius: 0.75rem;
		background: var(--glass);
		box-shadow: 0 8px 28px var(--shadow);
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
	}

	.create-menu button {
		display: block;
		width: 100%;
		padding: 0.5rem 0.75rem;
		border: 0;
		border-radius: 0;
		background: transparent;
		color: var(--fg-66);
		font: inherit;
		font-size: 0.9375rem;
		font-weight: 500;
		text-align: left;
		cursor: pointer;
	}

	.create-menu button:not(:last-child) {
		border-bottom: 0.33px solid var(--fg-8);
	}

	.create-menu button:hover {
		background: var(--fg-4);
		color: var(--fg);
	}

	.sidebar-nav {
		display: flex;
		flex-direction: column;
		gap: 0.125rem;
		flex: 1;
		min-height: 0;
		overflow-y: auto;
	}

	.sidebar-trail {
		flex-shrink: 0;
		margin-top: auto;
		min-width: 0;
		padding-top: 0.5rem;
	}

	/*
	 * Selected glass lives on the block so Mail can expand the same pill into a
	 * title + quiet sub-items panel. Always-on 1px transparent border + fixed
	 * icon box keep row height stable when switching sections. Expanded radius
	 * = half the collapsed pill height (pad-y*2 + ico + 2px border).
	 */
	.nav-block {
		--nav-ico: 18px;
		--nav-gap: 0.625rem;
		--nav-pad-x: 10px;
		--nav-pad-y: 0.5rem;
		--nav-expanded-radius: calc(var(--nav-pad-y) + (var(--nav-ico) / 2) + 1px);
		border: 1px solid transparent;
		border-radius: 999px;
		box-sizing: border-box;
		overflow: hidden;
	}

	.nav-block.active {
		background:
			linear-gradient(var(--nav-selected, var(--glass)), var(--nav-selected, var(--glass)))
				padding-box,
			var(--glass-shine) border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
	}

	.nav-block.expanded {
		border-radius: var(--nav-expanded-radius);
		padding-bottom: 0.25rem;
	}

	.nav-row {
		position: relative;
		display: flex;
		align-items: center;
		border-radius: 999px;
		overflow: hidden;
	}

	.nav-block:not(.active) .nav-row:hover {
		background: var(--white4);
	}

	.nav-block.active .nav-item {
		color: var(--white);
	}

	.nav-item {
		display: flex;
		align-items: center;
		gap: 0.625rem;
		flex: 1;
		min-width: 0;
		padding: 0.5rem 0.625rem;
		border: 0;
		border-radius: 999px;
		background: transparent;
		color: var(--white66);
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.875rem;
		font-weight: 500;
		text-align: left;
		cursor: pointer;
	}

	.nav-item.active {
		color: var(--white);
	}

	.nav-group {
		display: flex;
		flex-direction: column;
		gap: 1px;
		margin-top: 0.625rem;
		padding-top: 0.25rem;
	}

	.nav-group-head {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 0.5rem;
		padding: 0.25rem 0.75rem 0.375rem;
		min-height: 1.75rem;
		box-sizing: border-box;
	}

	.nav-group-eyebrow {
		font-size: 0.625rem;
		font-weight: 500;
		text-transform: uppercase;
		letter-spacing: 0.18em;
		color: var(--white33, var(--fg-33));
	}

	.nav-favicon {
		display: block;
		width: 1.125rem;
		height: 1.125rem;
		border-radius: 0.25rem;
		object-fit: cover;
	}

	.nav-favicon.space-brand {
		border-radius: 999px;
		box-sizing: border-box;
		border: 1px solid var(--fg-16, color-mix(in srgb, var(--fg, var(--ink)) 16%, transparent));
	}

	.icon-wrap.favicon {
		overflow: hidden;
		border-radius: 0.25rem;
	}

	.icon-wrap.favicon.space-brand {
		overflow: visible;
		border-radius: 999px;
	}

	.nav-ico-fallback {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 1.125rem;
		height: 1.125rem;
	}

	.nav-sub {
		display: flex;
		flex-direction: column;
		gap: 0;
		padding: 0 0 0.125rem;
	}

	/* Left pad = nav-item pad + icon + gap so labels line up with Mail. */
	.nav-sub-item {
		display: block;
		width: 100%;
		padding: 0.1875rem var(--nav-pad-x, 0.625rem) 0.1875rem
			calc(var(--nav-pad-x, 0.625rem) + var(--nav-ico, 1.125rem) + var(--nav-gap, 0.625rem));
		border: 0;
		border-radius: 0;
		background: transparent;
		color: var(--white33, var(--fg-33));
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.8125rem;
		font-weight: 500;
		text-align: left;
		cursor: pointer;
		box-sizing: border-box;
	}

	.nav-sub-item:hover {
		color: var(--white66, var(--fg-66));
	}

	.nav-sub-item.active {
		color: var(--white);
	}

	.nav-row.creatable .nav-item,
	.nav-row:has(.row-end) .nav-item {
		padding-right: 3.5rem;
	}

	.icon-wrap {
		width: 1.125rem;
		height: 1.125rem;
		flex-shrink: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		color: inherit;
	}

	.nav-label {
		flex: 1;
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.row-end {
		position: absolute;
		right: 0.375rem;
		top: 50%;
		min-width: 1.5rem;
		height: 1.5rem;
		transform: translateY(-50%);
		display: flex;
		align-items: center;
		justify-content: flex-end;
		gap: 0.375rem;
		pointer-events: none;
	}

	.row-end-stack {
		position: relative;
		width: 1.5rem;
		height: 1.5rem;
		display: grid;
		place-items: center;
		flex-shrink: 0;
	}

	.count,
	.row-plus {
		grid-area: 1 / 1;
	}

	.count {
		display: grid;
		place-items: center;
		width: 1.5rem;
		height: 1.5rem;
		margin: 0;
		padding: 0;
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.75rem;
		font-weight: 500;
		font-style: normal;
		line-height: 1;
		letter-spacing: 0;
		font-variant-numeric: lining-nums;
		font-feature-settings: 'lnum' 1, 'tnum' 0, 'onum' 0;
		color: var(--white33);
		text-align: center;
		transition: opacity 0.22s var(--ease);
	}

	.nav-row.active .count {
		color: var(--white66);
	}

	/* UnreadMark count: muted white on dark nav — never accent / full ink. */
	.nav-row :global(.unread-mark .n) {
		color: var(--white66);
		font-weight: 500;
	}

	.row-plus {
		width: 1.5rem;
		height: 1.5rem;
		display: grid;
		place-items: center;
		border: 0;
		border-radius: 999px;
		background: transparent;
		color: var(--nav-plus-fg);
		padding: 0;
		cursor: pointer;
		flex-shrink: 0;
		opacity: 0;
		pointer-events: none;
		box-sizing: border-box;
		transition:
			opacity 0.22s var(--ease),
			color 0.15s ease,
			background 0.15s ease;
	}

	.nav-row.creatable:hover .count,
	.nav-row.creatable:focus-within .count,
	.nav-row.creatable:hover :global(.unread-mark),
	.nav-row.creatable:focus-within :global(.unread-mark) {
		opacity: 0;
	}

	.nav-row.creatable:hover .row-plus,
	.nav-row.creatable:focus-within .row-plus {
		opacity: 1;
		pointer-events: auto;
	}

	.row-plus:hover {
		color: var(--white);
		background: var(--white8);
	}

	.workspace-wrap {
		flex: 1;
		min-width: 0;
		min-height: 0;
		padding: 0.75rem 0.75rem 0.75rem 0;
		box-sizing: border-box;
		display: flex;
		position: relative;
	}

	.workspace {
		flex: 1;
		min-width: 0;
		min-height: 0;
		display: flex;
		flex-direction: row;
		align-items: stretch;
		overflow: hidden;
		border-radius: 1.25rem;
		/* Follows theme page bg; comms feeds use --feed-bg in light */
		background: var(--bg);
		border: 1px solid var(--hairline);
		color: var(--white);
	}

	.shell.comms-feed .workspace {
		background: var(--feed-bg, var(--bg));
	}

	.list-col {
		position: relative;
		flex: 0 0 var(--list);
		width: var(--list);
		min-width: 280px;
		min-height: 0;
		display: flex;
		flex-direction: column;
		border-right: 1px solid var(--hairline);
		overflow: hidden;
	}

	/* Chat: feed↔activity uses φ:1 grow; morph only after a compact toggle (not on section enter). */
	.list-col.chat {
		flex-basis: var(--list-chat, var(--list));
		width: var(--list-chat, var(--list));
	}

	@media (min-width: 768px) {
		/* Feed : Notifications = φ : 1 — scales on tiny laptops and ultrawide. */
		.shell.list-chat:not(.chat-compact) .list-col.chat {
			flex: var(--phi, 1.618) 1 0;
			width: auto;
			min-width: 240px;
		}

		.shell.list-chat:not(.chat-compact) :global(.detail) {
			flex: 1 1 0;
			width: auto;
			min-width: 200px;
		}

		.shell.list-chat.chat-compact .list-col.chat {
			flex: 0 0 var(--list);
			width: var(--list);
			min-width: 280px;
		}

		.shell.list-chat.chat-compact .list-col.chat,
		.shell.list-chat.chat-was-compact:not(.chat-compact) .list-col.chat {
			transition:
				flex-grow 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				flex-basis 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				width 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				min-width 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.shell.list-chat.chat-compact :global(.detail) {
			flex: 1 1 0;
			width: auto;
			min-width: 0;
		}

		.shell.list-chat.chat-compact :global(.detail),
		.shell.list-chat.chat-was-compact:not(.chat-compact) :global(.detail) {
			transition:
				flex-grow 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				flex-basis 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				min-width 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}
	}

	.list-col.wide {
		flex: 1 1 auto;
		width: auto;
		min-width: 0;
		border-right: 0;
	}


	/* Floating header — solid control row; fade veil only as content scrolls under */
	.list-top {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		z-index: 6;
		display: flex;
		align-items: flex-start;
		justify-content: space-between;
		gap: 0.625rem;
		/* Control row ~48px; extra height is the scroll-driven fade zone */
		height: 72px;
		padding: 0 0.625rem 0 1rem;
		border-bottom: none;
		box-sizing: border-box;
		pointer-events: none;
		background: transparent;
	}

	.list-top.hidden {
		display: none;
	}

	/* Three-stop fade: solid near bar → ~80% mid → clear (no solid ::before cliff) */
	.list-top::after {
		content: '';
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		height: 72px;
		background: linear-gradient(
			to bottom,
			var(--chrome-fade) 0%,
			color-mix(in srgb, var(--chrome-fade) 80%, transparent) 50%,
			transparent 100%
		);
		opacity: var(--chrome-scroll-fade, 0);
		pointer-events: none;
		z-index: -1;
	}

	/* :global — slotted titles/actions (AppShell) share the control-row height;
	   scoped selectors alone never match cross-component class names. */
	@media (min-width: 768px) {
		.list-top > :global(.list-title-row),
		.list-top > .list-actions,
		.list-top > :global(.zoom-seg) {
			height: 3rem;
			display: flex;
			align-items: center;
		}
	}

	/* :global(*) — slotted ListTitleChrome roots are other components' scoped
	   nodes; without :global, Svelte rewrites this to >:where(.svelte-…) and
	   back/title clicks stay dead under list-top's pointer-events: none. */
	.list-top > :global(*) {
		pointer-events: auto;
	}

	.list-bottom-fade {
		display: none;
		pointer-events: none;
	}

	.list-title-row {
		display: flex;
		align-items: center;
		/* Title ↔ loader breathing room on desktop */
		gap: 0.625rem;
		min-width: 0;
	}

	.list-title {
		margin: 0;
		font-family: 'Inter', var(--font), system-ui, sans-serif;
		font-size: 0.8125rem;
		font-weight: 500;
		letter-spacing: 0;
		color: var(--white33);
	}

	.title-loader {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		align-self: center;
		flex: 0 0 auto;
		margin-left: 0.125rem;
		line-height: 0;
		overflow: visible;
		padding-top: 0.125rem;
		box-sizing: content-box;
		--loader-ink: var(--fg-66);
		color: var(--fg-66);
		opacity: 0.85;
		pointer-events: none;
	}

	.mobile-page-title {
		display: none;
	}

	/* Sit over Planning’s fixed label column */
	.list-title.planning-title {
		width: 200px;
		max-width: 200px;
		box-sizing: border-box;
		padding-right: 0.5rem;
	}

	.list-top.planning-top {
		padding-left: 1rem;
		/* Merge visually with Planning’s sticky head row */
		background: transparent;
		z-index: 8;
	}

	.list-top.planning-top::before,
	.list-top.planning-top::after {
		display: none;
	}

	/*
	 * Shared list start for Tasks / Mail / Files / Notes / … (not Chat).
	 * Desktop matches Tasks “In Progress” group header; mobile sits +4px lower.
	 */
	.shell.feed-pad {
		--feed-pad-top: 3rem;
	}

	.zoom-seg {
		display: inline-flex;
		align-items: center;
		gap: 0.125rem;
		height: 1.75rem;
		padding: 0.125rem;
		box-sizing: border-box;
		border: 1px solid transparent;
		border-radius: 0.625rem;
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
		position: relative;
		z-index: 2;
	}

	.zoom-seg button {
		height: 100%;
		padding: 0 0.625rem;
		border: 0;
		border-radius: 0.5rem;
		background: transparent;
		color: var(--fg-33);
		font: inherit;
		font-size: 0.6875rem;
		font-weight: 600;
		cursor: pointer;
		line-height: 1;
	}

	.zoom-seg button.on {
		background: var(--fg-8);
		color: var(--fg);
	}

	.list-actions {
		display: flex;
		align-items: center;
		gap: 0.375rem;
	}

	/* Desktop: chrome-desk squared; circles mobile-only — rem tracks --ui-scale */
	.chrome-btn {
		width: var(--chrome-desk, 1.75rem);
		height: var(--chrome-desk, 1.75rem);
		display: grid;
		place-items: center;
		padding: 0;
		border: 1px solid transparent;
		border-radius: var(--chrome-radius-desk, 0.625rem);
		background:
			linear-gradient(var(--glass), var(--glass)) padding-box,
			var(--glass-shine) border-box;
		color: var(--fg-66);
		cursor: pointer;
		box-sizing: border-box;
		backdrop-filter: blur(var(--glass-blur));
		-webkit-backdrop-filter: blur(var(--glass-blur));
		-webkit-tap-highlight-color: transparent;
		transition: transform 0.15s var(--ease);
	}

	.chrome-btn:hover,
	.chrome-btn.on,
	.chrome-btn[aria-expanded='true'] {
		transform: scale(1.06);
	}

	.chrome-btn:active {
		transform: scale(0.96);
	}

	.chrome-ico {
		display: block;
		width: var(--chrome-icon-desk, 1rem);
		height: var(--chrome-icon-desk, 1rem);
	}

	.pane {
		flex: 1;
		min-width: 0;
		min-height: 0;
		/* Scroll on .feed so rows pass under absolute chrome fades */
		overflow: hidden;
		padding: 0;
		display: flex;
		flex-direction: column;
	}

	.pane :global(.ptr) {
		flex: 1;
		min-height: 0;
		display: flex;
		flex-direction: column;
	}

	.pane-inner {
		flex: 1;
		min-height: 0;
		display: flex;
		flex-direction: column;
		overflow: hidden;
	}

	/* Mobile section switch: hide stale page instantly without unmounting SK outlet */
	.pane-inner.list-blank > :global(*) {
		visibility: hidden !important;
		pointer-events: none !important;
	}

	.sr-only {
		position: absolute;
		width: 1px;
		height: 1px;
		padding: 0;
		margin: -1px;
		overflow: hidden;
		clip: rect(0, 0, 0, 0);
		white-space: nowrap;
		border: 0;
	}

	@media (max-width: 767px) {
		.shell {
			--chrome-fade: var(--shell);
			--menu-progress: 0;
			/* Page peeks ~33% when drawer open; sidebar fills the rest */
			--mobile-drawer: 67vw;
			/* Page card letterbox (px — must match workspace-wrap margins) */
			--nav-open-gutter: 12px;
			/*
			 * Bottom letterbox clears the home indicator. The page card is --shell;
			 * a 12px-only gutter leaves that card painting under the indicator while
			 * theme-color already fixes the top status bar. Match sidebar bottom pad.
			 */
			--nav-open-gutter-bottom: calc(
				var(--nav-open-gutter) + env(safe-area-inset-bottom, 0px)
			);
			/*
			 * Shared chrome inset below safe-area for account row + compact chevron.
			 * Keep both bars on this token so they stay vertically aligned.
			 * Was 2.25rem (~36px×ui-scale) — sat too low under the status bar.
			 */
			--nav-open-chrome-pad: 0.625rem;
			--feed-pad-top: calc(3.25rem + env(safe-area-inset-top, 0px));
			--feed-pad-bottom: calc(1.75rem + env(safe-area-inset-bottom, 0px));
			position: relative;
			flex-direction: row;
			padding-bottom: 0;
			/* Gutters above/below the scaled page match the nav column */
			background: var(--bg);
		}

		.shell.feed-pad {
			--feed-pad-top: calc(3.75rem + env(safe-area-inset-top, 0px));
		}

		/*
		 * Drawer under the sliding page.
		 * Nav + shell gutters share --bg; the page keeps --shell (no surface swap).
		 */
		.sidebar {
			display: flex;
			position: absolute;
			left: 0;
			top: 0;
			bottom: 0;
			width: var(--mobile-drawer);
			z-index: 1;
			/*
			 * Viewport Y of account row = gutter + chrome-pad + safe-area — same as the
			 * compact chevron on the slid page (page already sits in --nav-open-gutter).
			 */
			padding: calc(
					var(--nav-open-gutter) + var(--nav-open-chrome-pad) + env(safe-area-inset-top, 0px)
				)
				0.875rem var(--nav-open-gutter-bottom);
			background: var(--bg);
			/* Glass/lift come from theme tokens — do not re-mix ink into bg here */
			pointer-events: none;
		}

		.shell.nav-open .sidebar,
		.shell.nav-dragging .sidebar {
			pointer-events: auto;
		}

		.sidebar .sidebar-inner {
			opacity: calc(0.35 + 0.65 * var(--menu-progress));
			transform: translate3d(calc(-8px * (1 - var(--menu-progress))), 0, 0);
			transition:
				opacity 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				transform 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.shell.nav-dragging .sidebar .sidebar-inner {
			transition: none;
		}

		/* Align account / search / plus with the 44px compact title chevron */
		.side-top {
			height: 2.75rem;
			padding: 0;
			gap: 0.5rem;
		}

		/* Mobile only — a bit more air between account row and nav items */
		.sidebar-inner {
			gap: 1rem;
		}

		.account-name {
			font-size: 0.875rem;
		}

		.top-search,
		.top-plus {
			width: 2.75rem;
			height: 2.75rem;
			border-radius: 9999px;
		}

		/* Nav Search glyph — fully :global so Svelte won’t strip unused .icon */
		:global(.top-search .icon),
		:global(.top-search svg) {
			width: 1.125rem !important;
			height: 1.125rem !important;
		}

		/* Match chat bubble body; scale icons/Mail pill with it (~20px radius). */
		.sidebar-nav .nav-block {
			--nav-ico: 20px;
			--nav-gap: 0.75rem;
			--nav-pad-x: 12px;
			--nav-pad-y: 0.5625rem;
			--nav-label-size: 0.9375rem;
		}

		.workspace-wrap {
			position: relative;
			z-index: 2;
			flex: 1 1 auto;
			width: 100%;
			min-width: 0;
			min-height: 0;
			padding: 0;
			display: flex;
			overflow: hidden;
			/* Same surface closed or open — never lighten/darken when slid */
			background: var(--shell);
			/* Allow horizontal menu swipe + vertical scroll; claim in JS after intent. */
			touch-action: manipulation;
			/*
			 * Slide + slight horizontal recess only. Uniform scale was double-counting
			 * with margin-block and bloating the letterbox (~30px gutters); rem’d
			 * 1.5rem height made it worse under --ui-scale. Keep gutter geometry in px.
			 */
			transform: translate3d(calc(var(--mobile-drawer) * var(--menu-progress)), 0, 0)
				scale(calc(1 - 0.045 * var(--menu-progress)), 1);
			transform-origin: left center;
			border-radius: calc(32px * var(--menu-progress));
			/*
			 * Asymmetric letterbox: top stays at --nav-open-gutter (theme-color paints
			 * the status bar). Bottom uses --nav-open-gutter-bottom so the --shell card
			 * ends above the home indicator and .shell’s --bg shows through.
			 */
			margin-top: calc(var(--nav-open-gutter) * var(--menu-progress));
			margin-bottom: calc(var(--nav-open-gutter-bottom) * var(--menu-progress));
			height: calc(
				100% -
					(
						(var(--nav-open-gutter) + var(--nav-open-gutter-bottom)) * var(--menu-progress)
					)
			);
			/* Quiet edge only — no fill/scrim that changes the page look */
			box-shadow:
				0 0 0 calc(1px * var(--menu-progress))
					color-mix(in srgb, var(--fg) calc(8% * var(--menu-progress)), transparent),
				-6px 0 22px color-mix(in srgb, var(--fg-raw, #000) calc(10% * var(--menu-progress)), transparent);
			transition:
				transform 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				border-radius 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				margin-top 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				margin-bottom 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				height 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				box-shadow 0.38s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
			will-change: transform;
		}

		.shell.nav-dragging .workspace-wrap {
			transition: none;
		}

		/*
		 * Tap the slid page body to dismiss. Starts below list chrome so the
		 * compact title pill (sibling stacking under transform) stays tappable.
		 */
		.nav-peek {
			display: block;
			position: absolute;
			top: calc(3.5rem + env(safe-area-inset-top, 0px));
			right: 0;
			bottom: 0;
			left: 0;
			z-index: 70;
			margin: 0;
			padding: 0;
			border: 0;
			border-radius: 0 0 inherit inherit;
			background: transparent;
			cursor: pointer;
			pointer-events: none;
			touch-action: manipulation;
			-webkit-tap-highlight-color: transparent;
		}

		/* Match nav-open list-top so the compact chevron stays tappable */
		.shell.nav-open .nav-peek,
		.shell.nav-dragging .nav-peek {
			top: calc(5.125rem + var(--nav-open-chrome-pad) + env(safe-area-inset-top, 0px));
		}

		.nav-peek.on {
			pointer-events: auto;
		}

		.workspace {
			border-radius: inherit;
			border: 0;
			background: var(--shell);
			width: 100%;
			overflow: hidden;
		}

		.list-col {
			flex: 1;
			width: 100%;
			max-width: none;
			min-width: 0;
			border-right: 0;
			background: var(--shell);
			transform-origin: left center;
			transform: translate3d(calc(-30% * var(--detail-progress, 0)), 0, 0)
				scale(calc(1 - 0.08 * var(--detail-progress, 0)));
			border-radius: calc(0.875rem * var(--detail-progress, 0));
			transition:
				transform 0.4s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				border-radius 0.4s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
			will-change: transform;
		}

		.list-col::before {
			content: '';
			position: absolute;
			inset: 0;
			z-index: 40;
			border-radius: inherit;
			background: color-mix(in srgb, var(--fg-raw, #000) 36%, transparent);
			opacity: var(--detail-progress, 0);
			pointer-events: none;
			transition: opacity 0.4s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		:global(.workspace.detail-dragging) .list-col,
		:global(.workspace.detail-dragging) .list-col::before {
			transition: none;
		}

		.list-top {
			height: calc(5.5rem + env(safe-area-inset-top, 0px));
			padding: calc(0.375rem + env(safe-area-inset-top, 0px)) 0.875rem 2.125rem 0.875rem;
			align-items: center;
			background: transparent;
			gap: 0.5rem;
		}

		/*
		 * Nav open: page card already sits in --nav-open-gutter, so list-top only
		 * needs --nav-open-chrome-pad (+ safe-area). Same token as sidebar pad above
		 * the gutter → account row and compact chevron stay vertically aligned.
		 */
		.shell.nav-open .list-top,
		.shell.nav-dragging .list-top {
			height: calc(5.125rem + var(--nav-open-chrome-pad) + env(safe-area-inset-top, 0px));
			padding: calc(var(--nav-open-chrome-pad) + env(safe-area-inset-top, 0px)) 0.875rem
				2.125rem 0.875rem;
		}

		.list-top::after {
			top: 0;
			height: 100%;
			background: linear-gradient(
				to bottom,
				var(--chrome-fade) 0%,
				color-mix(in srgb, var(--chrome-fade) 80%, transparent) 50%,
				transparent 100%
			);
		}

		.list-top > .list-actions {
			position: relative;
			z-index: 2;
			margin-left: auto;
			/* Never shrink round 44px chrome buttons into ovals. */
			flex-shrink: 0;
		}

		/* Section icon + title pill — opens the slide-over menu; collapses to round › when open */
		.mobile-title-pill {
			position: relative;
			z-index: 2;
			display: inline-flex;
			align-items: center;
			/* Tight chevron → icon → title; do not flex-grow the label away from the PNG. */
			gap: 0.375rem;
			max-width: min(58vw, 240px);
			height: 2.75rem;
			margin: 0;
			padding: 0 0.875rem 0 0.625rem;
			border: 1px solid transparent;
			border-radius: 9999px;
			background:
				linear-gradient(var(--glass), var(--glass)) padding-box,
				var(--glass-shine) border-box;
			color: var(--fg-66);
			cursor: pointer;
			box-sizing: border-box;
			/* Prefer ellipsis over flex-shrinking under list-actions. */
			flex-shrink: 0;
			min-width: 0;
			backdrop-filter: blur(var(--glass-blur));
			-webkit-backdrop-filter: blur(var(--glass-blur));
			-webkit-tap-highlight-color: transparent;
			transition:
				transform 0.15s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				width 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				max-width 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				padding 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				gap 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.mobile-title-pill.compact {
			width: 2.75rem;
			max-width: 2.75rem;
			min-width: 2.75rem;
			padding: 0;
			gap: 0;
			justify-content: center;
			flex-shrink: 0;
		}

		.mobile-title-pill:hover {
			transform: scale(1.02);
		}

		.mobile-title-pill:active {
			transform: scale(0.97);
		}

		/* Same 18×18 half-square chevron as ListTitleChrome / ChromeMobileBar */
		.pill-chev {
			display: grid;
			place-items: center;
			flex-shrink: 0;
			width: 1.375rem;
			height: 1.375rem;
			color: var(--fg-33);
		}

		.mobile-title-pill.compact .pill-chev {
			color: var(--fg-66);
		}

		.pill-ico {
			display: grid;
			place-items: center;
			flex-shrink: 0;
			width: 1.375rem;
			height: 1.375rem;
			/* Pull PNG a hair closer to the title (gap already 6px). */
			margin-left: 0;
			color: inherit;
		}

		.pill-ico.png {
			width: 1.375rem;
			height: 1.375rem;
		}

		.pill-ico.favicon {
			width: 1.375rem;
			height: 1.375rem;
			overflow: hidden;
			border-radius: 0.25rem;
		}

		.pill-ico.favicon.space-brand {
			overflow: visible;
			border-radius: 999px;
		}

		.pill-ico .nav-favicon {
			width: 1.375rem;
			height: 1.375rem;
		}

		.pill-title-stack {
			display: inline-flex;
			align-items: center;
			gap: 0.375rem;
			min-width: 0;
			/* Intrinsic — never flex-grow and invent space after the icon. */
			flex: 0 1 auto;
		}

		.pill-label {
			min-width: 0;
			flex: 0 1 auto;
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
			font-family: 'Inter', var(--font), system-ui, sans-serif;
			font-size: 0.9375rem;
			font-weight: 600;
			letter-spacing: -0.01em;
			line-height: 1.1;
			color: var(--fg-66);
		}

		.pill-loader {
			display: inline-flex;
			align-items: center;
			justify-content: center;
			flex: 0 0 auto;
			margin-left: 0.125rem;
			line-height: 0;
			overflow: visible;
			padding-top: 0.125rem;
			box-sizing: content-box;
			--loader-ink: var(--fg-66);
			color: var(--fg-66);
			opacity: 0.85;
			pointer-events: none;
		}

		/*
		 * Pill owns the section/folder label on mobile — hide slotted centered
		 * titles (ListTitleChrome + MailFolderTitle). CSS media (not JS class)
		 * so first paint never shows a duplicate. Restored after 268b7a6 drop.
		 */
		.list-title-slot :global(.mobile-page-title),
		.list-title-slot :global(.mail-title) {
			display: none !important;
		}

		.shell.detail-covering .mobile-title-pill {
			opacity: 0;
			visibility: hidden;
			pointer-events: none;
		}

		/* Soft veil at bottom (safe-area only — no pill). Chat owns its composer fade. */
		.list-bottom-fade {
			display: block;
			position: absolute;
			left: 0;
			right: 0;
			bottom: 0;
			z-index: 5;
			height: calc(2.5rem + env(safe-area-inset-bottom, 0px));
			background: linear-gradient(
				to top,
				color-mix(in srgb, var(--chrome-fade) 90%, transparent) 0%,
				color-mix(in srgb, var(--chrome-fade) 80%, transparent) 50%,
				transparent 100%
			);
		}

		.shell.list-chat .list-bottom-fade,
		.list-col.chat .list-bottom-fade {
			display: none;
		}

		.desktop-only-title {
			display: none;
		}

		.mobile-page-title {
			display: flex;
			align-items: center;
			align-self: center;
			justify-content: center;
			min-width: 0;
			flex: 1;
			height: 2.75rem;
			transition: transform 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		/* True screen-center: full bar width, text centered; side pad only avoids chrome overlap */
		.list-top > :global(.mobile-page-title),
		.list-top > .mobile-page-title,
		.list-top > :global(.mail-title) {
			position: absolute;
			left: 0;
			right: 0;
			top: calc(0.375rem + env(safe-area-inset-top, 0px));
			height: 2.75rem;
			flex: none;
			width: auto;
			padding: 0 58px;
			box-sizing: border-box;
			justify-content: center;
			pointer-events: none;
			z-index: 1;
		}

		.list-top > :global(.mobile-page-title) > :global(*),
		.list-top > .mobile-page-title > :global(*),
		.list-top > :global(.mail-title) > :global(*) {
			pointer-events: auto;
		}

		.shell.nav-open .list-top > :global(.mobile-page-title),
		.shell.nav-open .list-top > .mobile-page-title,
		.shell.nav-dragging .list-top > :global(.mobile-page-title),
		.shell.nav-dragging .list-top > .mobile-page-title,
		.shell.nav-open .list-top > :global(.mail-title),
		.shell.nav-dragging .list-top > :global(.mail-title) {
			position: relative;
			left: auto;
			right: auto;
			top: auto;
			flex: 1;
			width: auto;
			padding: 0;
			justify-content: flex-start;
			transform: none;
			z-index: auto;
		}

		.mobile-title-stack {
			position: relative;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			min-width: 0;
			flex: 1;
			height: 2.75rem;
			transition: align-items 0.34s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.shell.nav-open .mobile-title-stack,
		.shell.nav-dragging .mobile-title-stack,
		.shell.nav-open .list-top :global(.mobile-title-stack),
		.shell.nav-dragging .list-top :global(.mobile-title-stack) {
			align-items: flex-start;
		}

		.mobile-page-name {
			margin: 0;
			min-width: 0;
			max-width: 100%;
			font-family: 'Inter', var(--font), system-ui, sans-serif;
			/* Match deeper mobile chrome titles */
			font-size: 1.0625rem;
			font-weight: 600;
			letter-spacing: -0.01em;
			line-height: 1.15;
			color: var(--fg-66);
			text-align: center;
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
			transition:
				color 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1)),
				text-align 0.28s var(--ease, cubic-bezier(0.22, 1, 0.36, 1));
		}

		.shell.nav-open .mobile-page-name,
		.shell.nav-dragging .mobile-page-name,
		.shell.nav-open .list-top :global(.mobile-page-name),
		.shell.nav-dragging .list-top :global(.mobile-page-name),
		.shell.nav-open .list-top :global(.mobile-name),
		.shell.nav-dragging .list-top :global(.mobile-name) {
			text-align: left;
		}

		/* Slotted ListTitleChrome / MailFolderTitle share the quieter centered look */
		.list-top :global(.mobile-page-name),
		.list-top :global(.mobile-name) {
			font-size: 1.0625rem;
			font-weight: 600;
			letter-spacing: -0.01em;
			line-height: 1.15;
			color: var(--fg-66);
			text-align: center;
		}

		.list-top :global(.mobile-title-stack) {
			align-items: center;
			justify-content: center;
		}

		.shell.nav-open .list-top :global(.mobile-title-stack),
		.shell.nav-dragging .list-top :global(.mobile-title-stack) {
			align-items: flex-start;
		}

		.list-top :global(.title-btn.mobile) {
			justify-content: center;
			width: 100%;
		}

		.shell.nav-open .list-top :global(.title-btn.mobile),
		.shell.nav-dragging .list-top :global(.title-btn.mobile) {
			justify-content: flex-start;
		}

		.list-actions {
			gap: 0.5rem;
		}

		.chrome-btn {
			width: var(--chrome-mobile, 2.75rem);
			height: var(--chrome-mobile, 2.75rem);
			border-radius: 9999px;
			border: 1px solid transparent;
			background:
				linear-gradient(var(--glass), var(--glass)) padding-box,
				var(--glass-shine) border-box;
			color: var(--fg-66);
			backdrop-filter: blur(var(--glass-blur));
			-webkit-backdrop-filter: blur(var(--glass-blur));
		}

		.chrome-btn:hover,
		.chrome-btn.on,
		.chrome-btn[aria-expanded='true'] {
			transform: scale(1.04);
		}

		.chrome-ico {
			width: var(--chrome-icon, 1.375rem);
			height: var(--chrome-icon, 1.375rem);
		}

		.pane {
			padding: 0;
		}
	}

	.side-spacer {
		flex: 1;
		min-width: 0;
	}

</style>
