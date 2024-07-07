<script>
    import { onMount } from 'svelte';
    import { saveAs } from 'file-saver'; // Make sure you have installed file-saver: npm install file-saver
  
    let annotations = []; // For storing all annotations
    let activeTool = 'drawLow'; // Default tool
    let drawingThickness = 'low'; // Default drawing thickness
    let drawColor = '#000000'; // Default draw color
    let imageSrc = null; // For storing the uploaded image source
    let textColor = '#000000'; // Default text color
    let zoomLevel = 1.0; // Initial zoom level
    let shapeType = 'rectangle'; // Default shape type
  
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
          // Wait for the second click to define the end coordinates for shapes
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
  
    function finishShape(event) {
      const { offsetX, offsetY } = event;
      const endX = offsetX / zoomLevel;
      const endY = offsetY / zoomLevel;
  
      annotations.push({
        type: 'shape',
        shapeType,
        startX: annotations[annotations.length - 1].startX,
        startY: annotations[annotations.length - 1].startY,
        endX,
        endY,
        color: drawColor
      });
  
      redrawCanvas();
    }
  
    function eraseAnnotation(x, y) {
      annotations = annotations.filter(annotation => {
        if (annotation.type === 'draw'||annotation.type === 'text' || annotation.type === 'shape' || annotation.type === 'highlight') {
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
      redrawCanvas(); // Redraw canvas after clearing annotations
    }
  
    function exportAnnotations() {
      const jsonAnnotations = JSON.stringify(annotations, null, 2);
      const blob = new Blob([jsonAnnotations], { type: 'application/json' });
      saveAs(blob, 'annotations.json');
    }
  
    function selectTool(tool) {
    activeTool = tool;
    if (tool === 'shape') {
      shapeType = 'rectangle'; // Default shape type
  
      // Update shapeType options for selection
      const shapeTypeSelect = document.getElementById('shape-type-select');
      shapeTypeSelect.style.display = 'inline-block'; // Display shape type selector
      shapeTypeSelect.addEventListener('change', (event) => {
        switch (event.target.value) {
          case 'rectangle':
          case 'circle':
          case 'arrow':
          case 'line':
            shapeType = event.target.value;
            break;
          default:
            break;
        }
      });
    }
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
  
    let isDrawing = false;
    let lastX = 0;
    let lastY = 0;
  
    // Event handlers for drawing interaction
    canvas.addEventListener('mousedown', startDrawing);
    canvas.addEventListener('mouseup', stopDrawing);
    canvas.addEventListener('mousemove', draw);
    canvas.addEventListener('mouseleave', stopDrawing);
  
    function startDrawing(event) {
      isDrawing = true;
      const { offsetX, offsetY } = event;
      lastX = offsetX;
      lastY = offsetY;
      ctx.beginPath();
      ctx.moveTo(offsetX * zoomLevel, offsetY * zoomLevel);
    }
  
    function draw(event) {
      if (!isDrawing) return;
      const { offsetX, offsetY } = event;
      ctx.lineTo(offsetX * zoomLevel, offsetY * zoomLevel);
      ctx.stroke();
      lastX = offsetX;
      lastY = offsetY;
    }
  
    function stopDrawing() {
      isDrawing = false;
      ctx.closePath();
    }
  
    // Draw existing points
    annotation.points.forEach((point, index) => {
      if (index === 0) {
        ctx.beginPath();
        ctx.moveTo(point.x * zoomLevel, point.y * zoomLevel);
      } else {
        ctx.lineTo(point.x * zoomLevel, point.y * zoomLevel);
        ctx.stroke();
      }
    });
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
        ctx.strokeRect(annotation.startX * zoomLevel, annotation.startY * zoomLevel, (annotation.endX - annotation.startX) * zoomLevel, (annotation.endY - annotation.startY) * zoomLevel);
        break;
      case 'circle':
      const radius = Math.sqrt(Math.pow(annotation.endX - annotation.startX, 2) + Math.pow(annotation.endY - annotation.startY, 2));
            ctx.beginPath();
            ctx.arc(annotation.startX * zoomLevel, annotation.startY * zoomLevel, radius * zoomLevel, 0, 2 * Math.PI);
            ctx.stroke();
            break;
        // Draw circle
      case 'line':
        // Draw line
        ctx.beginPath();
            ctx.moveTo(annotation.startX * zoomLevel, annotation.startY * zoomLevel);
            ctx.lineTo(annotation.endX * zoomLevel, annotation.endY * zoomLevel);
            ctx.stroke();
            break;
      case 'arrow':
        // Draw arrow
        break;
      default:
        break;
    }
  }
  
      function drawHighlight(annotation) {
        ctx.fillStyle = annotation.color;
        ctx.fillRect(annotation.x * zoomLevel, annotation.y * zoomLevel, annotation.width * zoomLevel, annotation.height * zoomLevel);
      }
  
      canvas.addEventListener('mousedown', (event) => {
        const { offsetX, offsetY } = event;
        const x = offsetX / zoomLevel;
        const y = offsetY / zoomLevel;
  
        switch (activeTool) {
          case 'drawLow':
          case 'drawMedium':
          case 'drawHigh':
            const size = toolConfigs[activeTool].size;
            const newPoint = { x, y };
            const lastAnnotation = annotations[annotations.length - 1];
            if (lastAnnotation && lastAnnotation.type === 'draw') {
              lastAnnotation.points.push(newPoint);
            } else {
              annotations.push({
                type: 'draw',
                points: [newPoint],
                color: drawColor,
                size
              });
            }
            break;
          case 'shape':
            annotations.push({
              type: 'shape',
              shapeType,
              startX: x,
              startY: y,
              color: drawColor
            });
            canvas.removeEventListener('mousedown', addAnnotation);
            canvas.addEventListener('mouseup', finishShape);
            break;
          default:
            addAnnotation(event);
            break;
        }
        redrawCanvas();
      });
  
      function redrawCanvas() {
        drawAnnotations();
      }
  
      return () => {
        canvas.removeEventListener('mousedown', addAnnotation);
        canvas.removeEventListener('mouseup', finishShape);
      };
    });
  </script>
  
  <style>
    /* Add your custom styles here */
    #annotation-canvas {
      border: 1px solid #000;
      cursor: crosshair;
      margin-top: 20px;
      display: block;
      margin-left: auto;
      margin-right: auto;
      background-color: aliceblue;
    }
  
    .tools-container {
              position: absolute; /* Position the container absolutely */
              top: 50%; /* Center the container vertically */
              right: 0; /* Align the container to the right side */
              transform: translateY(-50%); /* Adjust the centering */
              display: flex;
              flex-direction: column; /* Arrange tools in a column */
              gap: 10px; /* Space between tools */
              padding: 8px 16px;
              margin: 5px 0;
              border: none;
              border-radius: 4px;
              background-color: #ffffff; /* Button background color */
              box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Light shadow for buttons */
              font-size: 16px;
              color: #333; 
              margin-top: 170px;
              margin-right: 50px;/* Dark text color */
          }
  
          .tool {
              cursor: pointer;
              padding: 10px;
              border: 1px solid #ccc;
              border-radius: 4px;
              transition: background-color 0.3s, transform 0.3s;
              display: flex;
              align-items: center;
              justify-content: center; /* Center text horizontally */
              background-color: aliceblue; /* Button background color */
          }
  /* Center the tools-container in its parent */
  
  
  /* If you want to add some spacing around the tools-container */
  
  
  
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
      background-color: antiquewhite;
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
      background-color: ghostwhite;
    }
  
    .zoom-controls .tool {
      display: block;
      margin-bottom: 5px;
    }
  
    .upload-container {
      position: absolute;
      text-align: center;
      margin-top: 20px;
    }
  
    .color-picker {
      margin-left: 10px;
    }
    .buttons-container{
      display: flex;
    justify-content: center; /* Center horizontally */
    align-items: center; /* Center vertically if needed */
    }
    button {
      padding: 10px 20px;
    margin-right: 10px;
    font-size: 16px;
    cursor: pointer;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: aliceblue;
    transition: background-color 0.3s, transform 0.3s;
    justify-content: center; /* Center horizontally */
    align-items: center;
     /* Center vertically if needed */
    }
  
    button:hover {
      background-color: #f0f0f0;
      transform: scale(1.1);
    }
  
    input[type="color"] {
      width: 27px; /* Adjust width to desired size */
      height: 20px; /* Adjust height to desired size */
      padding-left: 5px;
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
    
    <div class="tool-dropdown">
      <span class="tool" on:click={() => selectTool('shape')}>{toolConfigs.shape.icon} Shape</span>
      <div class="tool-dropdown-content">
        <div class="tool-option" on:click={() => shapeType = 'rectangle'}>Rectangle</div>
        <div class="tool-option" on:click={() => shapeType = 'circle'}>Circle</div>
        <div class="tool-option" on:click={() => shapeType = 'line'}>Line</div>
        <div class="tool-option" on:click={() => shapeType = 'arrow'}>Arrow</div>
      </div>
    </div>
    
    <span class="tool" on:click={() => selectTool('highlight')}>{toolConfigs.highlight.icon} Highlight</span>
    <span class="tool" on:click={() => selectTool('erase')}>{toolConfigs.erase.icon} Erase</span>
  
    <!-- Zoom controls -->
    <div class="zoom-controls">
      <button class="tool" on:click={zoomIn}>Zoom In 🌐</button>
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
  <canvas id="annotation-canvas" width="600" height="400"></canvas>
  