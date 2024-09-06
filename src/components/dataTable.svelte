<script>
  import { onMount } from 'svelte';

  export let columns = [];
  export let apiUrl = '';
  export let title = '';
  export let transformData = (data) => data; // Default to identity function

  let data = [];
  let error = '';
  let isTableVisible = false;

  onMount(async () => {
    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error('Network response was not ok.');
      let result = await response.json();

      // Apply transformData if provided
      data = transformData(result);
    } catch (e) {
      error = e.message;
    }
  });

  function toggleTableVisibility() {
    isTableVisible = !isTableVisible;
  }
</script>

<h1>{title}</h1>

<button on:click={toggleTableVisibility}>
  {isTableVisible ? 'Hide Table' : 'View Table'}
</button>

{#if error}
  <p>Error: {error}</p>
{:else if isTableVisible}
  <table>
    <thead>
      <tr>
        {#each columns as column}
          <th>{column}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each data as row}
        <tr>
          {#each columns as column}
            <td>{row[column]}</td>
          {/each}
        </tr>
      {/each}
    </tbody>
  </table>
{/if}

<style>
  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 10px;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
  }
  th {
    background-color: #f2f2f2;
    text-align: left;
  }
  button {
    margin: 10px 0;
    padding: 8px 12px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  button:hover {
    background-color: #0056b3;
  }
</style>
