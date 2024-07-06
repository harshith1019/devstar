<script>
    import { onMount } from 'svelte';

    let imgSrc = 'path/to/your/image.jpg';
    let annotations = [];

    onMount(() => {
        const canvas = document.getElementById('image-canvas');
        const ctx = canvas.getContext('2d');
        const img = new Image();
        img.src = imgSrc;
        img.onload = () => {
            ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
        };
    });

    function addAnnotation(event) {
        annotations.push({ x: event.clientX, y: event.clientY });
    }
</script>

<style>
    .annotation {
        position: absolute;
        transform: translate(-50%, -50%);
        background: red;
        width: 10px;
        height: 10px;
        border-radius: 50%;
    }
</style>

<canvas id="image-canvas" width="800" height="600" on:click={addAnnotation}></canvas>

{#each annotations as { x, y }}
    <div class="annotation" style="left: {x}px; top: {y}px;"></div>
{/each}
