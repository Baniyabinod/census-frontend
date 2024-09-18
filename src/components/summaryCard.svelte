<script>
  import { onMount } from "svelte";

  export let apiUrl = "";
  export let title = ""; // Title of the summary card
  export let dataMap = {}; // Key-value pairs for mapping
  export let labelField = ""; // The field in the stats for the label (e.g., 'abc')
  export let valueField = ""; // The field in the stats for the value (e.g., 'lll')

  let stats = [];
  let error = "";

  // Fetch data when component mounts
  onMount(async () => {
    try {
      const response = await fetch(apiUrl); // Use the API URL passed as a prop
      if (!response.ok) throw new Error("Network response was not ok.");
      stats = await response.json(); // Store the fetched stats
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
          <h3>{dataMap[item[labelField]] || "Unknown"}</h3>
          <!-- Dynamic label mapping -->
          <p>{item[valueField]}</p>
          <!-- Dynamic value display -->
        </div>
      {/each}
    </div>
  {/if}
</div>

<style>
  /* Summary Card Container */
  .summary-card {
    margin: 40px auto;
    max-width: 1200px; /* Max width for larger screens */
    padding: 20px;
    background-color: #ffffff;
    border-radius: 12px;
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
    text-align: center;
    border-radius: 20px;
  }

  /* Heading Style */
  .summary-card h2 {
    font-size: 28px;
    font-weight: 700;
    color: #333;
    margin-bottom: 30px;
  }

  /* Error Message */
  .summary-card p {
    color: #e74c3c; /* Red color for errors */
    font-size: 18px;
    margin: 20px 0;
  }

  /* Card Container */
  .card-container {
    display: flex;
    gap: 20px;
    justify-content: center;
    flex-wrap: wrap;
  }

  /* Individual Card */
  .card {
    padding: 20px;
    background-color: #ffffff;
    border-radius: 10px;
    text-align: center;
    width: 200px; /* Increased width for better content fit */
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    transition:
      box-shadow 0.3s ease,
      transform 0.3s ease;
    margin: 10px; /* Added margin to separate cards */
  }

  .card:hover {
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
    transform: translateY(-4px); /* Slight lift on hover */
  }

  /* Card Title */
  .card h3 {
    font-size: 20px;
    font-weight: 600;
    color: #3498db; /* Primary color */
    margin-bottom: 10px;
  }

  /* Card Value */
  .card p {
    font-size: 24px;
    font-weight: 700;
    color: #333;
    margin: 0;
  }

  /* Responsive Design */
  @media (max-width: 768px) {
    .summary-card {
      padding: 15px;
    }

    .card-container {
      gap: 15px;
    }

    .card {
      width: 150px; /* Smaller width for smaller screens */
    }

    .summary-card h2 {
      font-size: 24px;
    }

    .card h3 {
      font-size: 18px;
    }

    .card p {
      font-size: 20px;
    }
  }
</style>
