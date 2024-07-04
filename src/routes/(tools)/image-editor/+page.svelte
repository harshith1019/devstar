<script>
	import { onMount } from 'svelte';
	import { afterUpdate } from 'svelte';
  
	let imageSrc = '';
	let canvas;
	let context;
	let isDragging = false;
	let cropStartX = 0, cropStartY = 0, cropWidth = 0, cropHeight = 0;
	let offsetX, offsetY;
	let croppedImage = null;
	let downloadLink;
  
	let img; // Declare img here
	
	const loadImage = (event) => {
	  const file = event.target.files[0];
	  const reader = new FileReader();
	  reader.onload = (e) => {
		imageSrc = e.target.result;
		img = new Image(); // Initialize img here
		img.src = imageSrc;
		img.onload = () => {
		  cropWidth = img.width * 0.9;
		  cropHeight = img.height * 0.9;
		  draw();
		};
	  };
	  reader.readAsDataURL(file);
	};
  
	const draw = () => {
	  canvas.width = img.width;
	  canvas.height = img.height;
	  context.clearRect(0, 0, canvas.width, canvas.height);
	  context.drawImage(img, 0, 0);
	  context.strokeStyle = 'red';
	  context.lineWidth = 2;
	  context.strokeRect(cropStartX, cropStartY, cropWidth, cropHeight);
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
	  croppedImage = croppedCanvas.toDataURL();
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
	  img.onload = draw; // Redraw after flipping
	};
  
	const downloadImage = () => {
	  if (croppedImage) {
		downloadLink.href = croppedImage;
		downloadLink.click();
	  }
	};
  
	const startDragging = (event) => {
	  const rect = canvas.getBoundingClientRect();
	  offsetX = event.clientX - rect.left;
	  offsetY = event.clientY - rect.top;
	  if (offsetX > cropStartX && offsetX < cropStartX + cropWidth &&
		  offsetY > cropStartY && offsetY < cropStartY + cropHeight) {
		isDragging = true;
	  }
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
  
	onMount(() => {
	  canvas = document.getElementById('imageCanvas');
	  context = canvas.getContext('2d');
	  downloadLink = document.createElement('a');
	  downloadLink.download = 'cropped-image.png';
	});
  
	// Cleanup and redraw on img change
	afterUpdate(() => {
	  if (img) {
		draw();
	  }
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
	.cropped-image {
	  max-width: 100%;
	  height: auto;
	  border: 1px solid #ccc;
	}
  </style>
  
  <div class="image-editor">
	<input type="file" accept="image/*" on:change="{loadImage}" />
	<canvas id="imageCanvas"
			on:mousedown="{startDragging}"
			on:mousemove="{drag}"
			on:mouseup="{stopDragging}"
			on:mouseleave="{stopDragging}">
	</canvas>
	<button on:click="{cropImage}">Crop</button>
	<button on:click="{() => flipImage('horizontal')}">Horizontal Flip</button>
	<button on:click="{() => flipImage('vertical')}">Vertical Flip</button>
	{#if croppedImage}
	  <div>
		<img class="cropped-image" src="{croppedImage}" alt="Cropped Image" />
		<button on:click="{downloadImage}">Download</button>
	  </div>
	{/if}
  </div>
  