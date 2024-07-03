<!-- App.svelte -->
<script>
  import { onMount } from 'svelte';
  import { annotateDefaultEditor } from 'pintura.js';

  let editor;
  let imageState;

  onMount(() => {
      const annotateButton = document.querySelector('.annotate-button[data-action="annotate"]');
      const annotateEditor = document.querySelector('.annotate-editor');

      annotateButton.addEventListener('click', () => {
          // Show the annotate editor
          annotateEditor.style.display = '';
          annotateButton.style.display = 'none';

          // Initialize the annotate editor
          editor = annotateDefaultEditor('.annotate-editor', {
              src: './image.jpeg',
              imageState: imageState,
          });

          editor.on('process', (res) => {
              // Update image state
              imageState = res.imageState;

              // Hide the annotate editor
              annotateEditor.style.display = 'none';
              annotateButton.style.display = '';

              // Perform actions with the annotation result if needed
              console.log('Annotation result:', res);
          });
      });

      // Cleanup on component destruction
      return () => {
          if (editor) {
              editor.destroy();
              editor = undefined;
          }
      };
  });
</script>

<style>
  /* styles.css styles can be added here directly */
  /* You can also use Svelte's scoped styles for more isolation */
</style>

<h1>Pintura Image Annotator</h1>

<button class="annotate-button" data-action="annotate">
  Annotate Image
</button>

<div class="annotate-editor" style="display: none; width: 100%; height: 100vh;"></div>
