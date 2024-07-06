<script>
    let annotations = [];
    let text = '';
    let color = '#000000';
    let fontSize = '16px';

    function addAnnotation(event) {
        annotations.push({ x: event.clientX, y: event.clientY, text, color, fontSize });
        text = '';
    }
</script>

<style>
    .annotation {
        position: absolute;
        transform: translate(-50%, -50%);
        background: white;
        border: 1px solid black;
        padding: 2px;
    }
</style>

<div>
    <input type="text" bind:value={text} placeholder="Enter text" />
    <input type="color" bind:value={color} />
    <select bind:value={fontSize}>
        <option value="12px">Small</option>
        <option value="16px">Medium</option>
        <option value="20px">Large</option>
    </select>
    <button on:click|preventDefault={addAnnotation}>Add Text</button>
</div>

{#each annotations as { x, y, text, color, fontSize }}
    <div class="annotation" style="left: {x}px; top: {y}px; color: {color}; font-size: {fontSize};">
        {text}
    </div>
{/each}
