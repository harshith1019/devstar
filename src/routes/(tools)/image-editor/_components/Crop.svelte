<script>
  import { onMount } from 'svelte';
  export let image;

  let canvas;
  let context;
  let img = new Image();
  let originalImage = new Image();
  let downloadLink;
  let isDragging = false;
  let cropStartX = 0, cropStartY = 0, cropWidth = 0, cropHeight = 0;
  let showCropRect = false;
  let cropShape = 'rectangle'; // Default crop shape
  let rotation = 0;

  const draw = () => {
    // Calculate the new canvas dimensions based on the rotation
    const angle = rotation % 180;
    const radians = rotation * Math.PI / 180;
    const newWidth = Math.abs(img.width * Math.cos(radians)) + Math.abs(img.height * Math.sin(radians));
    const newHeight = Math.abs(img.height * Math.cos(radians)) + Math.abs(img.width * Math.sin(radians));

    canvas.width = newWidth;
    canvas.height = newHeight;
    context.clearRect(0, 0, canvas.width, canvas.height);
    context.save();
    context.translate(canvas.width / 2, canvas.height / 2);
    context.rotate(rotation * Math.PI / 180);
    context.translate(-img.width / 2, -img.height / 2);
    context.drawImage(img, 0, 0);
    context.restore();

    if (showCropRect) {
      context.strokeStyle = 'red';
      context.lineWidth = 2;
      if (cropShape === 'rectangle') {
        context.strokeRect(cropStartX, cropStartY, cropWidth, cropHeight);
      } else if (cropShape === 'circle') {
        context.beginPath();
        context.ellipse(
          cropStartX + cropWidth / 2, cropStartY + cropHeight / 2,
          Math.abs(cropWidth) / 2, Math.abs(cropHeight) / 2,
          0, 0, 2 * Math.PI
        );
        context.stroke();
      }
    }
  };

  const startDragging = (event) => {
    const rect = canvas.getBoundingClientRect();
    cropStartX = event.clientX - rect.left;
    cropStartY = event.clientY - rect.top;
    cropWidth = 0;
    cropHeight = 0;
    isDragging = true;
    showCropRect = true;
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
    draw();
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
    if (cropShape === 'circle') {
      const circleCanvas = document.createElement('canvas');
      const circleContext = circleCanvas.getContext('2d');
      circleCanvas.width = cropWidth;
      circleCanvas.height = cropHeight;
      circleContext.beginPath();
      circleContext.ellipse(
        cropWidth / 2, cropHeight / 2,
        cropWidth / 2, cropHeight / 2,
        0, 0, 2 * Math.PI
      );
      circleContext.clip();
      circleContext.drawImage(croppedCanvas, 0, 0);
      img.src = circleCanvas.toDataURL();
    } else {
      img.src = croppedCanvas.toDataURL();
    }
    img.onload = () => {
      cropStartX = 0;
      cropStartY = 0;
      cropWidth = 0;
      cropHeight = 0;
      showCropRect = false;
      draw();
    };
  };

  const undoCrop = () => {
    img.src = originalImage.src;
    img.onload = () => {
      draw();
    };
  };

  const rotateImage = () => {
    rotation = (rotation + 90) % 360;
    draw();
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
    originalImage.crossOrigin = 'anonymous';
    originalImage.src = image;
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
  .buttons {
    display: flex;
    /* flex-direction: column; */
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    gap: 1rem;
  }
  button {
    padding: 10px 20px;
    margin: 5px;
    border: none;
    border-radius: 5px;
    background-color: #4CAF50;
    color: white;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  button:hover {
    background-color: #45a049;
  }

  button:active {
    background-color: #397d3a;
  }

  .crop-btn {
    background-color: #FF5722;
  }

  .crop-btn:hover {
    background-color: #e64a19;
  }

  .crop-btn:active {
    background-color: #d84315;
  }

  .download-btn {
    background-color: #008CBA;
  }

  .download-btn:hover {
    background-color: #007bb5;
  }

  .download-btn:active {
    background-color: #006a99;
  }

  .rotate-btn {
    background-color: #9C27B0;
  }

  .rotate-btn:hover {
    background-color: #8E24AA;
  }

  .rotate-btn:active {
    background-color: #7B1FA2;
  }

  .shaped-crop-btn {
    background-color: #FF9800;
  }

  .shaped-crop-btn:hover {
    background-color: #FB8C00;
  }

  .shaped-crop-btn:active {
    background-color: #F57C00;
  }
</style>

<div class="image-editor">
  <canvas id="imageCanvas"
          on:mousedown="{startDragging}"
          on:mousemove="{drag}"
          on:mouseup="{stopDragging}"
          on:mouseleave="{stopDragging}"></canvas>
  <div class='buttons'>
    <button class="crop-btn" on:click="{cropImage}">Crop</button>
    <button on:click="{undoCrop}">Undo Crop</button>
    <button on:click="{() => flipImage('horizontal')}">Horizontal Flip</button>
    <button on:click="{() => flipImage('vertical')}">Vertical Flip</button>
    <button class='rotate-btn' on:click="{rotateImage}">Rotate Image</button>
    <button class='shaped-crop-btn' on:click="{() => cropShape = 'rectangle'}">Rectangle Crop</button>
    <button class='shaped-crop-btn' on:click="{() => cropShape = 'circle'}">Circle Crop</button>
    <button class='download-btn' on:click="{downloadImage}">Download Image</button>
  </div>
</div>
