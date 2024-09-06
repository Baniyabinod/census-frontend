<script>
  import { onMount } from 'svelte';

  export let apiUrl = '';         // The API endpoint passed as a prop
  export let title = '';          // Title of the summary card
  export let dataMap = {};        // Key-value pairs for mapping 
  export let labelField = '';     // The field in the stats for the label (e.g., 'abc')
  export let valueField = '';     // The field in the stats for the value (e.g., 'lll')
  
  let stats = [];
  let error = '';

  // Fetch data when component mounts
  onMount(async () => {
    try {
      const response = await fetch(apiUrl);  // Use the API URL passed as a prop
      if (!response.ok) throw new Error('Network response was not ok.');
      stats = await response.json();         // Store the fetched stats
    } catch (e) {
      error = e.message;
    }
  });
</script>

<div class="summary-card">
  <h2>{title}</h2>
  
  {#if error}
    <p>Error: {error}</p>
  {:else}
    <div class="card-container">
      {#each stats as item}
        <div class="card">
          <h3>{dataMap[item[labelField]] || 'Unknown'}</h3>  <!-- Dynamic label mapping -->
          <p>{item[valueField]}</p>                          <!-- Dynamic value display -->
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  .summary-card {
    margin: 20px 0;
  }

  h2 {
    text-align: center;
    margin-bottom: 20px;
  }

  .card-container {
    display: flex;
    gap: 20px;
    justify-content: center;
    flex-wrap: wrap;
  }

  .card {
    padding: 10px;
    background-color: #f5f5f5;
    border-radius: 8px;
    text-align: center;
    width: 150px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  }

  .card h3 {
    margin: 0;
    color: #42A5F5;
  }

  .card p {
    font-size: 18px;
    font-weight: bold;
  }
</style>
