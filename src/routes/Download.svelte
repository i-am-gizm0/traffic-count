<script lang="ts">
    import { createEventDispatcher, onMount } from "svelte";
    const dispatch = createEventDispatcher();


    export let url: string;
    export let name: string;

    let link: HTMLAnchorElement;

    onMount(()=>{
        link.click();
        setTimeout(()=>{
            dispatch('downloaded');
        }, 100);
    });
</script>

<!-- svelte-ignore a11y-missing-content -->
<a href={url} style="display: none;" download={name} bind:this={link} />
