<script>
  import { onMount } from 'svelte';
  import { saveAs } from 'file-saver'; // You will need to install file-saver: npm install file-saver

  let annotations = []; // For storing all annotations
  let activeTool = 'drawLow'; // Default tool
  let drawingThickness = 'low'; // Default drawing thickness
  let drawColor = '#000000'; // Default draw color
  let imageSrc = null; // For storing the uploaded image source
  let textColor = '#000000'; // Default text color
  let zoomLevel = 1.0; // Initial zoom level

  const toolConfigs = {
    drawLow: {
      name: 'Draw (Low)',
      icon: '✏️',
      size: 2
    },
    drawMedium: {
      name: 'Draw (Medium)',
      icon: '✏️',
      size: 4
    },
    drawHigh: {
      name: 'Draw (High)',
      icon: '✏️',
      size: 6
    },
    text: {
      name: 'Text',
      icon: '🔤',
      size: 16,
      fontFamily: 'Arial'
    },
    shape: {
      name: 'Shape',
      icon: '🔶',
      type: 'rectangle'
    },
    highlight: {
      name: 'Highlight',
      icon: '🌟',
      color: 'rgba(255, 255, 0, 0.5)'
    },
    erase: {
      name: 'Eraser',
      icon: '🧽',
      size: 10
    }
  };

  function addAnnotation(event) {
    const { offsetX, offsetY } = event;
    const x = offsetX / zoomLevel;
    const y = offsetY / zoomLevel;

    switch (activeTool) {
      case 'drawLow':
      case 'drawMedium':
      case 'drawHigh':
        annotations.push({
          type: 'draw',
          points: [{ x, y }],
          color: drawColor,
          size: toolConfigs[activeTool].size
        });
        break;
      case 'text':
        const text = prompt('Enter text:');
        if (text) {
          annotations.push({
            type: 'text',
            text,
            x,
            y,
            color: textColor,
            size: toolConfigs.text.size,
            fontFamily: toolConfigs.text.fontFamily
          });
        }
        break;
      case 'shape':
        const shapeType = prompt('Enter shape type (rectangle, circle, line, arrow):');
        if (shapeType && ['rectangle', 'circle', 'line', 'arrow'].includes(shapeType)) {
          annotations.push({
            type: 'shape',
            shapeType,
            x,
            y,
            color: drawColor
          });
        }
        break;
      case 'highlight':
        annotations.push({
          type: 'highlight',
          x,
          y,
          width: 100,
          height: 20,
          color: toolConfigs.highlight.color
        });
        break;
      case 'erase':
        eraseAnnotation(x, y);
        break;
      default:
        break;
    }
  }

  function eraseAnnotation(x, y) {
    annotations = annotations.filter(annotation => {
      if (annotation.type === 'draw') {
        return !annotation.points.some(point => isPointInRange(point, { x, y }, toolConfigs.erase.size));
      } else if (annotation.type === 'text' || annotation.type === 'shape' || annotation.type === 'highlight') {
        return !isPointInRange({ x: annotation.x, y: annotation.y }, { x, y }, toolConfigs.erase.size);
      }
      return true;
    });
  }

  function isPointInRange(point1, point2, range) {
    return Math.abs(point1.x - point2.x) < range && Math.abs(point1.y - point2.y) < range;
  }

  function clearAnnotations() {
    annotations = [];
    drawAnnotations(); // Redraw canvas after clearing annotations
  }

  function exportAnnotations() {
    const jsonAnnotations = JSON.stringify(annotations, null, 2);
    const blob = new Blob([jsonAnnotations], { type: 'application/json' });
    saveAs(blob, 'annotations.json');
  }

  function selectTool(tool) {
    activeTool = tool;
  }

  function selectDrawingThickness(thickness) {
    drawingThickness = thickness;
    switch (thickness) {
      case 'low':
        activeTool = 'drawLow';
        break;
      case 'medium':
        activeTool = 'drawMedium';
        break;
      case 'high':
        activeTool = 'drawHigh';
        break;
      default:
        break;
    }
  }

  function zoomIn() {
    if (zoomLevel < 2.0) {
      zoomLevel += 0.1;
      redrawCanvas();
    }
  }

  function zoomOut() {
    if (zoomLevel > 0.5) {
      zoomLevel -= 0.1;
      redrawCanvas();
    }
  }

  function handleImageUpload(event) {
    const file = event.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        imageSrc = e.target.result;
        redrawCanvas();
      };
      reader.readAsDataURL(file);
    }
  }

  onMount(() => {
    const canvas = document.getElementById('annotation-canvas');
    const ctx = canvas.getContext('2d');

    function drawAnnotations() {
      if (imageSrc) {
        const img = new Image();
        img.onload = () => {
          ctx.clearRect(0, 0, canvas.width, canvas.height);
          ctx.drawImage(img, 0, 0, canvas.width * zoomLevel, canvas.height * zoomLevel);
          annotations.forEach(annotation => {
            switch (annotation.type) {
              case 'draw':
                drawFreehand(annotation);
                break;
              case 'text':
                drawText(annotation);
                break;
              case 'shape':
                drawShape(annotation);
                break;
              case 'highlight':
                drawHighlight(annotation);
                break;
              default:
                break;
            }
          });
        };
        img.src = imageSrc;
      } else {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
      }
    }

    function drawFreehand(annotation) {
      ctx.strokeStyle = annotation.color;
      ctx.lineWidth = annotation.size;
      ctx.lineCap = 'round';
      ctx.beginPath();
      annotation.points.forEach(point => {
        ctx.lineTo(point.x * zoomLevel, point.y * zoomLevel);
      });
      ctx.stroke();
    }

    function drawText(annotation) {
      ctx.font = `${annotation.size * zoomLevel}px ${annotation.fontFamily}`;
      ctx.fillStyle = annotation.color;
      ctx.fillText(annotation.text, annotation.x * zoomLevel, annotation.y * zoomLevel);
    }

    function drawShape(annotation) {
      ctx.strokeStyle = annotation.color;
      ctx.lineWidth = 2;
      switch (annotation.shapeType) {
        case 'rectangle':
          ctx.strokeRect(annotation.x * zoomLevel, annotation.y * zoomLevel, 100 * zoomLevel, 50 * zoomLevel);
          break;
        case 'circle':
          ctx.beginPath();
          ctx.arc(annotation.x * zoomLevel, annotation.y * zoomLevel, 50 * zoomLevel, 0, 2 * Math.PI);
          ctx.stroke();
          break;
        case 'line':
          ctx.beginPath();
          ctx.moveTo(annotation.x * zoomLevel, annotation.y * zoomLevel);
          ctx.lineTo(annotation.x * zoomLevel + 100 * zoomLevel, annotation.y * zoomLevel + 50 * zoomLevel);
          ctx.stroke();
          break;
        case 'arrow':
          const arrowSize = 20; // Example size for arrowhead
          const angle = Math.atan2(annotation.y2 - annotation.y, annotation.x2 - annotation.x);

          // Draw line
          ctx.beginPath();
          ctx.moveTo(annotation.x * zoomLevel, annotation.y * zoomLevel);
          ctx.lineTo(annotation.x2 * zoomLevel, annotation.y2 * zoomLevel);
          ctx.stroke();

          // Draw arrowhead
          ctx.beginPath();
          ctx.moveTo(annotation.x2 * zoomLevel, annotation.y2 * zoomLevel);
          ctx.lineTo(
            annotation.x2 * zoomLevel - arrowSize * Math.cos(angle - Math.PI / 6) * zoomLevel,
            annotation.y2 * zoomLevel - arrowSize * Math.sin(angle - Math.PI / 6) * zoomLevel
          );
          ctx.lineTo(
            annotation.x2 * zoomLevel - arrowSize * Math.cos(angle + Math.PI / 6) * zoomLevel,
            annotation.y2 * zoomLevel - arrowSize * Math.sin(angle + Math.PI / 6) * zoomLevel
          );
          ctx.lineTo(annotation.x2 * zoomLevel, annotation.y2 * zoomLevel);
          ctx.closePath();
          ctx.fill();
          break;
        default:
      }
    }

    function drawHighlight(annotation) {
      ctx.fillStyle = annotation.color;
      ctx.fillRect(annotation.x * zoomLevel, annotation.y * zoomLevel, annotation.width * zoomLevel, annotation.height * zoomLevel);
    }

    canvas.addEventListener('mousedown', function(e) {
      addAnnotation(e);
      drawAnnotations();
    });

    function redrawCanvas() {
      drawAnnotations();
    }
    function redrawCanvas() {
      drawAnnotations();
    }

    $: annotations, redrawCanvas();
    $: imageSrc, redrawCanvas();
  });
</script>

<style>
  .tools-container {
    position: fixed;
    right: 20px;
    top: 50%;
    transform: translateY(-50%);
    display: flex;
    flex-direction: column;
    gap: 10px;
    display: flex;
  }

  .tool {
    cursor: pointer;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    transition: background-color 0.3s, transform 0.3s;
    display: flex;
    align-items: center;
  }

  .tool:hover {
    background-color: #f0f0f0;
    transform: scale(1.1);
  }

  .tool-dropdown {
    position: relative;
    display: inline-block;
  }

  .tool-dropdown-content {
    display: none;
    position: absolute;
    background-color: #f9f9f9;
    min-width: 160px;
    box-shadow: 0px 8px 16px 0px rgba(0, 0, 0, 0.2);
    z-index: 1;
  }

  .tool-dropdown-content .tool-option {
    padding: 12px 16px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .tool-dropdown-content .tool-option:hover {
    background-color: #f1f1f1;
  }

  .tool-dropdown:hover .tool-dropdown-content {
    display: block;
  }

  .zoom-controls {
    margin-top: 10px;
       margin-right: 10px;
    padding: 10px 20px 10px 20px;
    font-size: 16px;
    cursor: pointer;
    border: 1px solid #ccc;
    border-radius: 4px;
    transition: background-color 0.3s, transform 0.3s;
  }

  .zoom-controls .tool {
    display: block;
    margin-bottom: 5px;
  }

  #annotation-canvas {
    border: 1px solid #000;
    cursor: crosshair;
    margin-top: 20px;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }

  .buttons-container {
    text-align: center;
    margin-top: 20px;
  }

  .upload-container {
    text-align: center;
    margin-top: 20px;
  }

  .color-picker {
    margin-left: 10px;
  }

  button {
    padding-top: 10px;
    margin-right: 10px;
    padding: 10px 20px 10px 20px;
    font-size: 16px;
    cursor: pointer;
    border: 1px solid #ccc;
    border-radius: 4px;
    transition: background-color 0.3s, transform 0.3s;
  }

  button:hover {
    background-color: #f0f0f0;
    transform: scale(1.1);
  }

  input[type="color"] {
    width: 27px; /* Adjust width to desired size */
    height: 20px; /* Adjust height to desired size */
    padding-left: 5px;;
    margin-left: 5px;
    vertical-align: middle;
  }
</style>

<div class="tools-container">
  <!-- Dropdown for Draw tool -->
  <div class="tool-dropdown">
    <div class="tool">{toolConfigs.drawLow.icon} Draw
      {#if activeTool.startsWith('draw')}
      <div class="color-picker">
        <label for="draw-color"></label>
        <input type="color" id="draw-color" bind:value={drawColor}>
      </div>
    {/if}
    </div>
    <div class="tool-dropdown-content">
      <div class="tool-option" on:click={() => selectDrawingThickness('low')}>Low</div>
      <div class="tool-option" on:click={() => selectDrawingThickness('medium')}>Medium</div>
      <div class="tool-option" on:click={() => selectDrawingThickness('high')}>High</div>
    </div>
  
  </div>

  <!-- Other tools -->
  <span class="tool" on:click={() => selectTool('text')}>{toolConfigs.text.icon} Text 
    {#if activeTool === 'text'}
    <div class="color-picker">
      <label for="text-color"></label>
      <input type="color" id="text-color" bind:value={textColor}>
    </div>
  {/if}
  </span>
  

  <span class="tool" on:click={() => selectTool('shape')}>{toolConfigs.shape.icon} Shape</span>
  <span class="tool" on:click={() => selectTool('highlight')}>{toolConfigs.highlight.icon} Highlight</span>
  <span class="tool" on:click={() => selectTool('erase')}>{toolConfigs.erase.icon} Erase</span>

  <!-- Draw tool color picker (only shown when Draw tool is selected) -->
  
  <!-- Zoom controls -->
  <div class="zoom-controls">
    <button class="tool" on:click={zoomIn}>Zoom In 🌐   </button>
    <button class="tool" on:click={zoomOut}>Zoom Out 🌍</button>
  </div>
</div>

<!-- Image upload -->
<div class="upload-container">
  <input type="file" accept="image/*" on:change={handleImageUpload}>
</div>

<!-- Clear and Export buttons -->
<div class="buttons-container">
  <button on:click={clearAnnotations}>Clear Annotations</button>
  <button on:click={exportAnnotations}>Export Annotations</button>
</div>

<!-- Canvas for drawing annotations -->
<canvas id="annotation-canvas" width="800" height="400"></canvas>

