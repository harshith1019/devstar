<script>
    let drawing = false;
    let points = [];
    let color = '#000000';
    let size = '2px';

    function startDrawing(event) {
        drawing = true;
        points.push({ x: event.clientX, y: event.clientY, color, size });
    }

    function draw(event) {
        if (!drawing) return;
        points.push({ x: event.clientX, y: event.clientY, color, size });
    }

    function stopDrawing() {
        drawing = false;
    }
</script>

<style>
    .canvas {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        cursor: crosshair;
    }
    .line {
        position: absolute;
        background: currentColor;
        width: 1px;
    }
</style>

<div>
    <input type="color" bind:value={color} />
    <select bind:value={size}>
        <option value="2px">Small</option>
        <option value="4px">Medium</option>
        <option value="6px">Large</option>
    </select>
</div>

<div class="canvas" on:mousedown={startDrawing} on:mousemove={draw} on:mouseup={stopDrawing} on:mouseleave={stopDrawing}>
    {#each points as { x, y, color, size }, i}
        {#if i > 0}
            <div class="line" style="left: {points[i-1].x}px; top: {points[i-1].y}px; width: {Math.abs(points[i].x - points[i-1].x)}px; height: {size}; background: {color};"></div>
        {/if}
    {/each}
</div>
