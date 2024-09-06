<script>
  import { onMount } from 'svelte';

  export let apiUrl = '';      // API URL passed as a prop
  export let title = '';       // Title passed as a prop
  export let dataKey = '';     // Key of the data to display

  let data = null;
  let error = '';

  // Fetch data from the given apiUrl when the component mounts
  onMount(async () => {
    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error('Network response was not ok.');
      const result = await response.json();
      data = result[0][dataKey]; // Extract the data using the provided key
    } catch (e) {
      error = e.message;
    }
  });
</script>

<div class="card">
  <div class="card-header">
    <h2>{title}</h2>
  </div>

  {#if error}
    <div class="error-message">
      <p>{error}</p>
    </div>
  {:else}
    <div class="card-body">
      <p class="data">{data}</p>
    </div>
  {/if}
</div>

<style>
  /* Card container */
  .card {
    background-color: #fff;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    text-align: center;
    max-width: 300px;
    margin: 20px auto;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
  }

  .card:hover {
    transform: translateY(-5px);
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
  }

  /* Card header */
  .card-header h2 {
    font-size: 18px;
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 15px;
  }

  /* Error message */
  .error-message p {
    color: #e74c3c;
    font-size: 14px;
  }

  /* Card body with data */
  .card-body {
    margin-top: 10px;
  }

  .data {
    font-size: 40px;
    font-weight: bold;
    color: #3498db;
  }
</style>
