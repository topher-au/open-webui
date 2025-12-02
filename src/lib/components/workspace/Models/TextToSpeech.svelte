<script lang="ts">
	import Textarea from '$lib/components/common/Textarea.svelte';
	import { getContext, onMount } from 'svelte';
	import { config } from '$lib/stores';
	import { toast } from 'svelte-sonner';

	import { getModels as _getModels, getVoices as _getVoices } from '$lib/apis/audio';

	const i18n = getContext('i18n');

	export let onChange: (params: any) => void = () => {
	};

	let loading = false;

	const defaultTts: {
		model: string | null;
		voice: string | null;
		params: string | null
	} = {
		model: null,
		voice: null,
		params: null
	};

	export let tts = defaultTts;

	$: if (tts) {
		onChange(tts);
	}

	let engine = '';
	let voices: { id: string; name: string }[] = [];
	let models: { id: string; name: string }[] = [];

	onMount(async () => {
			engine = $config?.audio?.tts?.engine ?? '';

			if (tts?.model?.trim() === '')
				tts.model = null;

			if (tts?.voice?.trim() === '')
				tts.voice = null;

			if ((tts?.params ?? null) !== null) {
				try {
					let params = tts.params ? JSON.parse(tts?.params?.trim()) : null;
					if (!params || (params && Object.keys(params).length === 0)) {
						tts.params = null;
					} else {
						tts.params = JSON.stringify(params, null, 2);
					}
				} catch (e) {
					{
						toast.error(`${e}`);
						tts.params = null;
					}
				}
			}
		}
	);

	const getVoices = async () => {
		if (engine === 'browser-kokoro') {

		} else {
			if (engine === '') {
				// TODO:
			} else {
				const res = await _getVoices(localStorage.token).catch((e) => {
					toast.error(`${e}`);
				});

				if (res) {
					console.log(res);
					voices = res.voices;
				}
			}
		}
	};

	const getModels = async () => {
		if (engine === '') {
			models = [];
		} else {
			const res = await _getModels(
				localStorage.token
			).catch((e) => {
				toast.error(`${e}`);
			});

			if (res) {
				console.log(res);
				models = res.models;
			}
		}
	};
</script>

{#if (!loading)}
	<div class=" space-y-1 text-xs pb-safe-bottom">
		<div>
			<div class="py-0.5 flex w-full justify-between">
				<div class=" self-center text-xs font-medium">
					{$i18n.t('Model')}
				</div>
				<button
					class="p-1 px-3 text-xs flex rounded-sm transition"
					type="button"
					on:click={async () => {
															tts.model = (tts.model ?? null) === null ? '' : null;
															if ((tts.model ?? null) !== null) {
																await getModels();
															}
														}}
				>
					{#if (tts.model ?? null) === null}
						<span class="ml-2 self-center"> {$i18n.t('Default')}</span>
					{:else}
						<span class="ml-2 self-center"> {$i18n.t('Custom')}</span>
					{/if}
				</button>
			</div>

			{#if (tts.model ?? null) !== null}
				<div class="flex mt-0.5 space-x-2">
					<div class=" flex-1">
						<input
							list="model-list"
							class="w-full text-sm bg-transparent dark:text-gray-300 outline-hidden"
							bind:value={tts.model}
							placeholder={$i18n.t('Select a model')}
						/>
						<datalist id="model-list">
							{#each models as model}
								<option value={model.id}>{model.name}</option>
							{/each}
						</datalist>
					</div>
				</div>
			{/if}
		</div>
		<div>
			<div class="py-0.5 flex w-full justify-between">
				<div class=" self-center text-xs font-medium">
					{$i18n.t('Voice')}
				</div>
				<button
					class="p-1 px-3 text-xs flex rounded-sm transition"
					type="button"
					on:click={async () => {
						tts.voice = (tts.voice ?? null) === null ? '' : null;
						if ((tts.voice ?? null) !== null) {
							await getVoices();
						}
					}}
				>
					{#if (tts.voice ?? null) === null}
						<span class="ml-2 self-center"> {$i18n.t('Default')}</span>
					{:else}
						<span class="ml-2 self-center"> {$i18n.t('Custom')}</span>
					{/if}
				</button>
			</div>

			{#if (tts.voice ?? null) !== null}
				<div class="flex mt-0.5 space-x-2">
					<div class=" flex-1">
						<input
							list="voice-list"
							class="w-full text-sm bg-transparent dark:text-gray-300 outline-hidden"
							bind:value={tts.voice}
							placeholder={$i18n.t('Select a voice')}
						/>
						<datalist id="voice-list">
							{#each voices as voice}
								<option value={voice.id}>{voice.name}</option>
							{/each}
						</datalist>
					</div>
				</div>
			{/if}
		</div>

		{#if engine === 'openai'}
			<div>
				<div class="py-0.5 flex w-full justify-between">
					<div class=" self-center text-xs font-medium">
						{$i18n.t('Additional Parameters')}
					</div>
					<button
						class="p-1 px-3 text-xs flex rounded-sm transition"
						type="button"
						on:click={async () => {
															tts.params = (tts.params ?? null) === null ? '' : null;
														}}
					>
						{#if (tts.params ?? null) === null}
							<span class="ml-2 self-center"> {$i18n.t('Default')}</span>
						{:else}
							<span class="ml-2 self-center"> {$i18n.t('Custom')}</span>
						{/if}
					</button>
				</div>

				{#if (tts.params ?? null) !== null}
					<div class="mt-2 mb-1 text-xs text-gray-400 dark:text-gray-500">
						{$i18n.t(`Enter additional parameters in JSON format.`)}
					</div>
					<div class="flex w-full">
						<div class=" flex-1">
							<Textarea
								className="w-full rounded-lg py-2 px-4 text-sm bg-gray-50 dark:text-gray-300 dark:bg-gray-850 outline-hidden"
								placeholder={$i18n.t('Enter additional parameters in JSON format')}
								minSize={100}
								bind:value={tts.params}
							/>
						</div>
					</div>
				{/if}
			</div>
		{/if}
	</div>
{/if}