<script lang="ts">
    import * as THREE from 'three';
    import { T } from '@threlte/core';

    type Props = {
        start: [number, number, number] | undefined;
        end: [number, number, number] | undefined;
        strength: number; // value from data
    };

    let { start, end, strength }: Props = $props();

    // Reactive geometry generation
    const lineGeometry = $derived(() => {
        // Important: Only create geometry if start and end points are valid
        if (!start || !end) {
            console.warn('Link missing start or end position');
            return undefined; // Return undefined if points are missing
        }
        try {
            const points = [new THREE.Vector3(...start), new THREE.Vector3(...end)];
            return new THREE.BufferGeometry().setFromPoints(points);
        } catch (e) {
            console.error("Error creating link geometry:", e, { start, end });
            return undefined;
        }
    });

    const opacity = $derived(0.15 + strength * 0.4); // Adjusted opacity range

    // Dispose geometry when the component is destroyed or geometry changes
    $effect(() => {
        const currentGeometry = lineGeometry; // Capture current value
        return () => {
            currentGeometry?.dispose(); // Dispose previous geometry
        };
    });

</script>

{#if lineGeometry}
    <T.Line geometry={lineGeometry}>
        <T.LineBasicMaterial color="#a855f7" transparent={true} {opacity} />
    </T.Line>
{/if}