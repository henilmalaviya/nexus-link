<script lang="ts">
	import { Button } from '$lib/components/ui/button'
	import { Copy, Upload } from 'lucide-svelte'
	import * as Dialog from '$lib/components/ui/dialog'
	import { Label } from '$lib/components/ui/label'
	import { Input } from '$lib/components/ui/input'
	import { activeWorkspace, activeWorkspaceId, workspaces } from '$lib/store'
	import { toast } from 'svelte-sonner'
	import { CustomEvents, plausible } from '$lib/plausible'

	type Props = {
		compact?: boolean
	}

	let { compact = $bindable(false) }: Props = $props()

	let showDialog = $state(false)

	$effect(() => {
		if (showDialog) {
			plausible.trackEvent(CustomEvents.EXPORT_WORKSPACE)
		}
	})
</script>

<Dialog.Root bind:open={showDialog}>
	<Dialog.Content>
		<Dialog.Header>
			<Dialog.Title>Export Workspace</Dialog.Title>
			<Dialog.Description>
				These credentials allow others to access your workspace.
			</Dialog.Description>
		</Dialog.Header>

		<div class="flex items-end gap-1">
			<div class="flex w-full flex-col gap-1.5">
				<Label for="id">Workspace Id</Label>
				<Input
					type="text"
					id="id"
					disabled
					readonly
					placeholder="abcd123"
					bind:value={$activeWorkspaceId}
				/>
			</div>
			<Button
				onclick={() => {
					navigator.clipboard.writeText($activeWorkspaceId!)
					toast.info('Copied to clipboard')
				}}><Copy /></Button
			>
		</div>

		{#await $activeWorkspace then workspace}
			<div class="flex items-end gap-1">
				<div class="flex w-full flex-col gap-1.5">
					<Label for="secret">Workspace Secret</Label>
					<Input
						type="text"
						id="secret"
						disabled
						readonly
						placeholder="abcd123"
						bind:value={workspace!.secret}
					/>
				</div>
				<Button
					onclick={() => {
						navigator.clipboard.writeText(workspace!.secret)
						toast.info('Copied to clipboard')
					}}><Copy /></Button
				>
			</div>
		{/await}
	</Dialog.Content>
</Dialog.Root>

<Button class="w-full" onclick={() => (showDialog = true)}
	><Upload size={compact ? 18 : 20} />
	{#if compact === false}
		Export Workspace
	{/if}
</Button>
