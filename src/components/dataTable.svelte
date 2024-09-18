<script>
  import { onMount } from "svelte";

  export let columns = [];
  export let apiUrl = "";
  export let title = "";
  export let transformData = (data) => data; // Default to identity function

  let data = [];
  let error = "";
  let isTableVisible = false;

  onMount(async () => {
    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error("Network response was not ok.");
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

<div>
  <h1>{title}</h1>

  <button on:click={toggleTableVisibility}>
    {isTableVisible ? "Hide Table" : "View Table"}
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
</div>

<style>
  div {
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);

    background-color: white;
    padding: 4px;
    margin-bottom: 20px;
    border-radius: 20px;
  }
  /* Heading Style */
  h1 {
    font-size: 32px;
    font-weight: 700;
    color: #333;
    margin-bottom: 20px;
    text-align: center;
  }

  /* Button Style */
  button {
    padding: 10px 20px;
    background-color: #3498db; /* Primary color */
    color: #ffffff;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    font-size: 16px;
    transition:
      background-color 0.3s ease,
      transform 0.3s ease;
    margin: 20px 0;
  }

  button:hover {
    background-color: #2980b9; /* Darker shade on hover */
    transform: translateY(-2px); /* Slight lift effect */
  }

  button:active {
    background-color: #1f78b4; /* Even darker shade on click */
    transform: translateY(0); /* Remove lift effect on click */
  }

  /* Error Message Style */
  p {
    color: #e74c3c; /* Red for errors */
    font-size: 18px;
    text-align: center;
    margin: 20px 0;
  }

  /* Table Style */
  table {
    width: 100%;
    border-collapse: collapse;
    margin: 20px auto;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    overflow: hidden;
  }

  thead {
    background-color: #3498db; /* Primary color for header */
    color: #ffffff;
  }

  th,
  td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
  }

  th {
    font-weight: 600;
  }

  tbody tr:nth-child(odd) {
    background-color: #f9f9f9;
  }

  tbody tr:hover {
    background-color: #f1f1f1; /* Hover effect for rows */
  }

  /* Responsive Design */
  @media (max-width: 768px) {
    h1 {
      font-size: 24px;
      margin-bottom: 15px;
    }

    button {
      padding: 8px 16px;
      font-size: 14px;
    }

    table {
      font-size: 14px;
    }

    th,
    td {
      padding: 10px;
    }
  }
</style>
