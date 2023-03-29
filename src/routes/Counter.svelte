<script lang="ts">
    import hotkeys, { type HotkeysEvent } from 'hotkeys-js';
    import { onDestroy } from 'svelte';

    
    export let approach: 'l' | 'r';
    export let recordVehicle: (approach: 'l' | 'r', description: 'left' | 'straight' | 'right' | 'heavy' | 'pedestrian', diff?: number) => void;

    $: keys = approach === 'l' ? {
        left: 'a',
        straight: 'w',
        right: 'd'
    } : {
        left: 'left',
        straight: 'up',
        right: 'right'
    }

    $: allKeys = `${keys.left},${keys.straight},${keys.right},shift+${keys.left},shift+${keys.straight},shift+${keys.right},space+${keys.left},space+${keys.straight},space+${keys.right}`;

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
        if (hotkeys.isPressed('space')) {
            recordVehicle(approach, 'pedestrian');
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
    {#if approach == 'l'}
        <kbd>A</kbd>
        <kbd>W</kbd>
        <kbd>D</kbd>
    {:else}
        <kbd>←</kbd>
        <kbd>↑</kbd>
        <kbd>→</kbd>
    {/if}
</div>

<style>
    div.grid {
        max-width: 500px;
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        grid-template-rows: repeat(2, 1fr);
        gap: 1ch;
        place-items: baseline center;
    }
</style>
