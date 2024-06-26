<script lang="ts">
	import BranchLane from '$lib/components//BranchLane.svelte';
	import Button from '$lib/components/Button.svelte';
	import FullviewLoading from '$lib/components/FullviewLoading.svelte';
	import Modal from '$lib/components/Modal.svelte';
	import type { PageData } from './$types';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	export let data: PageData;

	let applyConflictedModal: Modal;
	let deleteBranchModal: Modal;

	$: ({ projectId, branchController, vbranchService } = data);
	$: branches$ = vbranchService.branches$;
	$: error = vbranchService.branchesError;

	$: branch = $branches$?.find((b) => b.id == $page.params.branchId);
</script>

{#if $error}
	<p>{JSON.stringify($error)}</p>
{:else if !$branches$}
	<FullviewLoading />
{:else if branch}
	<BranchLane {branch} isUnapplied={!branch.active} />
{:else}
	<p>Branch no longer exists</p>
{/if}

<Modal width="small" title="Merge conflicts" bind:this={applyConflictedModal}>
	<p>Applying this branch will introduce merge conflicts.</p>
	<svelte:fragment slot="controls" let:item let:close>
		<Button style="ghost" kind="solid" on:click={close}>Cancel</Button>
		<Button
			style="pop"
			kind="solid"
			on:click={() => {
				branchController.applyBranch(item.id);
				close();
				goto(`/${projectId}/board`);
			}}
		>
			Update
		</Button>
	</svelte:fragment>
</Modal>

<Modal width="small" title="Delete branch" bind:this={deleteBranchModal} let:item>
	<div>
		Deleting <code>{item.name}</code> cannot be undone.
	</div>
	<svelte:fragment slot="controls" let:close let:item>
		<Button style="ghost" kind="solid" on:mousedown={close}>Cancel</Button>
		<Button
			style="error"
			kind="solid"
			on:click={() => {
				branchController.deleteBranch(item.id);
				close();
				goto(`/${projectId}/board`);
			}}
		>
			Delete
		</Button>
	</svelte:fragment>
</Modal>
