<script>
  import { onMount } from 'svelte';

  let canvas, ctx;
  let image = new Image();
  let stickers = [
    "https://twemoji.maxcdn.com/v/latest/72x72/1f600.png", // Grinning Face
    "https://twemoji.maxcdn.com/v/latest/72x72/1f923.png", // Rolling on the Floor Laughing
    "https://twemoji.maxcdn.com/v/latest/72x72/1f602.png", // Face with Tears of Joy
    "https://twemoji.maxcdn.com/v/latest/72x72/1f609.png", // Winking Face
    "https://twemoji.maxcdn.com/v/latest/72x72/1f44d.png",  // Thumbs Up
    "https://twemoji.maxcdn.com/v/latest/72x72/1f618.png", // Face Blowing a Kiss
    "https://twemoji.maxcdn.com/v/latest/72x72/1f61c.png", // Winking Face with Tongue
    "https://twemoji.maxcdn.com/v/latest/72x72/1f60d.png", // Smiling Face with Heart-Eyes
    "https://twemoji.maxcdn.com/v/latest/72x72/1f929.png", // Star-Struck
  ];

  let frames = [
    "https://cdn.pixabay.com/photo/2017/01/12/01/54/frame-1973496_960_720.png",
    "https://cdn.pixabay.com/photo/2016/11/05/20/50/decorative-1801460_640.png",
    "https://cdn.pixabay.com/photo/2013/07/12/17/12/black-151767_1280.png",
    "https://cdn.pixabay.com/photo/2017/01/23/03/21/blue-2001488_640.png",
    "https://cdn.pixabay.com/photo/2020/05/04/20/36/frame-5130680_1280.png",
    "https://cdn.pixabay.com/photo/2024/05/07/20/01/frame-8746623_1280.png",
    "https://cdn.pixabay.com/photo/2013/07/12/17/12/border-151768_1280.png",
    "https://cdn.pixabay.com/photo/2021/11/26/19/07/pine-needles-6826441_960_720.png",
    "https://cdn.pixabay.com/photo/2020/09/09/18/16/frame-5558390_1280.png",
    "https://cdn.pixabay.com/photo/2019/10/25/12/53/gears-4576932_960_720.png",
    "https://cdn.pixabay.com/photo/2020/08/01/14/56/edge-5455565_1280.png",
    "https://cdn.pixabay.com/photo/2014/04/07/18/44/border-318820_1280.png"
  ];

  let addedStickers = [];
  let addedFrame = null;
  let showStickers = false;
  let showFrames = false;
  let draggingSticker = null;
  let selectedSticker = null;
  let draggingFrame = null;
  let selectedFrame = null;
  let hoveredSticker = null;
  let hoveredFrame = null;
  let offsetX, offsetY;

  const loadImage = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();
    reader.onload = (e) => {
      image.src = e.target.result;
    };
    reader.readAsDataURL(file);
  };

  const addSticker = (stickerSrc) => {
    const sticker = new Image();
    sticker.src = stickerSrc;
    sticker.onload = () => {
      addedStickers.push({
        image: sticker,
        x: (canvas.width - sticker.width) / 2,
        y: (canvas.height - sticker.height) / 2,
        scale: 1
      });
      draw();
    };
  };

  const addFrame = (frameSrc) => {
    const frame = new Image();
    frame.src = frameSrc;
    frame.onload = () => {
      addedFrame = {
        image: frame,
        x: 0,
        y: 0,
        scale: canvas.width / frame.width
      };
      draw();
    };
  };

  const zoomInSticker = () => {
    if (selectedSticker) {
      selectedSticker.scale += 0.1;
      draw();
    }
  };

  const zoomOutSticker = () => {
    if (selectedSticker) {
      selectedSticker.scale = Math.max(0.1, selectedSticker.scale - 0.1);
      draw();
    }
  };

  const zoomInFrame = () => {
    if (selectedFrame) {
      selectedFrame.scale += 0.1;
      draw();
    }
  };

  const zoomOutFrame = () => {
    if (selectedFrame) {
      selectedFrame.scale = Math.max(0.1, selectedFrame.scale - 0.1);
      draw();
    }
  };

  const draw = () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(image, 0, 0, canvas.width, canvas.height);
    if (addedFrame) {
      const width = addedFrame.image.width * addedFrame.scale;
      const height = addedFrame.image.height * addedFrame.scale;
      ctx.drawImage(addedFrame.image, addedFrame.x, addedFrame.y, width, height);
      if (hoveredFrame === addedFrame) {
        ctx.strokeStyle = 'red';
        ctx.lineWidth = 2;
        ctx.strokeRect(addedFrame.x, addedFrame.y, width, height);
      }
    }
    addedStickers.forEach(sticker => {
      const width = sticker.image.width * sticker.scale;
      const height = sticker.image.height * sticker.scale;
      ctx.drawImage(sticker.image, sticker.x, sticker.y, width, height);
      if (hoveredSticker === sticker) {
        ctx.fillStyle = 'gray';
        ctx.fillRect(sticker.x + width - 10, sticker.y - 10, 20, 20);
        ctx.fillStyle = 'black';
        ctx.fillText('X', sticker.x + width - 4, sticker.y + 3);
      }
    });
  };

  const handleMouseDown = (event) => {
    const mouseX = event.offsetX;
    const mouseY = event.offsetY;
    draggingSticker = addedStickers.find(sticker => {
      const width = sticker.image.width * sticker.scale;
      const height = sticker.image.height * sticker.scale;
      return (
        mouseX >= sticker.x &&
        mouseX <= sticker.x + width &&
        mouseY >= sticker.y &&
        mouseY <= sticker.y + height
      );
    });
    if (addedFrame) {
      const width = addedFrame.image.width * addedFrame.scale;
      const height = addedFrame.image.height * addedFrame.scale;
      if (
        mouseX >= addedFrame.x &&
        mouseX <= addedFrame.x + width &&
        mouseY >= addedFrame.y &&
        mouseY <= addedFrame.y + height
      ) {
        draggingFrame = addedFrame;
        selectedFrame = addedFrame;
        offsetX = mouseX - addedFrame.x;
        offsetY = mouseY - addedFrame.y;
      }
    }
    if (draggingSticker) {
      selectedSticker = draggingSticker;
      offsetX = mouseX - draggingSticker.x;
      offsetY = mouseY - draggingSticker.y;
    }
  };

  const handleMouseMove = (event) => {
    const mouseX = event.offsetX;
    const mouseY = event.offsetY;
    hoveredSticker = addedStickers.find(sticker => {
      const width = sticker.image.width * sticker.scale;
      const height = sticker.image.height * sticker.scale;
      return (
        mouseX >= sticker.x &&
        mouseX <= sticker.x + width &&
        mouseY >= sticker.y &&
        mouseY <= sticker.y + height
      );
    });
    if (addedFrame) {
      const width = addedFrame.image.width * addedFrame.scale;
      const height = addedFrame.image.height * addedFrame.scale;
      if (
        mouseX >= addedFrame.x &&
        mouseX <= addedFrame.x + width &&
        mouseY >= addedFrame.y &&
        mouseY <= addedFrame.y + height
      ) {
        hoveredFrame = addedFrame;
      } else {
        hoveredFrame = null;
      }
    }
    if (draggingSticker) {
      draggingSticker.x = mouseX - offsetX;
      draggingSticker.y = mouseY - offsetY;
    } else if (draggingFrame) {
      draggingFrame.x = mouseX - offsetX;
      draggingFrame.y = mouseY - offsetY;
    }
    draw();
  };

  const handleMouseUp = () => {
    draggingSticker = null;
    draggingFrame = null;
  };

  const handleClickOutside = (event) => {
    if (!event.target.closest('.stickers-container') && !event.target.closest('.add-stickers-btn')) {
      showStickers = false;
    }
  };

  const handleDeleteSticker = (event) => {
    const mouseX = event.offsetX;
    const mouseY = event.offsetY;
    addedStickers = addedStickers.filter(sticker => {
      const width = sticker.image.width * sticker.scale;
      const height = sticker.image.height * sticker.scale;
      const withinX = mouseX >= sticker.x + width - 10 && mouseX <= sticker.x + width + 10;
      const withinY = mouseY >= sticker.y - 10 && mouseY <= sticker.y + 10;
      return !(withinX && withinY);
    });
    draw();
  };

  const handleDeleteFrame = (event) => {
    const mouseX = event.offsetX;
    const mouseY = event.offsetY;
    if (addedFrame) {
      const width = addedFrame.image.width * addedFrame.scale;
      const height = addedFrame.image.height * addedFrame.scale;
      const withinX = mouseX >= addedFrame.x + width - 10 && mouseX <= addedFrame.x + width + 10;
      const withinY = mouseY >= addedFrame.y - 10 && mouseY <= addedFrame.y + 10;
      if (withinX && withinY) {
        addedFrame = null;
        draw();
      }
    }
  };

  onMount(() => {
    canvas = document.getElementById('canvas');
    ctx = canvas.getContext('2d');
    canvas.width = 600;
    canvas.height = 450;
    image.onload = draw;
    canvas.addEventListener('mousedown', handleMouseDown);
    canvas.addEventListener('mousemove', handleMouseMove);
    canvas.addEventListener('mouseup', handleMouseUp);
    canvas.addEventListener('click', handleDeleteSticker);
    canvas.addEventListener('click', handleDeleteFrame);
    document.addEventListener('click', handleClickOutside);
    return () => {
      canvas.removeEventListener('mousedown', handleMouseDown);
      canvas.removeEventListener('mousemove', handleMouseMove);
      canvas.removeEventListener('mouseup', handleMouseUp);
      canvas.removeEventListener('click', handleDeleteSticker);
      canvas.removeEventListener('click', handleDeleteFrame);
      document.removeEventListener('click', handleClickOutside);
    };
  });
</script>

<div class="flex flex-col items-center bg-gray-700 p-10 rounded-lg shadow-lg">
  <input type="file" accept="image/*" on:change={loadImage} class="mb-5 border border-gray-600 rounded"/>
  
  <div class="flex flex-row items-center space-x-2 mb-2">
    <div class="relative">
      <button 
        class="p-2 bg-gray-600 text-white rounded add-stickers-btn" 
        on:click={() => showStickers = !showStickers}
      >
        <img src="https://img.icons8.com/?size=100&id=ofh2E0Ym0IZ7&format=png&color=000000" alt="Add Stickers" class="w-6 h-6" /> 
      </button>
      {#if showStickers}
        <div class="absolute left-0 mt-8 p-2 bg-white rounded-lg shadow-md z-10 flex space-x-2 overflow-x-auto w-80">
          {#each stickers as sticker}
            <img 
              src={sticker} 
              alt="Sticker" 
              class="w-10 h-10 cursor-pointer border border-gray-400 rounded" 
              on:click={() => { addSticker(sticker); showStickers = false; }}
            />
          {/each}
        </div>
      {/if}
    </div>
    
    <div class="relative">
      <!-- <button 
        class="p-2 bg-gray-600 text-white rounded add-frames-btn" 
        on:click={() => showFrames = !showFrames}
      >
        <img src="https://img.icons8.com/ios-filled/50/000000/frame.png" alt="Add Frames" class="w-6 h-6" /> 
      </button> -->

      <button on:click={zoomInSticker} class="p-2 bg-gray-600 text-white rounded">
        <img src="https://img.icons8.com/ios-filled/50/000000/zoom-in.png" alt="Zoom In" class="w-6 h-6" />
      </button>
      <button on:click={zoomOutSticker} class="p-2 bg-gray-600 text-white rounded">
        <img src="https://img.icons8.com/ios-filled/50/000000/zoom-out.png" alt="Zoom Out" class="w-6 h-6" />
      </button>

      {#if showFrames}
        <div class="absolute left-0 mt-8 p-2 bg-white rounded-lg shadow-md z-10 flex space-x-2 overflow-x-auto w-80">
          {#each frames as frame}
            <img  src={frame} alt="Frame" class="w-10 h-10 cursor-pointer border border-gray-400 rounded" 
              on:click={() => { addFrame(frame); showFrames = false; }}
            />
          {/each}
        </div>
      {/if}
    </div>
  </div>

  <div class="flex flex-row items-center space-x-2">
    <!-- <button on:click={zoomInSticker} class="p-2 bg-gray-600 text-white rounded">
      <img src="https://img.icons8.com/ios-filled/50/000000/zoom-in.png" alt="Zoom In" class="w-6 h-6" />
    </button>
    <button on:click={zoomOutSticker} class="p-2 bg-gray-600 text-white rounded">
      <img src="https://img.icons8.com/ios-filled/50/000000/zoom-out.png" alt="Zoom Out" class="w-6 h-6" />
    </button> -->

    <button 
        class="p-2 bg-gray-600 text-white rounded add-frames-btn" 
        on:click={() => showFrames = !showFrames}
      >
        <img src="https://img.icons8.com/ios-filled/50/000000/frame.png" alt="Add Frames" class="w-6 h-6" /> 
      </button>

    <button on:click={zoomInFrame} class="p-2 bg-gray-600 text-white rounded">
      <img src="https://img.icons8.com/ios-filled/50/000000/zoom-in.png" alt="Zoom In" class="w-6 h-6" />
    </button>
    <button on:click={zoomOutFrame} class="p-2 bg-gray-600 text-white rounded">
      <img src="https://img.icons8.com/ios-filled/50/000000/zoom-out.png" alt="Zoom Out" class="w-6 h-6" />
    </button>
  </div>

  <canvas id="canvas" class="border border-gray-600 rounded mt-4"></canvas>
</div>
