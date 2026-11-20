<script lang="ts">
	import Scene from '#lib/Scene.svelte';

	// Placeholder contact. Replace with the real address or booking link.
	const CONTACT = 'mailto:hello@trainmindset.com?subject=First%20session';

	let progress = $state(0);
	let holding = $state(false);
	let held = $state(0);
	let phase = $state('');

	function onScroll() {
		const max = document.documentElement.scrollHeight - innerHeight;
		progress = max > 0 ? Math.min(1, Math.max(0, scrollY / max)) : 0;
	}

	function reveal(node: HTMLElement) {
		const io = new IntersectionObserver(
			([e]) => {
				if (e.isIntersecting) {
					node.classList.add('in');
					io.disconnect();
				}
			},
			{ threshold: 0.2 }
		);
		io.observe(node);
		return { destroy: () => io.disconnect() };
	}

	const steps = [
		{ n: '01', name: 'Notice', body: 'Catch the spike early. Name it in one word. Naming it shrinks it.' },
		{ n: '02', name: 'Settle', body: 'Four seconds in. Six seconds out. Slow the body and the mind follows.' },
		{ n: '03', name: 'Choose', body: 'Pick the next action. Small, clear, yours. Then do only that.' }
	];

	const format = [
		['Sessions', '1:1, 50 minutes'],
		['Practice', 'Six minutes a day'],
		['Between sessions', 'Message support']
	];
</script>

<svelte:head>
	<title>trainmindset. Quiet is a skill.</title>
	<meta
		name="description"
		content="Mindset coaching for people who perform under pressure. Learn to settle the mind on demand."
	/>
</svelte:head>

<svelte:window onscroll={onScroll} />

<Scene order={progress} bind:holding bind:held bind:phase />

<header
	class="fixed inset-x-0 top-0 z-20 flex items-center justify-between px-6 py-5 text-[13px] md:px-10"
>
	<a href="#top" class="font-display text-2xl tracking-tight">trainmindset</a>
	<a
		href="#book"
		class="border-b border-bone/40 pb-0.5 transition-colors hover:border-ember hover:text-ember"
		>book a session</a
	>
</header>

<main id="top" class="relative z-10">
	<!-- hero -->
	<section class="flex min-h-svh flex-col justify-end px-6 pb-10 select-none md:px-10">
		<h1
			class="font-display text-[clamp(3.6rem,12.5vw,11.5rem)] leading-[0.88] tracking-[-0.03em]"
		>
			Quiet is<br />a skill.<br />
			<em class="text-bone/90">Train it<span class="text-ember">.</span></em>
		</h1>
		<div
			class="mt-10 flex flex-col gap-6 border-t border-line pt-5 text-[13px] text-dim md:flex-row md:items-end md:justify-between"
		>
			<p class="max-w-sm leading-relaxed text-bone/80">
				Mindset coaching for athletes, founders and anyone who performs under pressure.
			</p>
			<p class="min-h-5 tabular-nums" aria-live="off">
				{#if holding}
					<span class="text-ember">breathe {phase === 'in' ? 'in' : 'out'}</span>
					<span class="ml-3">{String(held).padStart(2, '0')}s</span>
				{:else}
					hold anywhere to settle the water
				{/if}
			</p>
		</div>
	</section>

	<!-- noise -->
	<section class="flex min-h-svh items-center px-6 py-28 md:px-10">
		<div use:reveal class="reveal max-w-3xl">
			<p class="mb-8 text-[13px] text-dim">the problem</p>
			<p class="font-display text-[clamp(2rem,5vw,4.4rem)] leading-[1.04] tracking-[-0.015em]">
				Under pressure the mind gets loud. Doubt. Replays. Forecasts of everything that can go
				wrong.
				<span class="text-dim">You do not need less pressure. You need a steadier mind inside it.</span>
			</p>
		</div>
	</section>

	<!-- method -->
	<section class="px-6 py-28 md:px-10">
		<div use:reveal class="reveal mb-14 flex items-baseline justify-between text-[13px] text-dim">
			<p>the method</p>
			<p>three moves</p>
		</div>
		<ul class="max-w-5xl">
			{#each steps as s}
				<li
					use:reveal
					class="reveal group grid grid-cols-[3rem_1fr] gap-x-4 gap-y-3 border-t border-line py-9 last:border-b md:grid-cols-[6rem_1fr_22rem] md:items-baseline"
				>
					<span class="pt-2 text-[13px] text-dim">{s.n}</span>
					<h2
						class="font-display text-[clamp(2.6rem,7vw,6rem)] leading-none tracking-[-0.02em] transition-transform duration-500 group-hover:translate-x-3"
					>
						{s.name}<span
							class="text-ember opacity-0 transition-opacity duration-500 group-hover:opacity-100"
							>.</span
						>
					</h2>
					<p class="col-start-2 text-[13px] leading-relaxed text-bone/75 md:col-start-3">
						{s.body}
					</p>
				</li>
			{/each}
		</ul>
	</section>

	<!-- format -->
	<section class="px-6 py-28 md:px-10">
		<div use:reveal class="reveal grid max-w-5xl gap-14 md:grid-cols-[1fr_1fr]">
			<h2 class="font-display text-[clamp(2.2rem,5vw,4.2rem)] leading-[1] tracking-[-0.02em]">
				Built for people<br />who already train.
			</h2>
			<dl class="text-[13px]">
				{#each format as [k, v]}
					<div class="flex justify-between border-t border-line py-4 last:border-b">
						<dt class="text-dim">{k}</dt>
						<dd>{v}</dd>
					</div>
				{/each}
			</dl>
		</div>
	</section>

	<!-- book -->
	<section id="book" class="flex min-h-svh flex-col justify-end px-6 pb-10 md:px-10">
		<div use:reveal class="reveal">
			<h2
				class="font-display text-[clamp(3rem,10vw,9rem)] leading-[0.9] tracking-[-0.03em]"
			>
				Start with<br />one session<span class="text-ember">.</span>
			</h2>
			<a
				href={CONTACT}
				class="mt-12 inline-flex items-center gap-4 border border-bone/50 px-7 py-4 text-[13px] transition-colors duration-300 hover:border-ember hover:bg-ember hover:text-ink"
			>
				book a session <span aria-hidden="true">→</span>
			</a>
		</div>
		<footer class="mt-24 flex justify-between border-t border-line pt-5 text-[13px] text-dim">
			<span>trainmindset</span>
			<span>{new Date().getFullYear()}</span>
		</footer>
	</section>
</main>
