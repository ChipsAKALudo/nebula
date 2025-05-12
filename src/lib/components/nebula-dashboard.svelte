<script lang="ts">
    import { onMount } from 'svelte';
    import { browser } from '$app/environment'; // Import the browser check

    import NebulaSidebar from '$lib/components/nebula-sidebar.svelte';
    import NebulaNoteEditor from '$lib/components/nebula-note-editor.svelte';
    // Remove the static import of NebulaGraph
    // import NebulaGraph from '$lib/components/nebula-graph.svelte';

    // Component state
    let activeNote = $state<string | null>(null);
    let sidebarTab = $state<string>('notes');
    // let sidebarOpen = $state<boolean>(true); // Use if needed for collapsible sidebar

    // State to hold the dynamically imported component
    let NebulaGraphComponent: typeof import('$lib/components/nebula-graph.svelte').default | null = $state(null);

    // Use onMount to ensure this runs only on the client AFTER the component is in the DOM
    onMount(() => {
        // Double-check with 'browser' although onMount itself is client-side
        import('$lib/components/nebula-graph.svelte').then((module) => {
            NebulaGraphComponent = module.default;
        }).catch(error => {
            console.error("Failed to load NebulaGraph component:", error);
            // Handle loading error if necessary
        });
    });

</script>

<div class="h-screen w-full overflow-hidden bg-gradient-to-b from-background to-background/90">
    <div class="flex h-full">
        <NebulaSidebar bind:activeTab={sidebarTab} bind:activeNote />

        <div class="flex flex-1 flex-col md:flex-row overflow-hidden">
            <NebulaNoteEditor {activeNote} />

            {#if NebulaGraphComponent}
                <svelte:component this={NebulaGraphComponent} {activeNote} />
            {:else}
                <div class="flex-1 h-full flex items-center justify-center bg-muted/20">
                    <p class="text-muted-foreground animate-pulse">Loading Graph...</p>
                </div>
            {/if}
        </div>
    </div>
</div>

<style lang="postcss">
    /* Add any component-specific styles here if needed */
</style>