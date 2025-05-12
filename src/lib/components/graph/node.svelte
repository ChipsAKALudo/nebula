<script lang="ts">
    import * as THREE from 'three';
    import { T } from '@threlte/core';
    import { HTML } from '@threlte/extras';
    import { type Writable } from 'svelte/store'; // For activeNodeId type if needed

    type Props = {
        id: string;
        position: [number, number, number];
        label: string;
        group: string;
        size: number;
        groupColors: Record<string, string>;
        activeNodeId: string | null;
        onNodeClick: (id: string) => void;
    };

    let {
        id,
        position,
        label,
        group,
        size,
        groupColors,
        activeNodeId,
        onNodeClick
    }: Props = $props();

    let hovered = $state(false);
    const isActive = $derived(id === activeNodeId);
    const color = $derived(groupColors[group] || '#8b5cf6'); // Default color
    // Use size prop directly, scale affects the T.Mesh
    const nodeScale = $derived(isActive ? size * 1.3 : hovered ? size * 1.2 : size);
    const sphereRadius = 0.5; // Base radius, scaling applied to mesh
    const emissiveIntensity = $derived(isActive ? 0.4 : hovered ? 0.2 : 0);
    const showLabel = $derived(hovered || isActive);

    function handleClick() {
        onNodeClick(id);
    }

    function handlePointerEnter() {
        hovered = true;
        document.body.style.cursor = 'pointer'; // Add visual feedback
    }

    function handlePointerLeave() {
        hovered = false;
        document.body.style.cursor = 'auto';
    }

    // If you need per-frame animation later, add useTask here:
    // import { useTask } from '@threlte/core';
    // let meshRef: THREE.Mesh | undefined = $state(); // bind:ref={meshRef} on T.Mesh
    // useTask((delta) => {
    //   if (!meshRef) return;
    //   // meshRef.rotation.y += delta * 0.5;
    // });

</script>

<T.Group {position}>
    <T.Mesh
            scale={nodeScale}
            on:click={handleClick}
            on:pointerenter={handlePointerEnter}
            on:pointerleave={handlePointerLeave}
            castShadow
            receiveShadow
    >
        <T.SphereGeometry args={[sphereRadius, 32, 32]} />
        <T.MeshStandardMaterial
                {color}
                roughness={0.3}
                metalness={0.2}
                emissive={color}
                {emissiveIntensity}
        />
    </T.Mesh>

    {#if showLabel}
        <HTML distanceFactor={10} occlude>
        <div
                class="px-2 py-1 rounded-md bg-background/90 border border-border shadow-md text-xs whitespace-nowrap select-none pointer-events-none"
        >
            {label}
        </div>
        </HTML>
    {/if}
</T.Group>