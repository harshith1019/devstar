<script>
    let notes = [];
    let text = '';

    function addNote() {
        notes.push({ x: 100, y: 100, text });
        text = '';
    }
</script>

<style>
    .note {
        position: absolute;
        background: yellow;
        padding: 10px;
        border: 1px solid black;
        resize: both;
        overflow: auto;
    }
</style>

<div>
    <textarea bind:value={text} placeholder="Enter note"></textarea>
    <button on:click={addNote}>Add Note</button>
</div>

{#each notes as { x, y, text }}
    <div class="note" style="left: {x}px; top: {y}px;">
        {text}
    </div>
{/each}
