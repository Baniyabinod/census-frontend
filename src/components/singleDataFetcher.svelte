<script>
  import { onMount } from "svelte";

  export let apiUrl = "";
  export let title = ""; // Title passed as a prop
  export let dataKey = ""; // Key of the data to display

  let data = null;
  let error = "";

  // Fetch data from the given apiUrl when the component mounts
  onMount(async () => {
    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error("Network response was not ok.");
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
    background-color: #ffffff; /* White background */
    border-radius: 12px; /* Slightly more rounded corners */
    padding: 24px; /* Increased padding for a spacious feel */
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1); /* Slightly deeper shadow */
    text-align: center; /* Center-align text */
    max-width: 320px; /* Slightly wider card */
    margin: 20px auto; /* Center card horizontally */
    transition:
      transform 0.3s ease-in-out,
      box-shadow 0.3s ease-in-out; /* Smooth transitions */
  }

  /* Hover effect */
  .card:hover {
    transform: translateY(-8px); /* More pronounced lift */
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2); /* Enhanced shadow on hover */
  }

  /* Card header */
  .card-header h2 {
    font-size: 20px; /* Larger font size for better visibility */
    font-weight: 700; /* Bolder font weight */
    color: #34495e; /* Darker shade for better readability */
    margin-bottom: 20px; /* Increased margin for separation */
  }

  /* Error message */
  .error-message {
    background-color: #f8d7da; /* Light red background */
    border: 1px solid #f5c6cb; /* Light red border */
    border-radius: 8px; /* Rounded corners for the error message */
    padding: 12px; /* Padding inside error message */
    margin-bottom: 20px; /* Space below the error message */
  }

  .error-message p {
    color: #e74c3c; /* Bright red for error text */
    font-size: 16px; /* Slightly larger font size */
    font-weight: 500; /* Medium font weight */
    margin: 0; /* Remove default margin */
  }

  /* Card body with data */
  .card-body {
    margin-top: 10px; /* Space above the card body */
  }

  .data {
    font-size: 48px; /* Larger font size for data */
    font-weight: 700; /* Bold font weight */
    color: #3498db; /* Primary color for data */
    margin: 0; /* Remove default margin */
  }
</style>
