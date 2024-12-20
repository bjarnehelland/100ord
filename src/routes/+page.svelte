<script lang="ts">
	export const ssr = false;

	let allWords = [
		'og',
		'i',
		'det',
		'på',
		'som',
		'er',
		'en',
		'til',
		'å',
		'han',
		'av',
		'for',
		'med',
		'at',
		'var',
		'de',
		'ikke',
		'den',
		'har',
		'jeg',
		'om',
		'et',
		'men',
		'så',
		'seg',
		'hun',
		'hadde',
		'fra',
		'vi',
		'du',
		'kan',
		'da',
		'ble',
		'ut',
		'skal',
		'vil',
		'ham',
		'etter',
		'over',
		'ved',
		'også',
		'bare',
		'eller',
		'sa',
		'nå',
		'dette',
		'noe',
		'være',
		'meg',
		'mot',
		'opp',
		'der',
		'når',
		'inn',
		'dem',
		'kunne',
		'andre',
		'blir',
		'alle',
		'noen',
		'sin',
		'ha',
		'år',
		'henne',
		'må',
		'selv',
		'sier',
		'få',
		'kom',
		'denne',
		'enn',
		'to',
		'hans',
		'bli',
		'ville',
		'før',
		'vært',
		'skulle',
		'går',
		'her',
		'slik',
		'gikk',
		'mer',
		'hva',
		'igjen',
		'fikk',
		'man',
		'alt',
		'mange',
		'ingen',
		'får',
		'oss',
		'hvor',
		'under',
		'siden',
		'hele',
		'dag',
		'gang',
		'samme',
		'ned'
	];

	let selectedRanges = $state<number[]>([]);
	let words = $derived(selectedRanges.flatMap((i) => allWords.slice(i * 10, (i + 1) * 10)));

	let correctWords: string[] = $state([]);
	let avaliableWords = $derived(words.filter((w) => !correctWords.includes(w)));
	let word = $state('');

	let isStarted = $state(false);
	let startTime = $state<number | null>(null);
	let completionTime = $state<number | null>(null);

	let correctWordsCount = $derived(correctWords.length);
	let avaliableWordsCount = $derived(avaliableWords.length);

	let norwegianVoice: SpeechSynthesisVoice | null = null;
	if (typeof window !== 'undefined') {
		let voices = speechSynthesis.getVoices();
		norwegianVoice = voices.find((voice) => voice.lang === 'nb-NO') ?? null;
		speechSynthesis.addEventListener('voiceschanged', (ev) => {
			voices = speechSynthesis.getVoices();
			norwegianVoice = voices.find((voice) => voice.lang === 'nb-NO') ?? null;
			console.log(norwegianVoice);
		});
	}

	function toggleRange(rangeIndex: number) {
		if (selectedRanges.includes(rangeIndex)) {
			selectedRanges = selectedRanges.filter((i) => i !== rangeIndex);
		} else {
			selectedRanges = [...selectedRanges, rangeIndex];
		}
	}

	function startGame() {
		isStarted = true;
		startTime = Date.now();
		word = avaliableWords[Math.floor(Math.random() * avaliableWords.length)];
	}

	function nextWord() {
		let newWord;
		do {
			newWord = avaliableWords[Math.floor(Math.random() * avaliableWords.length)];
		} while (newWord === word && avaliableWords.length > 1);
		word = newWord;
	}

	function correctWord() {
		correctWords.push(word);
		if (avaliableWordsCount <= 0) {
			completionTime = Date.now() - startTime!;
		}
		nextWord();
	}

	function formatTime(ms: number): string {
		const seconds = Math.floor(ms / 1000);
		const minutes = Math.floor(seconds / 60);
		const remainingSeconds = seconds % 60;
		return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`;
	}

	function resetGame() {
		correctWords = [];
		isStarted = false;
		completionTime = null;
		startTime = null;
	}

	function speak() {
		if (!norwegianVoice) {
			console.error('No Norwegian voice found');
			return;
		}

		const utterance = new SpeechSynthesisUtterance(word);
		utterance.voice = norwegianVoice;
		speechSynthesis.speak(utterance);
	}
</script>

<div class="flex h-dvh flex-col items-center justify-between bg-gray-100">
	<div class="flex flex-col items-center">
		<p>Riktig: {correctWordsCount} / {words.length}</p>
	</div>

	<div class="flex flex-grow items-center justify-center">
		{#if !isStarted}
			<div class="flex flex-col items-center gap-3">
				<p class="text-lg font-bold">Velg ordområder:</p>
				<div class="grid gap-2">
					{#each Array(Math.floor(allWords.length / 10)) as _, i}
						<label
							class="flex cursor-pointer items-center gap-2 rounded-lg border border-gray-200 bg-white p-2 shadow-sm hover:bg-gray-50"
						>
							<div class="relative flex h-5 w-5 items-center justify-center">
								<input
									type="checkbox"
									name="wordRange"
									value={i}
									checked={selectedRanges.includes(i)}
									onchange={() => toggleRange(i)}
									class="peer h-5 w-5 appearance-none rounded border border-gray-300 checked:border-purple-500 checked:bg-purple-500"
								/>
								<svg
									class="pointer-events-none absolute hidden h-3 w-3 text-white peer-checked:block"
									xmlns="http://www.w3.org/2000/svg"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="4"
									stroke-linecap="round"
									stroke-linejoin="round"
								>
									<polyline points="20 6 9 17 4 12" />
								</svg>
							</div>
							<span class="text-sm">Ord {i * 10 + 1}-{(i + 1) * 10}</span>
						</label>
					{/each}
				</div>
				<button
					class="rounded-lg bg-purple-500 px-6 py-3 text-xl text-white hover:bg-purple-600"
					onclick={startGame}
				>
					Start
				</button>
			</div>
		{:else if completionTime}
			<div class="flex flex-col items-center gap-4">
				<p class="text-lg font-bold">Fullført tid:</p>
				<p class="text-5xl font-bold text-green-600">{formatTime(completionTime)}</p>
				<button
					class="rounded-lg bg-purple-500 px-6 py-3 text-xl text-white hover:bg-purple-600"
					onclick={resetGame}
				>
					Start på nytt
				</button>
			</div>
		{:else}
			<div class="flex flex-col items-center gap-4">
				<p class="text-5xl">{word}</p>
				{#if norwegianVoice}
					<button
						class="touch-manipulation rounded-lg bg-orange-500 px-4 py-2 text-white hover:bg-orange-600"
						onclick={speak}
						aria-label="Speak word"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-6 w-6"
							fill="none"
							viewBox="0 0 24 24"
							stroke="currentColor"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"
							/>
						</svg>
					</button>
				{/if}
			</div>
		{/if}
	</div>

	{#if isStarted && !completionTime}
		<div class="p-8">
			<button
				class="m-2 touch-manipulation rounded-lg bg-blue-500 px-6 py-3 text-xl text-white hover:bg-blue-600"
				onclick={nextWord}
			>
				Neste
			</button>
			<button
				class="m-2 touch-manipulation rounded-lg bg-green-500 px-6 py-3 text-xl text-white hover:bg-green-600"
				onclick={correctWord}
			>
				Korrekt
			</button>
		</div>
	{/if}
</div>
