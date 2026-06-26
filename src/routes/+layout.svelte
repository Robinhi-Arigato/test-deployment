<script lang="ts">
	import { Menu, Folder, Settings } from '@lucide/svelte';
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
	let isNavOpen = $state(true);
</script>

<svelte:head><link rel="icon" href={favicon} /></svelte:head>
<button
	aria-label="hamburger-menu"
	class="fixed top-4 left-4 flex h-12 w-12 cursor-pointer items-center justify-center rounded-xl bg-slate-100"
	onclick={() => (isNavOpen = !isNavOpen)}><Menu size={24} /></button
>
<div
	class={[
		'fixed top-18 left-4 z-10 flex h-120 w-50 flex-col items-start rounded-xl border border-slate-200 bg-white pt-2 shadow-sm',
		isNavOpen ? 'opacity-100' : 'opacity-0'
	]}
>
	<div
		class="flex w-full cursor-pointer items-center gap-2 px-4 py-2 font-Fredoka text-sm tracking-wide hover:bg-slate-100"
	>
		<Folder size={18} />Open
	</div>
	<div
		class="flex w-full cursor-pointer items-center gap-2 px-4 py-2 font-Fredoka text-sm tracking-wide hover:bg-slate-100"
	>
		<Settings size={18} />Settings
	</div>
</div>
{@render children()}
