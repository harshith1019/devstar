<script>
  export let data;
</script>

<main>
  <slot />
</main>

<style>
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: #6e111103;
  }
</style>
