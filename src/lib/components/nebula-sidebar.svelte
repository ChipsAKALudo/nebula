<script lang="ts">
    import * as Tabs from '$lib/components/ui/tabs';
    import { Input } from '$lib/components/ui/input';
    import { Button } from '$lib/components/ui/button';
    import { ScrollArea } from '$lib/components/ui/scroll-area';
    import { FileText, Search, History, Star, Plus, Settings, Folder, FolderOpen, BookOpen, Sparkles, Menu } from 'lucide-svelte'; // Added Menu for potential trigger

    // Props with bindable runes
    let { activeTab = $bindable(), activeNote = $bindable() }: { activeTab: string, activeNote: string | null } = $props();
    // Alternatively, if you only want to SET the active note from here:
    // export let setActiveNote: (noteId: string) => void;

    // Sample data
    const notes = [
        { id: '1', title: 'Quantum Physics Notes', tags: ['physics', 'science'] },
        { id: '2', title: 'Project Milestones', tags: ['work', 'planning'] },
        { id: '3', title: 'Travel Ideas', tags: ['personal', 'travel'] },
        { id: '4', title: 'Book Recommendations', tags: ['reading', 'personal'] },
        { id: '5', title: 'Meeting Notes - Q2', tags: ['work', 'meetings'] }
    ];

    type Note = typeof notes[0];

    // Component State
    let searchQuery = $state('');

    // Derived State
    const filteredNotes = $derived(
        notes.filter((note) => note.title.toLowerCase().includes(searchQuery.toLowerCase()))
    );

    function handleNoteClick(noteId: string) {
        activeNote = noteId; // Directly update the bound prop
        // If using setActiveNote function prop:
        // setActiveNote(noteId);
    }

    function createNewNote() {
        console.log("Create new note");
        // Add logic to create a new note and potentially set it as active
    }

    function openSettings() {
        console.log("Open settings");
    }

    // Example: State for collapsible sidebar (if not using Sheet)
    let isCollapsed = $state(false);
    function toggleCollapse() {
        isCollapsed = !isCollapsed;
    }

</script>

<aside class={`flex flex-col h-full border-r border-border/40 transition-all duration-300 ${isCollapsed ? 'w-16' : 'w-72'}`}>
    <div class="flex items-center justify-between p-4 border-b border-border/40">
        {#if !isCollapsed}
            <div class="flex items-center space-x-2">
                <Sparkles class="h-5 w-5 text-purple-400" />
                <h2 class="text-lg font-semibold bg-gradient-to-r from-purple-400 to-blue-400 bg-clip-text text-transparent">
                    Nebula
                </h2>
            </div>
        {/if}
        <Button variant="ghost" size="icon" on:click={toggleCollapse} class="ml-auto">
            <Menu class="h-5 w-5" />
        </Button>
    </div>

    {#if !isCollapsed}
        <Tabs.Root bind:value={activeTab} class="flex-1 flex flex-col overflow-hidden">
            <Tabs.List class="grid grid-cols-4 gap-1 mx-2 my-2">
                <Tabs.Trigger value="notes" class="text-xs px-1 h-8 data-[state=active]:bg-muted/50">
                    <FileText class="h-4 w-4 sm:mr-1" />
                    <span class="hidden sm:inline">Notes</span>
                </Tabs.Trigger>
                <Tabs.Trigger value="search" class="text-xs px-1 h-8 data-[state=active]:bg-muted/50">
                    <Search class="h-4 w-4 sm:mr-1" />
                    <span class="hidden sm:inline">Search</span>
                </Tabs.Trigger>
                <Tabs.Trigger value="history" class="text-xs px-1 h-8 data-[state=active]:bg-muted/50">
                    <History class="h-4 w-4 sm:mr-1" />
                    <span class="hidden sm:inline">History</span>
                </Tabs.Trigger>
                <Tabs.Trigger value="favorites" class="text-xs px-1 h-8 data-[state=active]:bg-muted/50">
                    <Star class="h-4 w-4 sm:mr-1" />
                    <span class="hidden sm:inline">Favorites</span>
                </Tabs.Trigger>
            </Tabs.List>

            <div class="flex-1 overflow-hidden"> <Tabs.Content value="notes" class="mt-0 flex flex-col h-full">
                <div class="flex items-center p-2 border-b border-border/40">
                    <Input
                            placeholder="Filter notes..."
                            class="h-8 flex-1"
                            bind:value={searchQuery}
                    />
                    <Button size="icon" variant="ghost" class="h-8 w-8 ml-1" on:click={createNewNote}>
                        <Plus class="h-4 w-4" />
                    </Button>
                </div>

                <ScrollArea class="flex-1 p-2"> <div class="mb-4">
                    <h3 class="text-xs font-semibold text-muted-foreground px-2 py-1 flex items-center">
                        <FolderOpen class="h-4 w-4 mr-2" /> Recent Notes
                    </h3>
                    <ul>
                        {#each filteredNotes as note (note.id)}
                            <li>
                                <button
                                        type="button"
                                        on:click={() => handleNoteClick(note.id)}
                                        class="flex items-center w-full text-left px-2 py-1.5 text-sm rounded-md hover:bg-muted group {activeNote === note.id ? 'bg-muted font-medium' : ''}"
                                >
                                    <BookOpen class="h-4 w-4 mr-2 text-purple-400 group-hover:text-purple-300 flex-shrink-0" />
                                    <span class="truncate flex-1">{note.title}</span>
                                </button>
                            </li>
                        {:else}
                            <p class="text-xs text-muted-foreground p-2">No matching notes.</p>
                        {/each}
                    </ul>
                </div>

                    <div>
                        <h3 class="text-xs font-semibold text-muted-foreground px-2 py-1 flex items-center">
                            <Folder class="h-4 w-4 mr-2" /> Collections
                        </h3>
                        <ul>
                            <li>
                                <button type="button" class="flex items-center w-full text-left px-2 py-1.5 text-sm rounded-md hover:bg-muted group">
                                    <div class="h-3 w-3 rounded-full bg-blue-400 mr-2 flex-shrink-0" />
                                    <span class="truncate flex-1">Research</span>
                                </button>
                            </li>
                            <li>
                                <button type="button" class="flex items-center w-full text-left px-2 py-1.5 text-sm rounded-md hover:bg-muted group">
                                    <div class="h-3 w-3 rounded-full bg-green-400 mr-2 flex-shrink-0" />
                                    <span class="truncate flex-1">Personal</span>
                                </button>
                            </li>
                            <li>
                                <button type="button" class="flex items-center w-full text-left px-2 py-1.5 text-sm rounded-md hover:bg-muted group">
                                    <div class="h-3 w-3 rounded-full bg-purple-400 mr-2 flex-shrink-0" />
                                    <span class="truncate flex-1">Work</span>
                                </button>
                            </li>
                        </ul>
                    </div>
                </ScrollArea>
            </Tabs.Content>

                <Tabs.Content value="search" class="mt-0 p-4 h-full overflow-y-auto">
                    <Input placeholder="Search all notes..." class="mb-2 h-8" />
                    <p class="text-sm text-muted-foreground p-2">
                        Enter keywords to search across all your notes and connections. (Search results would appear here)
                    </p>
                </Tabs.Content>

                <Tabs.Content value="history" class="mt-0 p-4 h-full overflow-y-auto">
                    <p class="text-sm text-muted-foreground p-2">Your recently viewed and edited notes will appear here.</p>
                </Tabs.Content>

                <Tabs.Content value="favorites" class="mt-0 p-4 h-full overflow-y-auto">
                    <p class="text-sm text-muted-foreground p-2">
                        Star your important notes to access them quickly from this tab.
                    </p>
                </Tabs.Content>
            </div>
        </Tabs.Root>

        <div class="p-4 mt-auto border-t border-border/40">
            <Button variant="outline" size="sm" class="w-full flex items-center justify-center" on:click={openSettings}>
                <Settings class="h-4 w-4 mr-2" />
                Settings
            </Button>
        </div>
    {:else}
        <div class="flex flex-col items-center p-2 space-y-3 mt-4">
            <Button variant={activeTab === 'notes' ? 'secondary' : 'ghost'} size="icon" on:click={() => activeTab = 'notes'} title="Notes"><FileText class="h-5 w-5" /></Button>
            <Button variant={activeTab === 'search' ? 'secondary' : 'ghost'} size="icon" on:click={() => activeTab = 'search'} title="Search"><Search class="h-5 w-5" /></Button>
            <Button variant={activeTab === 'history' ? 'secondary' : 'ghost'} size="icon" on:click={() => activeTab = 'history'} title="History"><History class="h-5 w-5" /></Button>
            <Button variant={activeTab === 'favorites' ? 'secondary' : 'ghost'} size="icon" on:click={() => activeTab = 'favorites'} title="Favorites"><Star class="h-5 w-5" /></Button>
            <Button variant="ghost" size="icon" class="mt-auto" on:click={createNewNote} title="New Note"><Plus class="h-5 w-5" /></Button>
        </div>
        <div class="p-2 mt-auto">
            <Button variant="ghost" size="icon" class="w-full" on:click={openSettings} title="Settings">
                <Settings class="h-5 w-5" />
            </Button>
        </div>
    {/if}


</aside>