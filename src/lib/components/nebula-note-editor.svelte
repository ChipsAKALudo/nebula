<script lang="ts">
    import { Card, CardContent, CardHeader, CardTitle } from '$lib/components/ui/card'; // Assuming Card subcomponents are needed
    import { Button } from '$lib/components/ui/button';
    import * as Tabs from '$lib/components/ui/tabs';
    import {
        Bold, Italic, List, ListOrdered, ImageIcon, Link, Code, Heading1, Heading2, Sparkles, Save, Share
    } from 'lucide-svelte';
    import { marked } from 'marked'; // Recommended: Use a proper Markdown library

    // Props
    let { activeNote }: { activeNote: string | null } = $props();

    // Sample note data (same as React version)
    const sampleNotes: Record<string, { title: string; content: string; lastEdited: string }> = {
        "1": { title: "Quantum Physics Notes", content: "# Quantum Physics\n\nQuantum physics is the study of matter and energy at its most fundamental level. A central concept is quantum entanglement...", lastEdited: "2023-04-15T10:30:00Z" },
        "2": { title: "Project Milestones", content: "# Project Milestones\n\n## Phase 1: Research\n- [x] Market analysis...", lastEdited: "2023-04-10T14:45:00Z" },
        "3": { title: "Travel Ideas", content: "# Places to Visit\n\n1. Japan...", lastEdited: "2023-03-28T09:15:00Z" },
        "4": { title: "Book Recommendations", content: "# Reading List\n\n## Fiction...", lastEdited: "2023-04-05T16:20:00Z" },
        "5": { title: "Meeting Notes - Q2", content: "# Q2 Planning Meeting\n\n**Date:** April 3, 2023...", lastEdited: "2023-04-03T11:00:00Z" },
    };

    type Note = { title: string; content: string; lastEdited: string } | null;

    // Component State
    let note = $state<Note>(null);
    let editorMode = $state<'edit' | 'preview'>('edit');

    // Effect to load note when activeNote changes
    $effect(() => {
        if (activeNote && sampleNotes[activeNote]) {
            note = { ...sampleNotes[activeNote] }; // Create copy to avoid mutating original
        } else {
            // Fallback to the first note if activeNote is invalid or null initially
            note = { ...sampleNotes['1'] };
        }
        editorMode = 'edit'; // Reset to edit mode when note changes
    });

    // Derived state for computed values
    const wordCount = $derived(note?.content.split(/\s+/).filter(Boolean).length ?? 0);

    const previewHtml = $derived(() => {
        if (!note?.content) return '';
        // Use marked (or similar library) for safe and accurate Markdown parsing
        try {
            // Basic GFM support with breaks
            marked.setOptions({
                gfm: true,
                breaks: true,
            });
            return marked.parse(note.content);
        } catch (e) {
            console.error("Markdown parsing error:", e);
            return "<p>Error rendering preview.</p>"; // Fallback
        }

        // --- Original simple replacement (less robust, potential XSS risk) ---
        /*
        return note.content
           .replace(/^# (.*$)/gm, "<h1>$1</h1>")
           .replace(/^## (.*$)/gm, "<h2>$1</h2>")
           // ... other replacements ...
           .replace(/\n/g, "<br />");
        */
    });

    // Event Handlers (Placeholders for functionality)
    function saveNote() {
        console.log('Saving note:', note);
        // Add actual save logic here (e.g., API call, local storage)
    }

    function shareNote() {
        console.log('Sharing note:', note?.title);
        // Add sharing logic
    }

    function formatText(style: string) {
        console.log('Apply style:', style);
        // Implement actual text formatting logic on the textarea content
        // This often involves manipulating the selection range.
    }

    function aiAssist() {
        console.log('AI Assist clicked');
        // Implement AI feature
    }

</script>

{#if note}
    <div class="flex flex-col w-full md:w-[45%] h-full border-r border-border/40 overflow-hidden">
        <div class="flex items-center justify-between p-4 border-b border-border/40">
            <div>
                <h2 class="text-lg font-medium">{note.title}</h2>
                <p class="text-xs text-muted-foreground">
                    Last edited: {new Date(note.lastEdited).toLocaleString()}
                </p>
            </div>
            <div class="flex space-x-2">
                <Button size="sm" variant="outline" on:click={saveNote}>
                    <Save class="h-4 w-4 mr-1" />
                    Save
                </Button>
                <Button size="sm" variant="outline" on:click={shareNote}>
                    <Share class="h-4 w-4 mr-1" />
                    Share
                </Button>
            </div>
        </div>

        <Tabs.Root bind:value={editorMode} class="flex-1 flex flex-col">
            <div class="flex items-center justify-between px-4 py-2 border-b border-border/40 bg-background/50">
                <Tabs.List>
                    <Tabs.Trigger value="edit">Edit</Tabs.Trigger>
                    <Tabs.Trigger value="preview">Preview</Tabs.Trigger>
                </Tabs.List>

                {#if editorMode === 'edit'}
                    <div class="flex space-x-1">
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('bold')}><Bold class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('italic')}><Italic class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('h1')}><Heading1 class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('h2')}><Heading2 class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('ul')}><List class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('ol')}><ListOrdered class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('link')}><Link class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('image')}><ImageIcon class="h-4 w-4" /></Button>
                        <Button variant="ghost" size="icon" class="h-8 w-8" on:click={() => formatText('code')}><Code class="h-4 w-4" /></Button>
                    </div>
                {/if}

                <Button variant="outline" size="sm" class="ml-auto flex items-center" on:click={aiAssist}>
                    <Sparkles class="h-4 w-4 mr-1 text-purple-400" />
                    AI Assist
                </Button>
            </div>

            <div class="flex-1 overflow-auto">
                <Tabs.Content value="edit" class="h-full m-0 focus-visible:outline-none">
                    <div class="h-full relative">
						<textarea
                                class="w-full h-full p-6 resize-none focus:outline-none bg-[#fcfcfc] dark:bg-[#121212] font-mono text-sm leading-relaxed"
                                style="background-image: linear-gradient(to right, rgba(200, 200, 255, 0.05) 1px, transparent 1px), linear-gradient(to bottom, rgba(200, 200, 255, 0.05) 1px, transparent 1px); background-size: 20px 20px; box-shadow: inset 0 0 30px rgba(0, 0, 0, 0.05);"
                                bind:value={note.content}
                                placeholder="Start writing your note..."
                        />
                        <div class="absolute bottom-4 right-4 text-xs text-muted-foreground pointer-events-none">
                            {wordCount} words
                        </div>
                    </div>
                </Tabs.Content>

                <Tabs.Content value="preview" class="h-full m-0">
                    <Card class="h-full overflow-auto border-0 rounded-none shadow-none">
                        <CardContent class="p-6 prose dark:prose-invert max-w-none">
                            {@html previewHtml}
                        </CardContent>
                    </Card>
                </Tabs.Content>
            </div>
        </Tabs.Root>
    </div>
{:else}
    <div class="flex flex-col items-center justify-center w-full md:w-[45%] h-full border-r border-border/40 text-muted-foreground">
        <p>Select a note to view or edit.</p>
    </div>
{/if}