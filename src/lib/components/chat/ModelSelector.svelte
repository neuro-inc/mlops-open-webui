<script lang="ts">
	import { models, showSettings, settings, user, mobile, config } from '$lib/stores';
	import { onMount, tick, getContext } from 'svelte';
	import { toast } from 'svelte-sonner';
	import Selector from './ModelSelector/Selector.svelte';
	import Tooltip from '../common/Tooltip.svelte';

	import { updateUserSettings } from '$lib/apis/users';
	const i18n = getContext('i18n');

	export let selectedModels = [''];
	export let disabled = false;

	export let showSetDefault = true;

	const saveDefaultModel = async () => {
		const hasEmptyModel = selectedModels.filter((it) => it === '');
		if (hasEmptyModel.length) {
			toast.error($i18n.t('Choose a model before saving...'));
			return;
		}
		settings.set({ ...$settings, models: selectedModels });
		await updateUserSettings(localStorage.token, { ui: $settings });

		toast.success($i18n.t('Default model updated'));
	};

	$: if (selectedModels.length > 0 && $models.length > 0) {
		selectedModels = selectedModels.map((model) =>
			$models.map((m) => m.id).includes(model) ? model : ''
		);
	}
</script>

<div class="flex flex-col w-full items-start">
	{#each selectedModels as selectedModel, selectedModelIdx}
		<div class="flex w-full max-w-fit">
			<div class="overflow-hidden w-full">
				<div class="mr-1 max-w-full">
					<Selector
						id={`${selectedModelIdx}`}
						placeholder={$i18n.t('Select a model')}
						items={$models.map((model) => ({
							value: model.id,
							label: model.name,
							model: model
						}))}
						showTemporaryChatControl={$user?.role === 'user'
							? ($user?.permissions?.chat?.temporary ?? true) &&
								!($user?.permissions?.chat?.temporary_enforced ?? false)
							: true}
						bind:value={selectedModel}
					/>
				</div>
			</div>

			{#if $user?.role === 'admin' || ($user?.permissions?.chat?.multiple_models ?? true)}
				{#if selectedModelIdx === 0}
					<div
						class="  self-center mx-1 disabled:text-gray-600 disabled:hover:text-gray-600 -translate-y-[0.5px]"
					>
						<Tooltip content={$i18n.t('Add Model')}>
							<button
								class=" "
								{disabled}
								on:click={() => {
									selectedModels = [...selectedModels, ''];
								}}
								aria-label="Add Model"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="2"
									stroke="currentColor"
									class="size-3.5"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m6-6H6" />
								</svg>
							</button>
						</Tooltip>
					</div>
				{:else}
					<div
						class="  self-center mx-1 disabled:text-gray-600 disabled:hover:text-gray-600 -translate-y-[0.5px]"
					>
						<Tooltip content={$i18n.t('Remove Model')}>
							<button
								{disabled}
								on:click={() => {
									selectedModels.splice(selectedModelIdx, 1);
									selectedModels = selectedModels;
								}}
								aria-label="Remove Model"
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="2"
									stroke="currentColor"
									class="size-3"
								>
									<path stroke-linecap="round" stroke-linejoin="round" d="M19.5 12h-15" />
								</svg>
							</button>
						</Tooltip>
					</div>
				{/if}
			{/if}
		</div>
	{/each}
</div>

<div class="flex w-full">
	<svg style="height: 50px; fill: #00747f" xmlns="http://www.w3.org/2000/svg" viewBox="18 29.1 324 157.8"><path class="st0" d="M145.6 67.4c.8 1.4 1.9 2.5 3.3 3.2 1.3.7 2.8 1.1 4.6 1.1 1.1 0 2.1-.2 3-.5.9-.3 1.7-.8 2.4-1.4.7-.6 1.2-1.3 1.6-2.1.4-.8.6-1.7.6-2.8 0-1.2-.2-2.2-.7-2.9-.5-.8-1.2-1.4-2-1.9s-1.8-.9-2.8-1.3c-1-.4-2.1-.7-3.2-1.1-1.1-.4-2.2-.8-3.2-1.2-1-.5-2-1-2.8-1.8-.8-.7-1.5-1.6-2-2.7s-.7-2.4-.7-4c0-1.5.3-2.9.9-4 .6-1.2 1.4-2.2 2.4-2.9 1-.8 2.2-1.4 3.5-1.8 1.3-.4 2.7-.6 4.1-.6 1.9 0 3.6.3 5.2 1s3 1.8 4.1 3.3l-2.7 2c-.8-1.1-1.7-2-2.8-2.5-1.1-.6-2.4-.8-3.8-.8-1 0-2 .1-2.9.4-.9.3-1.7.7-2.4 1.2s-1.2 1.2-1.7 2c-.4.8-.6 1.8-.6 2.8 0 1.7.4 3 1.3 3.9.9.9 2 1.6 3.3 2.2 1.3.5 2.7 1 4.2 1.4s2.9.9 4.2 1.6c1.3.7 2.4 1.6 3.3 2.8.9 1.2 1.3 2.9 1.3 5 0 1.5-.3 2.9-.9 4.1-.6 1.2-1.4 2.2-2.3 3-1 .8-2.1 1.4-3.4 1.9-1.3.4-2.6.6-4 .6-2.3 0-4.3-.4-6.2-1.2-1.9-.8-3.5-2.1-4.9-4l2.7-2zM173.6 39.6h3.2v15h.3l16.5-15h4.4l-17.3 15.6 18.2 18.5h-4.6L177 56h-.3v17.6h-3.2v-34h.1zM201.1 56.6c0-2.6.4-5 1.3-7.2s2.1-4.1 3.6-5.7c1.5-1.6 3.4-2.8 5.6-3.7 2.2-.9 4.6-1.3 7.2-1.3 2.6 0 5 .4 7.2 1.3s4 2.1 5.6 3.7c1.5 1.6 2.7 3.5 3.6 5.7.9 2.2 1.3 4.6 1.3 7.2s-.4 5-1.3 7.2-2.1 4.1-3.6 5.7-3.4 2.8-5.6 3.7c-2.2.9-4.6 1.3-7.2 1.3-2.6 0-5-.4-7.2-1.3s-4-2.1-5.6-3.7-2.7-3.5-3.6-5.7c-.9-2.2-1.3-4.6-1.3-7.2zm3.1 0c0 2.1.3 4.1 1 5.9s1.6 3.4 2.9 4.8c1.2 1.3 2.8 2.4 4.5 3.2 1.8.8 3.8 1.2 6 1.2s4.2-.4 6-1.2c1.8-.8 3.3-1.8 4.6-3.2 1.2-1.4 2.2-2.9 2.9-4.8.7-1.8 1-3.8 1-5.9s-.3-4.1-1-5.9c-.7-1.8-1.6-3.4-2.9-4.8-1.3-1.3-2.8-2.4-4.6-3.2-1.8-.8-3.8-1.2-6-1.2s-4.2.4-6 1.2c-1.8.8-3.3 1.8-4.5 3.2-1.3 1.4-2.2 2.9-2.9 4.8-.6 1.8-1 3.8-1 5.9zM245.4 39.6h3.2v15h.3l16.5-15h4.4l-17.3 15.6 18.2 18.5h-4.6L248.9 56h-.3v17.6h-3.2v-34zM279.8 39.6h3.2v34.1h-3.2V39.6zM293.9 39.6h20.9v2.9h-17.7v12.1h16.6v2.9h-16.6v13.3h18.5v2.9H294l-.1-34.1zM144.7 88.9h9.8c1.2 0 2.4.1 3.7.4 1.3.3 2.5.7 3.6 1.4s2 1.6 2.7 2.8 1 2.6 1 4.4c0 1.5-.3 2.9-.8 4-.6 1.1-1.3 2.1-2.3 2.8-.9.7-2 1.3-3.3 1.6-1.2.4-2.5.5-3.9.5h-7.4V123h-3.2V88.9h.1zm3.2 15h7.2c.9 0 1.8-.1 2.7-.3.9-.2 1.6-.5 2.3-1 .7-.5 1.2-1.1 1.6-1.9.4-.8.6-1.7.6-2.8s-.2-2-.7-2.8c-.4-.8-1-1.4-1.7-1.9s-1.5-.9-2.4-1.1c-.9-.2-1.8-.3-2.7-.3h-7v12.1h.1zM199.4 109.9c0 1.8-.2 3.6-.6 5.3-.4 1.7-1.1 3.2-2.1 4.4-1 1.3-2.3 2.3-3.8 3.1-1.6.8-3.5 1.2-5.9 1.2s-4.3-.4-5.9-1.2c-1.6-.8-2.9-1.8-3.9-3.1s-1.7-2.8-2.1-4.4c-.4-1.7-.6-3.4-.6-5.3v-21h3.2v20.2c0 1.2.1 2.5.3 3.9s.7 2.7 1.3 3.9c.7 1.2 1.6 2.2 2.8 2.9 1.2.8 2.8 1.2 4.8 1.2s3.6-.4 4.8-1.2 2.1-1.8 2.8-2.9c.7-1.2 1.1-2.5 1.3-3.9s.3-2.7.3-3.9V88.9h3.2v21h.1zM210 88.9h9.8c1.3 0 2.6.1 3.9.3 1.3.2 2.5.6 3.5 1.3s1.9 1.6 2.6 2.7c.7 1.2 1 2.7 1 4.7 0 1.7-.5 3.2-1.5 4.5-1 1.3-2.6 2.1-4.6 2.6v.1c1.1.1 2.2.4 3.1.8.9.4 1.7 1 2.4 1.7s1.2 1.6 1.6 2.6c.4 1 .6 2.1.6 3.3 0 2-.4 3.6-1.1 4.8-.8 1.2-1.7 2.2-2.8 2.9s-2.4 1.2-3.7 1.4c-1.3.3-2.6.4-3.7.4H210V88.9zm3.2 14.7h6.6c1.6 0 3-.2 4-.6s1.8-.9 2.4-1.5c.6-.6 1-1.2 1.2-1.9.2-.7.3-1.3.3-1.8 0-1.2-.2-2.2-.6-3-.4-.8-.9-1.4-1.6-1.9s-1.5-.8-2.4-1c-.9-.2-1.8-.3-2.8-.3h-7l-.1 12zm0 16.5h7.3c1.9 0 3.4-.2 4.5-.7 1.1-.4 2-1 2.6-1.7.6-.7 1-1.4 1.2-2.1.2-.8.3-1.4.3-2 0-1.3-.2-2.3-.7-3.2-.5-.9-1.1-1.6-1.9-2.2-.8-.6-1.6-1-2.6-1.2-1-.3-2-.4-3-.4h-7.8v13.5h.1zM242.1 88.9h3.2v31.2h15.9v2.9h-19.1V88.9zM270.1 88.9h3.2V123h-3.2V88.9zM313 117.6c-1.6 2.2-3.5 3.8-5.7 4.8s-4.5 1.4-7 1.4c-2.6 0-5-.4-7.2-1.3s-4-2.1-5.6-3.7c-1.5-1.6-2.7-3.5-3.6-5.7-.9-2.2-1.3-4.6-1.3-7.2s.4-5 1.3-7.2 2.1-4.1 3.6-5.7 3.4-2.8 5.6-3.7c2.2-.9 4.6-1.3 7.2-1.3 2.3 0 4.5.4 6.5 1.2 2 .8 3.8 2.2 5.3 4.1l-2.6 2.2c-1.1-1.6-2.4-2.8-4.1-3.5-1.7-.7-3.3-1.1-5.1-1.1-2.2 0-4.2.4-6 1.2-1.8.8-3.3 1.8-4.6 3.2-1.3 1.3-2.2 2.9-2.9 4.8-.7 1.8-1 3.8-1 5.9s.3 4.1 1 5.9c.7 1.8 1.6 3.4 2.9 4.8 1.3 1.3 2.8 2.4 4.6 3.2 1.8.8 3.8 1.2 6 1.2.9 0 1.8-.1 2.7-.3s1.8-.5 2.7-1c.9-.4 1.7-1 2.5-1.6.8-.7 1.5-1.5 2.1-2.4l2.7 1.8zM144.7 138.2h3.2v31.2h15.9v2.9h-19.1v-34.1zM172.6 138.2h3.2v34.1h-3.2v-34.1zM186.4 138.2h9.8c1.3 0 2.6.1 3.9.3 1.3.2 2.5.6 3.5 1.3s1.9 1.6 2.6 2.7c.7 1.2 1 2.7 1 4.6 0 1.7-.5 3.2-1.5 4.5-1 1.3-2.6 2.1-4.6 2.6v.1c1.1.1 2.2.4 3.1.8.9.4 1.7 1 2.4 1.7s1.2 1.6 1.6 2.6c.4 1 .6 2.1.6 3.3 0 2-.4 3.6-1.1 4.8-.8 1.2-1.7 2.2-2.8 2.9-1.1.7-2.4 1.2-3.7 1.4-1.3.3-2.6.4-3.7.4h-10.9v-34h-.2zm3.2 14.7h6.6c1.6 0 3-.2 4-.6 1-.4 1.8-.9 2.4-1.5.6-.6 1-1.2 1.2-1.9.2-.7.3-1.3.3-1.8 0-1.2-.2-2.2-.6-3-.4-.8-.9-1.4-1.6-1.9-.7-.5-1.5-.8-2.4-1s-1.8-.3-2.8-.3h-7v12h-.1zm0 16.5h7.3c1.9 0 3.4-.2 4.5-.7 1.1-.4 2-1 2.6-1.7s1-1.4 1.2-2.1c.2-.8.3-1.4.3-2.1 0-1.3-.2-2.3-.7-3.2-.5-.9-1.1-1.6-1.9-2.2-.8-.6-1.7-1-2.6-1.2-1-.3-2-.4-3-.4h-7.8l.1 13.6zM218.2 138.2H229c.8 0 1.6.1 2.5.3s1.8.4 2.7.8c.9.4 1.8.9 2.5 1.5.7.7 1.3 1.5 1.8 2.6.5 1 .7 2.3.7 3.8 0 1.6-.3 3-.8 4-.6 1.1-1.3 1.9-2.1 2.6s-1.8 1.2-2.8 1.5-2 .5-2.9.7l9.9 16.4h-3.6l-9.6-16.2h-5.9v16.2h-3.2v-34.2zm3.2 15h6.7c1.7 0 3-.2 4.1-.6 1-.4 1.9-.9 2.4-1.5.6-.6 1-1.3 1.2-2 .2-.7.3-1.4.3-1.9 0-.6-.1-1.2-.3-1.9s-.6-1.4-1.2-2c-.6-.6-1.4-1.1-2.4-1.5-1-.4-2.4-.6-4.1-.6h-6.7v12zM261.3 138.2h3.3l14.4 34.1h-3.6l-3.7-9.1h-18.1l-3.9 9.1h-3.3l14.9-34.1zm1.5 3.5l-8 18.7h15.7l-7.7-18.7zM286.9 138.2h10.8c.8 0 1.6.1 2.5.3.9.2 1.8.4 2.7.8.9.4 1.8.9 2.5 1.5.7.7 1.3 1.5 1.8 2.6.5 1 .7 2.3.7 3.8 0 1.6-.3 3-.8 4-.6 1.1-1.3 1.9-2.1 2.6s-1.8 1.2-2.8 1.5-2 .5-2.9.7l9.9 16.4h-3.6l-9.6-16.2h-5.9v16.2h-3.2v-34.2zm3.2 15h6.7c1.7 0 3-.2 4.1-.6 1-.4 1.9-.9 2.4-1.5.6-.6 1-1.3 1.2-2 .2-.7.3-1.4.3-1.9 0-.6-.1-1.2-.3-1.9s-.6-1.4-1.2-2c-.6-.6-1.4-1.1-2.4-1.5-1-.4-2.4-.6-4.1-.6h-6.7v12zM326.3 157.6l-12.5-19.4h3.8l10.3 16.2 10.5-16.2h3.6l-12.5 19.4v14.7h-3.2v-14.7zM80.8 186.9L55 175.6 18 77.7l73.1-48.6-.4 39.2 32.5-21.9L108 169.5l-21.5-9.4 1-86-21.7 14.6 15 98.2zm-23.4-13.6l19.5 8.5-14.5-94.6 25.1-16.9.4-35.5-66.1 44 35.6 94.5zm32.2-15.2l15.8 6.9 13.8-112.2L90.6 72l-1 86.1z"></path></svg>
</div>

{#if showSetDefault}
	<div class=" absolute text-left mt-[1px] ml-1 text-[0.7rem] text-gray-500 font-primary">
		<button on:click={saveDefaultModel}> {$i18n.t('Set as default')}</button>
	</div>
{/if}
