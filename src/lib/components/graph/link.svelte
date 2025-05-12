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
        console.log('Generating link geometry:', { start, end });
        // Important: Only create geometry if start and end points are valid
        if (!start || !end) {
            console.log('Link missing start or end position');
            console.warn('Link missing start or end position');
            return undefined; // Return undefined if points are missing
        }
        try {
            console.log('Creating link geometry:', { start, end });
            const points = [new THREE.Vector3(...start), new THREE.Vector3(...end)];
            return new THREE.BufferGeometry().setFromPoints(points);
        } catch (e) {
            console.log('Error creating link geometry:', e, { start, end });
            console.error("Error creating link geometry:", e, { start, end });
            return undefined;
        }
    });

    const opacity = $derived(0.15 + strength * 0.4); // Adjusted opacity range

    // Dispose geometry when the component is destroyed or geometry changes
    $effect(() => {
        const currentGeometry = lineGeometry; // Capture current value
        return () => {
            console.log('Cleanup check for geometry:', currentGeometry, typeof currentGeometry);
            if (currentGeometry && typeof currentGeometry.dispose === 'function') {
                console.log('Cleanup: Disposing geometry ID:', currentGeometry.uuid);
                currentGeometry.dispose();
            }
        };
    });

</script>

{#if lineGeometry instanceof THREE.BufferGeometry}
    <T.Line geometry={lineGeometry}>
        <T.LineBasicMaterial color="#a855f7" transparent={true} {opacity} />
    </T.Line>
{/if}