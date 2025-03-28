<script lang="ts">
	import Link from '$lib/components/Link.svelte'
	import Skeleton from '$lib/components/ui/skeleton/skeleton.svelte'
	import {
		activeWorkspace,
		activeWorkspaceDefaultApiKey,
		activeWorkspaceId,
		activeWorkspaceStats,
	} from '$lib/store'
	import { getLinkById, getWorkspaceStats } from '$lib/utils/api'
	import { ExternalLink, Meh } from 'lucide-svelte'
</script>

{#snippet StatsNumberCard(title: string, value: number)}
	<div class="flex flex-col gap-6 rounded border p-4 pb-6 hover:shadow">
		<h2 class="text-base font-semibold">{title}</h2>
		<div class="text-center text-7xl font-semibold text-brand-600">
			{value}
		</div>
	</div>
{/snippet}

<div class="flex w-full flex-grow flex-col gap-6">
	<h1 class="text-3xl font-semibold">Dashboard</h1>
	{#await $activeWorkspaceStats}
		<div class="flex gap-4">
			<Skeleton class="h-[130px] w-[300px]" />
			<Skeleton class="h-[130px] w-[300px]" />
			<Skeleton class="h-[130px] w-[300px]" />
		</div>
		<div class="flex gap-4">
			<Skeleton class="h-[300px] w-[500px]" />
			<Skeleton class="h-[300px] w-[500px]" />
		</div>
	{:then stats}
		{#if !stats}
			<div
				class="flex w-full flex-col items-center justify-center gap-2 py-16"
			>
				<Meh size={64} class="text-destructive" />
				<h1 class="text-xl font-semibold text-destructive">
					Oops! Something went wrong.
				</h1>
			</div>
		{:else}
			{@const data = stats}
			<div class="stats-grid grid gap-4">
				{@render StatsNumberCard('Total Links', data.numberOfLinks)}
				{@render StatsNumberCard(
					'Total Engagements',
					data.totalEngagements,
				)}
				{@render StatsNumberCard(
					'Total Engagements (Last Week)',
					data.totalEngagementsLastWeek,
				)}
			</div>
			{#if data.topPerformingLinks.length > 0}
				<div
					class="w-fit min-w-96 max-w-96 space-y-4 rounded border p-4 hover:shadow"
				>
					<h2 class="px-2 text-base font-semibold">
						Top Performing Links
					</h2>
					<div class="flex flex-col gap-2">
						{#snippet Skels()}
							<Skeleton class="h-[40px] w-full" />
							<Skeleton class="h-[40px] w-full" />
							<Skeleton class="h-[40px] w-full" />
						{/snippet}

						{#await $activeWorkspaceDefaultApiKey}
							{@render Skels()}
						{:then apiKey}
							{#each data.topPerformingLinks as performingLink, index}
								{#await getLinkById(performingLink.id, $activeWorkspaceId!, apiKey!.key)}
									<Skeleton class="h-[40px] w-full" />
								{:then { data: response, error }}
									{#if error || !response?.data?.data}
										<div class="flex p-2 text-destructive">
											Failed to fetch
										</div>
									{:else}
										{@const link = response.data.data}
										{@const linkHrefWithoutProtocol = `${location.host}/${link.shortName || link.id}`}
										{@const linkHref = `${location.origin}/${link.shortName || link.id}`}
										{@const hasTitle =
											link.title !== null ||
											link.title !== undefined ||
											link.title?.trim?.()?.length > 0}

										<div
											class="flex items-start gap-2 rounded border border-transparent p-2 transition hover:border-brand-600/20 hover:bg-brand-300/5"
										>
											<div class="text-sm font-semibold">
												{index + 1}.
											</div>
											<div
												class="flex flex-grow flex-col gap-0.5"
											>
												{#if link.title}
													<div
														class="max-w-[30ch] overflow-hidden text-ellipsis whitespace-nowrap text-sm"
													>
														{link.title?.trim?.()}
													</div>
												{:else}
													<Link
														href={linkHref}
														class="max-w-[30ch] overflow-hidden text-ellipsis whitespace-nowrap tracking-tighter"
													>
														{linkHrefWithoutProtocol}
													</Link>
												{/if}

												<Link
													href={link.title
														? linkHref
														: link.url}
													class="max-w-[30ch] overflow-hidden text-ellipsis whitespace-nowrap text-sm tracking-tighter text-muted-foreground"
												>
													{link.title
														? linkHrefWithoutProtocol
														: link.url}
												</Link>
											</div>
											<div
												class="text-sm font-semibold text-brand-600"
											>
												{performingLink.totalEngagements}
											</div>
										</div>
									{/if}
								{/await}
							{/each}
						{/await}
					</div>
				</div>
			{/if}
		{/if}
	{/await}
</div>

<style>
	.stats-grid {
		grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
	}
</style>
