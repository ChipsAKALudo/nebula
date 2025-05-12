<script lang="ts">
    import { type Writable } from 'svelte/store';
    import * as THREE from 'three';
    import { Canvas, T, useThrelte } from '@threlte/core';
    import { OrbitControls, HTML, Environment } from '@threlte/extras';
    import { Button } from '$lib/components/ui/button';
    import { Slider } from '$lib/components/ui/slider';
    import { ZoomIn, ZoomOut, RotateCcw, Focus, Maximize, Minimize, Network } from 'lucide-svelte';

    import Node from '$lib/components/graph/node.svelte';
    import Link from '$lib/components/graph/link.svelte';

    // Define Props
    let { activeNote = $bindable() }: { activeNote: string | null } = $props();

    // Sample graph data (same as before)
    const graphData = {
        nodes: [
            { id: "1", label: "Quantum Physics", group: "science", size: 1.2 },
            { id: "2", label: "Project Milestones", group: "work", size: 1 },
            { id: "3", label: "Travel Ideas", group: "personal", size: 0.9 },
            { id: "4", label: "Book Recommendations", group: "personal", size: 1 },
            { id: "5", label: "Meeting Notes", group: "work", size: 0.8 },
            { id: "6", label: "Wave-Particle Duality", group: "science", size: 0.7 },
            { id: "7", label: "Quantum Entanglement", group: "science", size: 0.7 },
            { id: "8", label: "Japan", group: "travel", size: 0.6 },
            { id: "9", label: "Norway", group: "travel", size: 0.6 },
            { id: "10", label: "Project Hail Mary", group: "book", size: 0.5 },
        ],
        links: [
            { source: "1", target: "6", value: 1 },
            { source: "1", target: "7", value: 1 },
            { source: "2", target: "5", value: 0.7 },
            { source: "3", target: "8", value: 0.5 },
            { source: "3", target: "9", value: 0.5 },
            { source: "4", target: "10", value: 0.3 },
            { source: "6", target: "7", value: 0.8 },
            { source: "1", target: "4", value: 0.2 },
            { source: "2", target: "3", value: 0.1 },
            { source: "5", target: "2", value: 0.9 }, // Example of potential duplicate link direction
        ],
    };

    const groupColors: Record<string, string> = {
        science: '#8b5cf6', // purple
        work: '#3b82f6', // blue
        personal: '#10b981', // green
        travel: '#f59e0b', // amber
        book: '#ec4899' // pink
    };

    console.log("Calculating initial node positions...");
    const initialPositions: Record<string, [number, number, number]> = {};
    graphData.nodes.forEach((node, i) => {
        const angle = (i / graphData.nodes.length) * Math.PI * 2;
        // Math.random() is only called ONCE per node here
        const radius = 6 + Math.random() * 3; // Slightly larger base radius for better spread?
        initialPositions[node.id] = [
            Math.cos(angle) * radius,
            Math.sin(angle) * radius,
            (Math.random() - 0.5) * 6 // Slightly more depth variance?
        ];
    });
    console.log("Initial positions calculated:", initialPositions);

    // Component State
    let nodePositions = $state<Record<string, [number, number, number]>>(initialPositions);

    // --- Other Component State ---
    let activeNodeId = $state<string | null>(null);
    let zoomLevel = $state(5);
    let isFullscreen = $state(false);
    let controlsRef: OrbitControls | undefined = $state();
    const { camera } = useThrelte(); // Get camera reactively


    // Sync activeNote prop with internal state
    $effect(() => {
        if (activeNote !== activeNodeId) {
            activeNodeId = activeNote;
            // Optionally focus immediately when prop changes
            if (activeNote) {
              focusNode(activeNote);
            }
        }
    });

    // Focus on node logic
    function focusNode(nodeId: string | null) {
        console.log('Focusing on node:', nodeId);
        if (!nodeId) {
            console.warn('No node ID provided to focus on.'); // Maybe remove this warning for null
            return;
        }
        // Check reactive camera ($camera) exists via useThrelte()
        if (!controlsRef || !$camera) {
            console.warn('Focus aborted: Controls or Camera not ready.');
            return;
        }

        const positionArray = nodePositions[nodeId]; // Use the state variable
        if (!positionArray) {
            // This warning is important!
            console.warn(`Node position not found for id: ${nodeId}. Available positions:`, Object.keys(nodePositions));
            return;
        }

        const targetPosition = new THREE.Vector3(...positionArray);

        // --- Smooth Animation Logic (Optional but Recommended) ---
        // Use GSAP or a simple tweening function for smooth transitions
        // For simplicity, we set directly, but animation is much better UX.
        // Example: import { tweened } from 'svelte/motion'; const camPos = tweened(...);

        // Set OrbitControls target
        controlsRef.target.copy(targetPosition);

        // Calculate desired camera position (behind and slightly above the node)
        const offset = new THREE.Vector3(2, 1, 5); // Adjust offset as needed
        const desiredCamPos = targetPosition.clone().add(offset);

        // Set camera position directly (or animate it)
        $camera.position.copy(desiredCamPos);

        // Ensure controls update after manual changes
        controlsRef.update();
    }

    // Effect to focus camera when activeNodeId changes *internally*
    // (e.g., by clicking)
    // Make sure the effect syncing activeNote prop still works
    $effect(() => {
        console.log("Prop sync effect: activeNote=", activeNote, "activeNodeId=", activeNodeId);
        if (activeNote !== activeNodeId) {
            activeNodeId = activeNote;
            // Focus is now handled by the click or potentially the initial effect
            // You might still want to call focusNode here if the prop changes *after* initial load
            if (activeNote && Object.keys(nodePositions).length > 0) { // Ensure positions exist
               focusNode(activeNote);
            }
        }
    });


    // Derived state for active node details
    const activeNodeDetails = $derived(() => {
        if (!activeNodeId) return null;
        const node = graphData.nodes.find((n) => n.id === activeNodeId);
        if (!node) return null;
        // Count unique connections (handle bidirectional links)
        const connections = new Set<string>();
        graphData.links.forEach(link => {
            if (link.source === activeNodeId) connections.add(link.target);
            if (link.target === activeNodeId) connections.add(link.source);
        });
        return { label: node.label, connectionCount: connections.size };
    });

    // Event Handlers
    function handleZoomSlider(value: number[]) {
        zoomLevel = value[0];
        if (!controlsRef || !$camera) return;

        // Calculate desired distance based on slider (adjust mapping as needed)
        // Lower slider value = closer distance
        const desiredDistance = 2 + (10 - zoomLevel) * 1.8; // Map 1-10 to a distance range

        const target = controlsRef.target;
        const direction = new THREE.Vector3();
        direction.subVectors($camera.position, target).normalize(); // Get direction from target to camera

        const newCamPos = target.clone().add(direction.multiplyScalar(desiredDistance));
        $camera.position.copy(newCamPos);
        controlsRef.update(); // Update controls
    }

    function resetView() {
        activeNodeId = null; // Clear active node
        if (!controlsRef || !$camera) return;

        controlsRef.target.set(0, 0, 0); // Reset target
        $camera.position.set(0, 0, 15); // Reset camera position (adjust Z distance)
        controlsRef.update();
        zoomLevel = (15 - 2) / 1.8 + 1; // Reset slider roughly
        zoomLevel = 10 - (15-2)/1.8; // map distance back to zoomLevel (1..10) -> approx 2.7
        zoomLevel = Math.max(1, Math.min(10, 10 - (15-2)/1.8)); // Ensure it's within range

    }

    async function toggleFullscreen() {
        const container = document.querySelector<HTMLElement>('#graph-container'); // Need an ID on the container div
        if (!container) return;

        if (!document.fullscreenElement) {
            try {
                await container.requestFullscreen();
                isFullscreen = true;
            } catch (err: any) {
                console.error(`Error attempting to enable full-screen mode: ${err.message} (${err.name})`);
                // Fallback: Toggle class anyway for visual change if API fails
                isFullscreen = !isFullscreen;
            }
        } else {
            try {
                await document.exitFullscreen();
                isFullscreen = false;
            } catch (err: any) {
                console.error(`Error attempting to exit full-screen mode: ${err.message} (${err.name})`);
                // Fallback: Toggle class anyway
                isFullscreen = !isFullscreen;
            }
        }
    }
    $effect(() => {
        const handler = () => {
            isFullscreen = !!document.fullscreenElement;
        }
        document.addEventListener('fullscreenchange', handler);
        return () => document.removeEventListener('fullscreenchange', handler);
    });


    function handleNodeClick(nodeId: string) {
        // If clicking the already active node, maybe deselect? Optional.
        // if (activeNodeId === nodeId) {
        //  activeNodeId = null;
        // } else {
        activeNodeId = nodeId; // This will trigger the $effect to focus
        // }
    }
</script>

<div id="graph-container" class={`relative ${isFullscreen ? 'fixed inset-0 z-50 bg-background' : 'flex-1 h-full'}`}>
    <div class="absolute top-4 left-4 z-10 flex flex-col space-y-2">
        <Button variant="outline" size="icon" on:click={resetView} title="Reset View">
            <RotateCcw class="h-4 w-4" />
        </Button>
        <Button variant="outline" size="icon" on:click={toggleFullscreen} title={isFullscreen ? 'Exit Fullscreen' : 'Enter Fullscreen'}>
            {#if isFullscreen}
                <Minimize class="h-4 w-4" />
            {:else}
                <Maximize class="h-4 w-4" />
            {/if}
        </Button>
    </div>

    <div class="absolute top-4 right-4 z-10 bg-background/80 backdrop-blur-sm p-2 rounded-lg border border-border/40">
        <div class="flex items-center space-x-2 mb-2">
            <Network class="h-4 w-4 text-purple-400" />
            <span class="text-sm font-medium">Knowledge Graph</span>
        </div>
        <div class="flex items-center space-x-2">
            <ZoomOut class="h-3 w-3" />
            <Slider
                    value={[zoomLevel]}
                    min={1}
                    max={10}
                    step={0.1}
                    onValueChange={handleZoomSlider}
                    class="w-24"
                    aria-label="Zoom level"
            />
            <ZoomIn class="h-3 w-3" />
        </div>
    </div>

    <T.PerspectiveCamera makeDefault position={[0, 0, 15]} fov={60}>
        <OrbitControls
                bind:ref={controlsRef}
                enableDamping={true}
                dampingFactor={0.05}
                rotateSpeed={0.5}
                minDistance={1.5}
                maxDistance={30}
                target={[0,0,0]}
        />
    </T.PerspectiveCamera>

    <T.AmbientLight intensity={0.6} />
    <T.DirectionalLight position={[5, 10, 7]} intensity={1.2} castShadow shadow.mapSize={[2048, 2048]} />
    <T.DirectionalLight position={[-5, -5, -2]} intensity={0.4} />
    <Environment preset="city" background={false} />

    {#each graphData.links as link, i (link.source + '-' + link.target + '-' + i)}
        <Link
                start={nodePositions[link.source]}
                end={nodePositions[link.target]}
                strength={link.value}
        />
    {/each}

    {#each graphData.nodes as node (node.id)}
        <Node
                id={node.id}
                position={nodePositions[node.id]}
                label={node.label}
                group={node.group}
                size={node.size}
                {groupColors}
                {activeNodeId}
                onNodeClick={handleNodeClick}  />
    {/each}

    <T.Mesh rotation.x={-Math.PI / 2} position.y={-5} receiveShadow>
        <T.PlaneGeometry args={[50, 50]} />
        <T.MeshStandardMaterial color="#333" roughness={0.8} metalness={0.1} />
    </T.Mesh>

    {#if activeNodeDetails}
        {@const details = activeNodeDetails}
        <div class="absolute bottom-4 left-4 z-10 bg-background/80 backdrop-blur-sm p-3 rounded-lg border border-border/40 max-w-xs">
            <h3 class="text-sm font-medium mb-1">{details.label}</h3>
            <p class="text-xs text-muted-foreground">
                Connected to {details.connectionCount} {details.connectionCount === 1 ? 'note' : 'notes'}
            </p>
            <Button size="sm" variant="outline" class="mt-2 w-full" on:click={() => focusNode(activeNodeId)}>
                <Focus class="h-3 w-3 mr-1" />
                Focus on this note
            </Button>
        </div>
    {/if}
</div>