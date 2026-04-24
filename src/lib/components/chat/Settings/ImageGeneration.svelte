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

	const formatSize = (value: string) => (value === 'auto' ? '自动' : value.replace('x', '×'));
	const formatQuality = (value: string) =>
		({ auto: '自动', low: '低', medium: '中', high: '高' })[value] ?? '自动';
	const formatBackground = (value: string) => ({ auto: '自动', opaque: '不透明' })[value] ?? '自动';

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

		if (config.apiBaseUrl && !config.apiBaseUrl.startsWith('http')) {
			config.apiBaseUrl = '';
		}
	});
</script>

<form
	id="tab-image_generation"
	class="flex flex-col h-full justify-between text-sm"
	autocomplete="off"
	on:submit|preventDefault={saveHandler}
>
	<input class="hidden" type="text" name="username" autocomplete="username" tabindex="-1" />
	<input class="hidden" type="password" name="password" autocomplete="current-password" tabindex="-1" />

	<div class="overflow-y-scroll scrollbar-hidden h-full pr-1.5">
		<div class="font-medium mb-1">{$i18n.t('Image Generation')}</div>
		<div class="text-xs text-gray-500 dark:text-gray-400 mb-4">
			{$i18n.t('By default, images use the API URL and key from your selected direct chat connection. Set these fields only when you want a dedicated image API configuration.')}
		</div>

		<div class="space-y-3">
			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-api-base-url">
					{$i18n.t('Image API Base URL')}
				</label>
				<input
					id="image-api-base-url"
					name="image-api-base-url"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					placeholder="https://foropencode.com/v1"
					autocomplete="off"
					spellcheck="false"
					bind:value={config.apiBaseUrl}
				/>
			</div>

			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-api-key">
					{$i18n.t('Image API Key')}
				</label>
				<input
					id="image-api-key"
					name="image-api-token"
					type="password"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					placeholder={$i18n.t('Leave blank to follow the selected direct chat connection')}
					autocomplete="new-password"
					spellcheck="false"
					bind:value={config.apiKey}
				/>
			</div>

			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-model">
					{$i18n.t('Image Model')}
				</label>
				<input
					id="image-model"
					name="image-generation-model"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					autocomplete="off"
					spellcheck="false"
					bind:value={config.model}
				/>
			</div>

			<div>
				<label class="text-xs text-gray-500 dark:text-gray-400" for="image-api-version">
					{$i18n.t('API Version')}
				</label>
				<input
					id="image-api-version"
					name="image-api-version"
					class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
					placeholder={$i18n.t('Optional')}
					autocomplete="off"
					spellcheck="false"
					bind:value={config.apiVersion}
				/>
			</div>

			<div class="pt-2 border-t border-gray-100 dark:border-gray-850">
				<div class="font-medium mb-1">默认图片参数</div>
				<div class="text-xs text-gray-500 dark:text-gray-400 mb-3">
					聊天中开启生图时会默认使用这些参数，仍可在输入框的图片参数卡片里临时调整。
				</div>

				<div class="grid grid-cols-1 md:grid-cols-3 gap-3">
					<div>
						<label class="text-xs text-gray-500 dark:text-gray-400" for="image-size">
							分辨率
						</label>
						<select
							id="image-size"
							class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
							bind:value={config.size}
						>
							{#each sizes as size}
								<option value={size}>{formatSize(size)}</option>
							{/each}
						</select>
					</div>

					<div>
						<label class="text-xs text-gray-500 dark:text-gray-400" for="image-quality">
							质量
						</label>
						<select
							id="image-quality"
							class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
							bind:value={config.quality}
						>
							{#each qualities as quality}
								<option value={quality}>{formatQuality(quality)}</option>
							{/each}
						</select>
					</div>

					<div>
						<label class="text-xs text-gray-500 dark:text-gray-400" for="image-background">
							背景
						</label>
						<select
							id="image-background"
							class="w-full mt-1 rounded-lg py-2 px-3 text-sm bg-transparent border dark:border-gray-850 outline-hidden"
							bind:value={config.background}
						>
							{#each backgrounds as background}
								<option value={background}>{formatBackground(background)}</option>
							{/each}
						</select>
					</div>
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
