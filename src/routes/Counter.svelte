<script lang="ts">
    import hotkeys, { type HotkeysEvent } from 'hotkeys-js';
    import { onDestroy } from 'svelte';

    
    export let approach: 'l' | 'r';
    export let recordVehicle: (approach: 'l' | 'r', description: 'left' | 'straight' | 'right' | 'heavy' | 'pedestrian', diff?: number) => void;

    $: keys = approach === 'l' ? {
        left: 'a',
        straight: 'w',
        right: 'd',
        pedestrian: 's'
    } : {
        left: 'left',
        straight: 'up',
        right: 'right',
        pedestrian: 'down'
    }

    $: allKeys = `${keys.left},${keys.straight},${keys.right},${keys.pedestrian},shift+${keys.left},shift+${keys.straight},shift+${keys.right}`;

    $: keys && hotkeys(
        allKeys,
        {
            keydown: false,
            keyup: true
        },
        handleHotkey
    );

    function handleHotkey(event: KeyboardEvent, handler: HotkeysEvent) {
        event.preventDefault();

        // Pedestrian
        if (hotkeys.isPressed(keys.pedestrian)) {
            recordVehicle(approach, 'pedestrian');
            return;
        }

        // Shift: Heavy
        if (hotkeys.isPressed('shift')) {
            recordVehicle(approach, 'heavy');
        }

        if (hotkeys.isPressed(keys.left)) {
            recordVehicle(approach, 'left');
        } else if (hotkeys.isPressed(keys.straight)) {
            recordVehicle(approach, 'straight');
        } else if (hotkeys.isPressed(keys.right)) {
            recordVehicle(approach, 'right');
        }
    }

    onDestroy(()=>{
        hotkeys.unbind(allKeys);
    });
</script>

<div class="grid">
    <span
        class="key"
    >↰</span>
    <span
        class="key"
    >↑</span>
    <span
        class="key"
    >↱</span>
    <span>🚶</span>
    {#if approach == 'l'}
        <kbd>A</kbd>
        <kbd>W</kbd>
        <kbd>D</kbd>
        <kbd>S</kbd>
    {:else}
        <kbd>←</kbd>
        <kbd>↑</kbd>
        <kbd>→</kbd>
        <kbd>↓</kbd>
    {/if}
</div>

<style>
    div.grid {
        max-width: 500px;
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(2, 1fr);
        gap: 1ch;
        place-items: baseline center;
    }
</style>
