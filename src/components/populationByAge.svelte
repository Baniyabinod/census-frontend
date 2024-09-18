<script>
  import { onMount } from "svelte";

  // Props for dynamic input
  export let apiUrl = ""; // API endpoint passed as a prop
  export let dataMap = {}; // Key-value pairs for mapping (e.g., gender map)
  export let labelField = ""; // The field in the stats for the label (e.g., 'aaa')
  export let valueField = ""; // The field in the stats for the value (e.g., 'Number_of_Persons')

  let minAge = 20;
  let maxAge = 30;
  let stats = [];
  let error = "";
  let loading = false; // State for loading

  // Function to fetch data from the backend
  async function fetchPopulationByAge() {
    loading = true; // Start loading
    try {
      // Construct the URL with age range parameters
      const url = `${apiUrl}?min_age=${minAge}&max_age=${maxAge}`;
      console.log(`Fetching data from: ${url}`); // Log the URL for debugging

      const response = await fetch(url);
      if (!response.ok) throw new Error("Network response was not ok.");

      // Parse the response data
      stats = await response.json(); // Update the stats array, triggers UI update
      error = "";
    } catch (e) {
      console.error("Fetch error:", e.message); // Log error
      error = e.message;
    } finally {
      loading = false; // Stop loading
    }
  }

  // Trigger fetch on component mount
  onMount(fetchPopulationByAge);
</script>

<div class="population-by-age">
  <h2>Population by Age Group</h2>

  <!-- Input fields for selecting age range -->
  <div class="input-container">
    <label for="min-age">Min Age:</label>
    <input type="number" id="min-age" bind:value={minAge} min="0" />

    <label for="max-age">Max Age:</label>
    <input type="number" id="max-age" bind:value={maxAge} min="0" />

    <button on:click={fetchPopulationByAge} disabled={loading}>
      {#if loading}
        <span class="loader"></span> Loading...
      {:else}
        Fetch Population
      {/if}
    </button>
  </div>

  {#if error}
    <p class="error-message">Error: {error}</p>
  {:else if stats.length === 0}
    <p>No data available for the selected age range.</p>
  {:else}
    <div class="card-container">
      {#each stats as item}
        <div class="card">
          <!-- Dynamic label mapping based on dataMap and labelField -->
          <h3>{dataMap[item[labelField]] || "Unknown"}</h3>

          <!-- Dynamic value display based on valueField -->
          <p>{item[valueField]}</p>
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  .population-by-age {
    margin: 40px auto;
    max-width: 800px;
    padding: 6px;
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
    text-align: center;
  }

  .population-by-age h2 {
    font-size: 24px;
    font-weight: 700;
    color: #333;
    margin-bottom: 20px;
  }

  .input-container {
    display: flex;
    gap: 15px;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;
  }

  .input-container label {
    font-size: 16px;
    font-weight: 500;
    color: #333;
  }

  .input-container input {
    width: 80px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 6px;
    font-size: 16px;
    outline: none;
    transition: border-color 0.3s ease;
  }

  .input-container input:focus {
    border-color: #3498db; /* Primary color on focus */
    box-shadow: 0 0 4px rgba(0, 0, 0, 0.2);
  }

  .input-container button {
    position: relative;
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
    overflow: hidden; /* Ensure loader is contained */
  }

  .input-container button:hover {
    background-color: #2980b9; /* Darker shade on hover */
    transform: translateY(-2px); /* Subtle lift effect */
  }

  .input-container button:active {
    background-color: #1f78b4; /* Even darker shade on click */
    transform: translateY(0); /* Remove lift effect on click */
  }

  .input-container button:disabled {
    background-color: #7f8c8d; /* Gray color when disabled */
    cursor: not-allowed;
  }

  .loader {
    border: 4px solid rgba(0, 0, 0, 0.1); /* Light border */
    border-radius: 50%;
    border-top: 4px solid #3498db; /* Primary color for loader */
    width: 24px;
    height: 24px;
    animation: spin 1s linear infinite;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

  @keyframes spin {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }

  .error-message {
    color: #e74c3c; /* Red for error messages */
    font-size: 16px;
    margin: 20px 0;
  }

  .card-container {
    display: flex;
    gap: 20px;
    justify-content: center;
    flex-wrap: wrap;
  }

  .card {
    padding: 20px;
    background-color: #ffffff;
    border-radius: 10px;
    text-align: center;
    width: 300px;
    height: 100px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transition:
      box-shadow 0.3s ease,
      transform 0.3s ease;
  }

  .card:hover {
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
    transform: translateY(-4px);
  }

  .card h3 {
    margin: 0 0 10px 0;
    color: #3498db; /* Primary color for titles */
    font-size: 18px;
    font-weight: 600;
  }

  .card p {
    margin: 0;
    font-size: 24px;
    font-weight: 700;
    color: #333;
  }
</style>
