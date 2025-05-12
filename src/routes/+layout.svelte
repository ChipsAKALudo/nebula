<script lang="ts">
    import { writable } from 'svelte/store';
	import '../app.css';

    export const isSidebarOpen = writable(true);

    let { children } = $props();
</script>

{@render children()}
