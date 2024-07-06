<script>
    let highlights = [];
    let color = '#ffff00';

    function addHighlight(event) {
        highlights.push({ x: event.clientX, y: event.clientY, width: 100, height: 20, color });
    }
</script>

<style>
    .highlight {
        position: absolute;
        background: rgba(255, 255, 0, 0.5);
    }
</style>

<div>
    <input type="color" bind:value={color} />
    <button on:click|preventDefault={addHighlight}>Add Highlight</button>
</div>

{#each highlights as { x, y, width, height, color }}
    <div class="highlight" style="left: {x}px; top: {y}px; width: {width}px; height: {height}px; background: {color};"></div>
{/each}
