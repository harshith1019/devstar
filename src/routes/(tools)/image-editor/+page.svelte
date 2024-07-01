<script>
	import { onMount } from 'svelte';
	import { openDefaultEditor, getEditorDefaults } from '@pqina/pintura';
  
	let editor;
	let imageUrl = 'https://via.placeholder.com/800x600.png'; // Initial image URL
	let previewUrl = imageUrl; // URL for image preview
  
	function handleFileChange(event) {
	  const file = event.target.files[0];
	  if (file) {
		const reader = new FileReader();
		reader.onload = function (e) {
		  previewUrl = e.target.result; // Update preview URL
		  imageUrl = e.target.result;
		  initializeEditor();
		};
		reader.readAsDataURL(file);
	  }
	}
  
	function initializeEditor() {
	  const editorContainer = document.getElementById('editor-container');
	  if (editor) {
		editor.destroy();
	  }
	  const options = {
		...getEditorDefaults(),
		src: imageUrl,
		appendTo: editorContainer,
		tools: ['annotate', 'crop', 'rotate', 'flip', 'filter'], // Ensure 'annotate' is included
	  };
	  editor = openDefaultEditor(options);
	  editor.on('process', (result) => {
		console.log(result.dest);
	  });
	}
  
	onMount(() => {
	  initializeEditor();
	  return () => {
		if (editor) {
		  editor.destroy();
		}
	  };
	});
  
	function annotateImage() {
	  if (editor) {
		editor.setTool('annotate'); // Switch to the annotation tool
	  }
	}
  </script>
  
  <style>
	.container {
	  display: flex;
	  flex-direction: column;
	  justify-content: center;
	  align-items: center;
	  height: 100vh;
	  text-align: center;
	}
  
	.file-input-container {
	  margin-bottom: 20px;
	}
  
	.preview-container img {
	  max-width: 100%;
	  max-height: 400px;
	  margin-top: 10px;
	}
  
	#editor-container {
	  width: 100%;
	  height: 600px;
	  position: relative;
	  display: flex;
	  justify-content: center;
	  align-items: center;
	  margin-top: 20px;
	}
  
	.annotate-button {
	  position: absolute;
	  left: 10px;
	  top: 50%;
	  transform: translateY(-50%);
	  z-index: 10;
	  padding: 10px 20px;
	  background-color: #007bff;
	  color: white;
	  border: none;
	  border-radius: 4px;
	  cursor: pointer;
	}
  
	.annotate-button:hover {
	  background-color: #0056b3;
	}
  </style>
  
  <div class="container">
	<div class="file-input-container">
	  <input type="file" accept="image/*" on:change="{handleFileChange}" />
	</div>
  
	<div class="preview-container">
	  <h2>Selected Image Preview:</h2>
	  <img src="{previewUrl}" alt="Image Preview" />
	</div>
  
	<button class="annotate-button" on:click="{annotateImage}">Annotate</button>
  
	<div id="editor-container"></div>
  </div>
  