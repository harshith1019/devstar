<script>
    import { onMount } from 'svelte';
  
    export let image;
    export let canvasWidth = 800; // Default canvas width
    export let canvasHeight = 600; // Default canvas height
  
    let canvas;
    let context;
    let img = new Image();
    let originalImage = new Image();
    let downloadLink;
    let history = [];
    let historyIndex = -1;
    let currentFilter = 'none';
    let zoomLevel = 1;
  
    function saveState() {
      if (historyIndex < history.length - 1) {
        history = history.slice(0, historyIndex + 1);
      }
      history.push(canvas.toDataURL());
      historyIndex++;
    }
  
    const draw = () => {
      canvas.width = canvasWidth * zoomLevel;
      canvas.height = canvasHeight * zoomLevel;
      context.clearRect(0, 0, canvas.width, canvas.height);
      context.save();
      context.filter = currentFilter; // Apply the current filter
      context.drawImage(img, 0, 0, canvasWidth * zoomLevel, canvasHeight * zoomLevel);
      context.restore();
    };
  
    const applyFilter = (filter) => {
      currentFilter = filter;
      draw();
      saveState();
    };
  
    const applyPunchFilter = () => {
      currentFilter = 'saturate(200%)';
      draw();
      saveState();
    };
  
    const applyZoomIn = () => {
      zoomLevel += 0.1;
      draw();
      saveState();
    };
  
    const applyZoomOut = () => {
      zoomLevel -= 0.1;
      if (zoomLevel < 0.1) zoomLevel = 0.1; // Limit zoom out to prevent negative scaling
      draw();
      saveState();
    };
  
    const applyHighlight = () => {
      // Implement highlight functionality
      // Example: context.globalCompositeOperation = 'lighter';
      saveState();
    };
  
    const applyShapes = () => {
  
    };
  
    function loadSavedImage() {
      const dataURL = localStorage.getItem('savedImage');
      if (dataURL) {
        const img = new Image();
        img.src = dataURL;
  
        img.onload = () => {
          canvas.width = img.width;
          canvas.height = img.height;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0);
        };
      }
    }
  
    const downloadImage = () => {
      const dataURL = canvas.toDataURL('image/png');
      localStorage.setItem('savedImage', dataURL);
      downloadLink.href = canvas.toDataURL();
      downloadLink.download = 'edited-image.png';
      downloadLink.click();
      saveState();
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
  
    .filter-btn {
      background-color: #FF5722;
    }
  
    .filter-btn:hover {
      background-color: #e64a19;
    }
  
    .filter-btn:active {
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
  
    .punch-btn {
      background-color: #9C27B0;
    }
  
    .punch-btn:hover {
      background-color: #8E24AA;
    }
  
    .punch-btn:active {
      background-color: #7B1FA2;
    }
  
    .zoom-btn {
      background-color: #2196F3;
    }
  
    .zoom-btn:hover {
      background-color: #1e88e5;
    }
  
    .zoom-btn:active {
      background-color: #1976d2;
    }
  
    .highlight-btn {
      background-color: #FFC107;
    }
  
    .highlight-btn:hover {
      background-color: #ffb300;
    }
  
    .highlight-btn:active {
      background-color: #ffa000;
    }
  
    .shapes-btn {
      background-color: #607D8B;
    }
  
    .shapes-btn:hover {
      background-color: #546E7A;
    }
  
    .shapes-btn:active {
      background-color: #455A64;
    }
  </style>
  
  <div class="image-editor">
    <canvas id="imageCanvas"></canvas>
    <div class='buttons'>
      <button on:click="{() => applyFilter('none')}">Reset</button>
      
     
     
      <button class='zoom-btn' on:click="{applyZoomIn}">Zoom In</button>
      <button class='zoom-btn' on:click="{applyZoomOut}">Zoom Out</button>
    
      <button on:click="{loadSavedImage}">Load Saved Image</button>
      <button class='download-btn' on:click="{downloadImage}">Download Image</button>
    </div>
  </div>
  