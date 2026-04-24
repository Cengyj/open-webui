<script lang="ts">
	import { getContext, onMount } from 'svelte';
	import { settings } from '$lib/stores';

	export let saveSettings: Function;

	const i18n = getContext('i18n');

	let config = {
		apiBaseUrl: '',
		apiKey: '',
		apiVersion: '',
		model: 'gpt-image-2',
		size: 'auto',
		quality: 'auto',
		background: 'auto'
	};

	const sizes = ['auto', '1024x1024', '1536x1024', '1024x1536', '2048x2048', '3840x2160', '2160x3840'];
	const qualities = ['auto', 'low', 'medium', 'high'];
	const backgrounds = ['auto', 'opaque'];

	const saveHandler = async () => {
		config.apiBaseUrl = config.apiBaseUrl.replace(/\/$/, '').trim();
		config.apiKey = config.apiKey.trim();
		config.apiVersion = config.apiVersion.trim();
		config.model = config.model.trim() || 'gpt-image-2';

		await saveSettings({
			imageGeneration: config
		});
	};

	onMount(() => {
		config = {
			...config,
			...($settings?.imageGeneration ?? {})
		};
	});
</script>

<form
	id="tab-image_generation"
	class="flex flex-col h-full justify-between text-sm"
	on:submit|preventDefault={saveHandler}
>
	<div class="overflow-y-scroll scrollbar-hidden h-full pr-1.5">
		<div class="font-medium mb-1">{$i18n.t('Image Generation')}</div>
		<div class="text-xs text-gray-500 dark:text-gray-400 mb-4">
			{$i18n.t('By default, images use the API URL and key from your selected direct chat connection. Set the fields below only when you want a dedicated image API configuration.')}
		</div>

		<div class="space-y-3">
			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-api-base-url">
					{$i18n.t('Image API Base URL')}
				</label>
				<input
					id="image-api-base-url"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					placeholder="https://api.openai.com/v1"
					bind:value={config.apiBaseUrl}
				/>
			</div>

			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-api-key">
					{$i18n.t('Image API Key')}
				</label>
				<input
					id="image-api-key"
					type="password"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					placeholder={$i18n.t('Leave blank to follow the selected direct chat connection')}
					bind:value={config.apiKey}
				/>
			</div>

			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-model">
					{$i18n.t('Image Model')}
				</label>
				<input
					id="image-model"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					bind:value={config.model}
				/>
			</div>

			<div class="grid grid-cols-1 md:grid-cols-3 gap-3">
				<div>
					<label class="text-xs text-gray-500 dark:text-gray-400" for="image-size">
						{$i18n.t('Size')}
					</label>
					<select
						id="image-size"
						class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
						bind:value={config.size}
					>
						{#each sizes as size}
							<option value={size}>{size}</option>
						{/each}
					</select>
				</div>

				<div>
					<label class="text-xs text-gray-500 dark:text-gray-400" for="image-quality">
						{$i18n.t('Quality')}
					</label>
					<select
						id="image-quality"
						class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
						bind:value={config.quality}
					>
						{#each qualities as quality}
							<option value={quality}>{quality}</option>
						{/each}
					</select>
				</div>

				<div>
					<label class="text-xs text-gray-500 dark:text-gray-400" for="image-background">
						{$i18n.t('Background')}
					</label>
					<select
						id="image-background"
						class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
						bind:value={config.background}
					>
						{#each backgrounds as background}
							<option value={background}>{background}</option>
						{/each}
					</select>
				</div>
			</div>
		</div>
	</div>

	<div class="flex justify-end pt-3 text-sm font-medium">
		<button
			class="px-3.5 py-1.5 text-sm font-medium bg-black hover:bg-gray-900 text-white dark:bg-white dark:text-black dark:hover:bg-gray-100 transition rounded-full"
			type="submit"
		>
			{$i18n.t('Save')}
		</button>
	</div>
</form>
