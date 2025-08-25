<script lang="ts">
	import type { PinyinDict } from '$lib/types';

	// Props using $props rune
	let { pinyinData, activeButton, onSelection }: {
		pinyinData: PinyinDict;
		activeButton: HTMLElement | null;
		onSelection: (pinyin: string, element: EventTarget | null) => void;
	} = $props();

	// Derived state to group pinyin by type
	let groupedPinyin = $derived(() => {
		const groups = {
			initials: [] as string[],
			'simple-finals': [] as string[],
			'compound-finals': [] as string[],
			'nasal-finals': [] as string[]
		};

		for (const [pinyin, data] of Object.entries(pinyinData)) {
			if (data.type === 'initial') {
				groups.initials.push(pinyin);
			} else if (data.type === 'simple-final') {
				groups['simple-finals'].push(pinyin);
			} else if (data.type === 'compound-final') {
				groups['compound-finals'].push(pinyin);
			} else if (data.type === 'nasal-final') {
				groups['nasal-finals'].push(pinyin);
			}
		}

		return groups;
	});

	function handleButtonClick(pinyin: string, event: MouseEvent) {
		onSelection(pinyin, event.target);
	}
</script>

<div class="bg-white p-6 rounded-2xl shadow-lg">
	<!-- Iniciales -->
	<div>
		<h2 class="text-xl font-semibold mb-4 border-b pb-2 text-indigo-700">Iniciales (声母)</h2>
		<div class="grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 gap-3">
			{#each groupedPinyin().initials as pinyin}
				<button
					class="pinyin-btn bg-white border border-gray-300 rounded-lg py-2 text-lg font-medium text-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
					class:active={activeButton?.dataset?.pinyin === pinyin}
					data-pinyin={pinyin}
					onclick={(event) => handleButtonClick(pinyin, event)}
				>
					{pinyin}
				</button>
			{/each}
		</div>
	</div>

	<!-- Finales Simples -->
	<div class="mt-8">
		<h2 class="text-xl font-semibold mb-4 border-b pb-2 text-indigo-700">Finales Simples (单韵母)</h2>
		<div class="grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 gap-3">
			{#each groupedPinyin()['simple-finals'] as pinyin}
				<button
					class="pinyin-btn bg-white border border-gray-300 rounded-lg py-2 text-lg font-medium text-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
					class:active={activeButton?.dataset?.pinyin === pinyin}
					data-pinyin={pinyin}
					onclick={(event) => handleButtonClick(pinyin, event)}
				>
					{pinyin}
				</button>
			{/each}
		</div>
	</div>

	<!-- Finales Compuestas -->
	<div class="mt-8">
		<h2 class="text-xl font-semibold mb-4 border-b pb-2 text-indigo-700">Finales Compuestas (复韵母)</h2>
		<div class="grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 gap-3">
			{#each groupedPinyin()['compound-finals'] as pinyin}
				<button
					class="pinyin-btn bg-white border border-gray-300 rounded-lg py-2 text-lg font-medium text-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
					class:active={activeButton?.dataset?.pinyin === pinyin}
					data-pinyin={pinyin}
					onclick={(event) => handleButtonClick(pinyin, event)}
				>
					{pinyin}
				</button>
			{/each}
		</div>
	</div>

	<!-- Finales Nasales -->
	<div class="mt-8">
		<h2 class="text-xl font-semibold mb-4 border-b pb-2 text-indigo-700">Finales Nasales (鼻韵母)</h2>
		<div class="grid grid-cols-4 sm:grid-cols-5 md:grid-cols-6 gap-3">
			{#each groupedPinyin()['nasal-finals'] as pinyin}
				<button
					class="pinyin-btn bg-white border border-gray-300 rounded-lg py-2 text-lg font-medium text-gray-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
					class:active={activeButton?.dataset?.pinyin === pinyin}
					data-pinyin={pinyin}
					onclick={(event) => handleButtonClick(pinyin, event)}
				>
					{pinyin}
				</button>
			{/each}
		</div>
	</div>
</div>