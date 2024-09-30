<script lang="ts">
    import '../app.css';
</script>

<svelte:head>
  <title>ThirdSpace</title>
  <!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-2YJQSF6DB4"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-2YJQSF6DB4');
</script>
</svelte:head>  
<main class="bg-indigo">
  <slot></slot>
</main>