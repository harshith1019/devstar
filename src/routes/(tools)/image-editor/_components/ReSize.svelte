<script>
    import { onMount } from 'svelte';
    export let image;
    
    let canvas;
    let context;
    let img = new Image();
    let newWidth;
    let newHeight;
  
    // Function to draw the original image on the canvas
    const draw = (width = img.width, height = img.height) => {
      canvas.width = width;
      canvas.height = height;
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.drawImage(img, 0, 0, width, height);
    };
  
    // Function to resize the image and return its data URL
    const getResizedImageDataURL = () => {
      const resizedCanvas = document.createElement('canvas');
      const resizedContext = resizedCanvas.getContext('2d');
  
      resizedCanvas.width = newWidth;
      resizedCanvas.height = newHeight;
      resizedContext.drawImage(img, 0, 0, newWidth, newHeight);
  
      return resizedCanvas.toDataURL();
    };
  
    // Function to update new dimensions
    const updateDimensions = () => {
      // Update dimensions if needed
    };
  
    // Function to download the resized image
    const downloadImage = () => {
      const resizedImageDataURL = getResizedImageDataURL();
      const link = document.createElement('a');
      link.href = resizedImageDataURL;
      link.download = 'resized-image.png';
      link.click();
    };
  
    // Function to initialize the image and canvas on component mount
    onMount(() => {
      canvas = document.getElementById('imageCanvas');
      context = canvas.getContext('2d');
      img.crossOrigin = 'anonymous';
      img.src = image;
      img.onload = () => {
        newWidth = img.width;
        newHeight = img.height;
        draw();
      };
    });
  </script>
  
  <style>
    .image-editor {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 1rem;
    }
    canvas {
      border: 1px solid #ccc;
      max-width: 100%;
      height: auto;
    }
    .controls {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 0.5rem;
    }
  </style>
  
  <div class="image-editor">
    <canvas id="imageCanvas"></canvas>
    <div class="controls">
      <label>
        Width:
        <input type="number" bind:value="{newWidth}" on:input="{updateDimensions}">
      </label>
      <label>
        Height:
        <input type="number" bind:value="{newHeight}" on:input="{updateDimensions}">
      </label>
      <button on:click="{() => draw(newWidth, newHeight)}">Resize Image</button>
      <button on:click="{downloadImage}">Download Image</button>
    </div>
  </div>
  