<script>
    import { onMount } from 'svelte';
    export let image;
    
    let canvas;
    let context;
    let img = new Image();
    let downloadLink;
    let isDragging = false;
    let cropStartX = 0, cropStartY = 0, cropWidth = 0, cropHeight = 0;
    
    const draw = () => {
      canvas.width = img.width;
      canvas.height = img.height;
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.drawImage(img, 0, 0);
      context.strokeStyle = 'red';
      context.lineWidth = 2;
      context.strokeRect(cropStartX, cropStartY, cropWidth, cropHeight);
    };
    
    const startDragging = (event) => {
      const rect = canvas.getBoundingClientRect();
      cropStartX = event.clientX - rect.left;
      cropStartY = event.clientY - rect.top;
      cropWidth = 0;
      cropHeight = 0;
      isDragging = true;
    };
    
    const drag = (event) => {
      if (isDragging) {
        const rect = canvas.getBoundingClientRect();
        const mouseX = event.clientX - rect.left;
        const mouseY = event.clientY - rect.top;
        cropWidth = mouseX - cropStartX;
        cropHeight = mouseY - cropStartY;
        draw();
      }
    };
    
    const stopDragging = () => {
      isDragging = false;
    };
  
    const cropImage = () => {
      const croppedCanvas = document.createElement('canvas');
      const croppedContext = croppedCanvas.getContext('2d');
      croppedCanvas.width = cropWidth;
      croppedCanvas.height = cropHeight;
      croppedContext.drawImage(
        canvas,
        cropStartX, cropStartY, cropWidth, cropHeight,
        0, 0, cropWidth, cropHeight
      );
      img.src = croppedCanvas.toDataURL();
      img.onload = () => {
        draw();
      };
    };
    
    const flipImage = (direction) => {
      const flipCanvas = document.createElement('canvas');
      const flipContext = flipCanvas.getContext('2d');
    
      flipCanvas.width = img.width;
      flipCanvas.height = img.height;
    
      if (direction === 'horizontal') {
        flipContext.scale(-1, 1); // Flip horizontally
        flipContext.drawImage(img, -img.width, 0);
      } else if (direction === 'vertical') {
        flipContext.scale(1, -1); // Flip vertically
        flipContext.drawImage(img, 0, -img.height);
      }
    
      img.src = flipCanvas.toDataURL();
      img.onload = () => {
        draw();
      };
    };
    
    const downloadImage = () => {
      downloadLink.href = canvas.toDataURL();
      downloadLink.download = 'edited-image.png';
      downloadLink.click();
    };
    
    onMount(() => {
      canvas = document.getElementById('imageCanvas');
      context = canvas.getContext('2d');
      downloadLink = document.createElement('a');
      img.crossOrigin = 'anonymous';
      img.src = image;
      img.onload = () => {
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
  </style>
  
  <div class="image-editor">
    <canvas id="imageCanvas"
            on:mousedown="{startDragging}"
            on:mousemove="{drag}"
            on:mouseup="{stopDragging}"
            on:mouseleave="{stopDragging}"></canvas>
    <button on:click="{cropImage}">Crop</button>
    <button on:click="{() => flipImage('horizontal')}">Horizontal Flip</button>
    <button on:click="{() => flipImage('vertical')}">Vertical Flip</button>
    <button on:click="{downloadImage}">Download Image</button>
  </div>
  