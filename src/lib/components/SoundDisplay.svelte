<script lang="ts">
	import { onMount } from 'svelte';
	import type { PinyinDict } from '$lib/types';

	// Props using $props rune
	let { currentSound, pinyinData }: {
		currentSound: string | null;
		pinyinData: PinyinDict;
	} = $props();

	// Local state using $state runes
	let characterTarget: HTMLDivElement | null = $state(null);
	let writer: any = $state(null);
	let synth: any = $state(null);
	let audioLoop: any = $state(null);
	let isPlaying = $state(false);

	// Derived values using $derived rune
	let currentData = $derived(() => {
		return currentSound ? pinyinData[currentSound] : null;
	});

	let showPlaceholder = $derived(() => !currentSound);

	// Initialize Tone.js synthesizer
	onMount(async () => {
		if (typeof window !== 'undefined' && (window as any).Tone) {
			synth = new (window as any).Tone.Synth().toDestination();
		}
	});

	// Effect to handle character animation when sound changes
	$effect(() => {
		const data = currentData();
		if (data && characterTarget && typeof window !== 'undefined' && (window as any).HanziWriter) {
			createCharacterAnimation(data.char);
		}
	});

	// Effect to stop audio when component unmounts or sound changes
	$effect(() => {
		return () => {
			stopSound();
		};
	});

	function createCharacterAnimation(char: string) {
		if (!characterTarget) return;
		
		characterTarget.innerHTML = '';
		
		try {
			writer = (window as any).HanziWriter.create(characterTarget, char, {
				width: 224,
				height: 224,
				padding: 20,
				showOutline: true,
				strokeAnimationSpeed: 1,
				delayBetweenStrokes: 250,
				strokeColor: '#4f46e5',
				outlineColor: '#d1d5db'
			});
			
			writer.animateCharacter();
		} catch (error) {
			console.error('Error creating character animation:', error);
		}
	}

	function handleCharacterClick() {
		if (writer) {
			writer.animateCharacter();
		}
	}

	async function toggleSound() {
		const data = currentData();
		if (!synth || !data) return;

		try {
			// Tone.js requires user gesture to start audio context
			await (window as any).Tone.start();

			if (isPlaying) {
				stopSound();
			} else {
				playSound();
			}
		} catch (error) {
			console.error('Error with audio:', error);
		}
	}

	function playSound() {
		const data = currentData();
		if (!synth || !data) return;

		try {
			// Create a loop for continuous playback
			audioLoop = new (window as any).Tone.Loop((time: number) => {
				synth.triggerAttackRelease(data.pitch, "8n", time);
			}, "4n").start(0);

			(window as any).Tone.Transport.start();
			isPlaying = true;
		} catch (error) {
			console.error('Error playing sound:', error);
		}
	}

	function stopSound() {
		if (audioLoop) {
			try {
				audioLoop.stop(0);
				audioLoop.dispose();
				audioLoop = null;
			} catch (error) {
				console.error('Error stopping audio loop:', error);
			}
		}

		if (typeof window !== 'undefined' && (window as any).Tone) {
			try {
				(window as any).Tone.Transport.stop();
			} catch (error) {
				console.error('Error stopping transport:', error);
			}
		}

		isPlaying = false;
	}
</script>

<div class="bg-white p-6 rounded-2xl shadow-lg">
	<div class="sticky top-8">
		{#if showPlaceholder()}
			<!-- Placeholder -->
			<div class="text-center text-gray-500 flex flex-col items-center justify-center h-full min-h-[60vh]">
				<svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mb-4 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
					<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.5 14h-.79l-.28-.27A6.5 6.5 0 0016 9.5 6.5 6.5 0 109.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z" />
				</svg>
				<p class="text-lg">Selecciona un sonido pinyin para comenzar.</p>
				<p class="text-sm mt-2">Haz clic en un botón de la izquierda para ver los detalles y escuchar el sonido.</p>
			</div>
		{:else}
			<!-- Sound Details -->
			<div class="grid grid-cols-1 md:grid-cols-2 gap-6 items-center">
				<!-- Articulación y Controles -->
				<div class="flex flex-col items-center">
					<h3 class="text-2xl font-bold text-indigo-700 mb-4">{currentData()?.note}</h3>
					<div class="w-full max-w-xs bg-gray-50 rounded-lg p-4 border">
						<img 
							src="https://placehold.co/400x300/e2e8f0/4a5568?text=Articulación+de+'{currentSound}'" 
							alt="Diagrama de articulación para {currentSound}" 
							class="w-full h-auto object-contain rounded-md"
						>
					</div>
					<p class="text-center text-sm text-gray-500 mt-2">Posición de la boca y la lengua</p>
					<button 
						onclick={toggleSound}
						class="mt-6 w-full max-w-xs bg-indigo-600 text-white font-bold py-3 px-4 rounded-lg shadow-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-transform transform hover:scale-105"
					>
						{isPlaying ? `Detener Sonido (${currentSound})` : `Reproducir Sonido (${currentSound})`}
					</button>
				</div>
				
				<!-- Carácter y Animación -->
				<div class="flex flex-col items-center">
					<h3 class="text-xl font-semibold text-gray-800 mb-4">Carácter de Ejemplo</h3>
					<div 
						bind:this={characterTarget}
						onclick={handleCharacterClick}
						onkeydown={(e) => {
							if (e.key === 'Enter' || e.key === ' ') {
								e.preventDefault();
								handleCharacterClick();
							}
						}}
						tabindex="0"
						role="button"
						aria-label="Reproducir animación de escritura del carácter"
						class="w-48 h-48 md:w-56 md:h-56 bg-gray-50 rounded-lg border cursor-pointer focus:outline-none focus:ring-2 focus:ring-indigo-500" 
						title="Haz clic para animar de nuevo"
					></div>
					<p class="text-center text-sm text-gray-500 mt-2">Animación de escritura</p>
				</div>
			</div>
		{/if}
	</div>
</div>