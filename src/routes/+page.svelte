<script lang="ts">
	import '$lib/app.css';
	import { createVirtualizer, notUndefined } from '@tanstack/svelte-virtual';

	const numbers = Array.from({ length: 10000 }).map((_, i) => i + 1);

	let virtualListElement = $state<HTMLElement | null>(null);
	let virtualListItemElements: HTMLTableRowElement[] = $state([]);

	let virtualizer = $derived(
		createVirtualizer<HTMLElement, HTMLTableRowElement>({
			count: numbers.length,
			getScrollElement: () => virtualListElement,
			estimateSize: () => 24
		})
	);
	let virtualListItems = $derived($virtualizer.getVirtualItems());

	///////////// It doesn't work without this hack ////////////
	let mounted = $state(false);
	$effect(() => {
		if (!mounted && virtualListElement !== null) {
			mounted = true;
			$virtualizer._willUpdate();
		}
	});
	////////////////////////////////////////////////////////////

	// Ref: https://github.com/TanStack/virtual/issues/640#issuecomment-1885029911
	// Ref: https://github.com/TanStack/virtual/discussions/476#discussioncomment-4724139
	let [virtualListBefore, virtualListAfter] = $derived(
		virtualListItems.length > 0
			? [
					notUndefined(virtualListItems[0]).start - $virtualizer.options.scrollMargin,
					$virtualizer.getTotalSize() -
						notUndefined(virtualListItems[virtualListItems.length - 1]).end
				]
			: [0, 0]
	);

	$effect(() => {
		if (virtualListItemElements.length) {
			virtualListItemElements.forEach((el) => $virtualizer.measureElement(el));
		}
	});
</script>

<main>
	<h1>TanStack Virtual in Svelte 5</h1>

	<figure bind:this={virtualListElement}>
		<div>
			<table>
				<thead>
					<tr>
						<th>Number</th>
					</tr>
				</thead>

				<tbody>
					{#if virtualListBefore > 0}
						<tr>
							<td style={`height: ${virtualListBefore}px;`}></td>
						</tr>
					{/if}

					{#each virtualListItems as { index }, virtualListItemArrIndex (index)}
						{@const number = numbers[index]}
						<tr bind:this={virtualListItemElements[virtualListItemArrIndex]} data-index={index}>
							<td>{number}</td>
						</tr>
					{/each}

					{#if virtualListAfter > 0}
						<tr>
							<td style={`height: ${virtualListAfter}px;`}></td>
						</tr>
					{/if}
				</tbody>
			</table>
		</div>
	</figure>
</main>

<style>
	main {
		padding: 30px;
	}

	figure {
		overflow: auto;
		margin: 0;
	}

	table {
		border-collapse: collapse;
	}

	thead {
		position: sticky;
		top: 0;
		background: #fff;
	}

	th,
	td {
		height: 24px;
	}
</style>
