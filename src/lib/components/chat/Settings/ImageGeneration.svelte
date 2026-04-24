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
	let customSize = '';
	let sizeError = '';

	const formatSize = (value: string) => (value === 'auto' ? '自动' : value.replace('x', '×'));
	const formatQuality = (value: string) =>
		({ auto: '自动', low: '低', medium: '中', high: '高' })[value] ?? '自动';
	const formatBackground = (value: string) => ({ auto: '自动', opaque: '不透明' })[value] ?? '自动';
	const normalizeSize = (value: string) => value.trim().toLowerCase().replace('×', 'x');
	const validateSize = (value: string) => {
		const size = normalizeSize(value);
		if (!size || size === 'auto') return { valid: true, value: 'auto', message: '' };

		const match = size.match(/^(\d+)x(\d+)$/);
		if (!match) return { valid: false, value: 'auto', message: '请输入 宽x高，例如 3840x2160' };

		const width = Number(match[1]);
		const height = Number(match[2]);
		if (width <= 0 || height <= 0) return { valid: false, value: 'auto', message: '宽高必须大于 0' };
		if (Math.max(width, height) > 3840) return { valid: false, value: 'auto', message: '最大边不能超过 3840' };
		if (width % 16 !== 0 || height % 16 !== 0) {
			return { valid: false, value: 'auto', message: '宽高都必须是 16 的倍数' };
		}
		if (Math.max(width, height) / Math.min(width, height) > 3) {
			return { valid: false, value: 'auto', message: '长宽比不能超过 3:1' };
		}

		return { valid: true, value: `${width}x${height}`, message: '' };
	};
	const applySize = (value: string) => {
		const result = validateSize(value);
		config.size = result.value;
		sizeError = result.message;
		customSize = result.valid && result.value !== 'auto' ? result.value : '';
	};

	const saveHandler = async () => {
		if (customSize.trim()) {
			applySize(customSize);
		}

		config.apiBaseUrl = config.apiBaseUrl.replace(/\/$/, '').trim();
		config.apiKey = config.apiKey.trim();
		config.apiVersion = config.apiVersion.trim();
		config.model = config.model.trim() || 'gpt-image-2';
		const sizeResult = validateSize(config.size);
		config.size = sizeResult.value;
		sizeError = sizeResult.message;

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

		if (config.size && !sizes.includes(config.size)) {
			customSize = config.size;
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

				<div class="grid grid-cols-1 gap-3">
					<div class="rounded-2xl border border-gray-100 dark:border-gray-850 bg-gray-50/70 dark:bg-gray-950/30 p-3">
						<label class="text-xs text-gray-500 dark:text-gray-400" for="image-size">
							分辨率
						</label>
						<div class="grid grid-cols-2 md:grid-cols-4 gap-1.5 mt-2">
							{#each sizes as size}
								<button
									type="button"
									class="rounded-xl px-2.5 py-2 text-xs transition {config.size === size
										? 'bg-sky-100 text-sky-700 dark:bg-sky-400/20 dark:text-sky-200'
										: 'bg-white/80 hover:bg-white dark:bg-gray-900/60 dark:hover:bg-gray-800'}"
									on:click={() => applySize(size)}
								>
									{formatSize(size)}
								</button>
							{/each}
						</div>

						<div class="flex items-center gap-2 mt-2">
							<input
								id="image-size"
								class="min-w-0 flex-1 rounded-xl py-2 px-3 text-sm bg-white/80 dark:bg-gray-900 border border-gray-100 dark:border-gray-800 outline-hidden focus:border-sky-300 dark:focus:border-sky-500 transition"
								placeholder="自定义，如 3840x2160"
								bind:value={customSize}
								spellcheck="false"
								on:keydown={(event) => {
									if (event.key === 'Enter') {
										event.preventDefault();
										applySize(customSize);
									}
								}}
								on:blur={() => {
									if (customSize.trim()) applySize(customSize);
								}}
							/>
							<button
								type="button"
								class="shrink-0 rounded-xl px-3 py-2 text-sm font-medium bg-gray-900 text-white hover:bg-black dark:bg-white dark:text-gray-900 dark:hover:bg-gray-100 transition"
								on:click={() => applySize(customSize)}
							>
								应用
							</button>
						</div>

						<div class="mt-1.5 text-xs {sizeError ? 'text-red-500' : 'text-gray-500 dark:text-gray-400'}">
							{sizeError || '规则：最大边 ≤ 3840，宽高为 16 的倍数，比例不超过 3:1；不符合会回退为自动。'}
						</div>
					</div>
				</div>

				<div class="grid grid-cols-1 md:grid-cols-2 gap-3 mt-3">
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
