<script lang="ts">
	import { cn } from '$lib/utils';
	import { Badge } from '../ui/badge';
	import * as Dialog from '$lib/components/ui/dialog';
	import { page } from '$app/stores';
	import TagsPage from '../../../routes/tags/[tag]/+page.svelte';
	import { goto, preloadData, pushState } from '$app/navigation';

	export let tag: string;
	export let count: number = 0;
	export let href: string = '';

	let hover = false;
	let className: string | undefined | null = undefined;
	export { className as class };

	export let shallow: boolean = false;

	let showModal__: boolean = false;

	async function showModal(e: MouseEvent) {
		if (!shallow) return;
		if (e.metaKey || e.ctrlKey || innerWidth < 800) return;

		e.preventDefault();
		const { href } = e.currentTarget as HTMLAnchorElement;
		const result = await preloadData(href);
		console.log(result);
		if (result.type === 'loaded' && result.status === 200) {
			pushState(href, {
				tagsData: result.data
			});
			showModal__ = true;
		} else {
			goto(href);
		}

		// for some reason, going back doesn't close the dialog. so have to include this
		window.addEventListener('popstate', () => {
			showModal__ = false;
		});
	}
</script>

{#if count > 0}
	<a
		{href}
		class="flex rounded-md border border-muted-foreground/50 hover:border-primary"
		on:mouseenter={() => (hover = true)}
		on:mouseleave={() => (hover = false)}
	>
		<h1 class="px-2 text-base font-medium text-primary">
			{tag}
		</h1>
		<p
			class={cn(
				'rounded-r-[5px] border-l border-muted-foreground/50 bg-zinc-200 px-2 dark:bg-zinc-700/80',
				hover && 'border-primary'
			)}
		>
			{count}
		</p>
	</a>
{:else if shallow}
	<Badge class={cn('rounded', className)} {href} on:click={showModal}>{tag}</Badge>
{:else}
	<Badge on:click={showModal} class={cn('rounded', className)} {href}>{tag}</Badge>
{/if}

<Dialog.Root
	open={showModal__}
	onOpenChange={() => {
		history.back();
		showModal__ = false;
	}}
>
	<Dialog.Content class="h-[80vh] max-w-5xl overflow-scroll">
		<div class="h-full w-full">
			{#if $page.state.tagsData}
				<TagsPage data={$page.state.tagsData} />
			{/if}
		</div>
	</Dialog.Content>
</Dialog.Root>
