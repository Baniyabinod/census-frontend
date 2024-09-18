<script>
  import { onMount } from "svelte";

  let data = [];
  let error = "";
  let showNames = false; // To control visibility of names
  let searchQuery = ""; // For the search functionality

  onMount(async () => {
    try {
      const response = await fetch("/api/population-by-names");
      if (!response.ok) throw new Error("Network response was not ok.");
      data = await response.json();
    } catch (e) {
      error = e.message;
    }
  });

  // Function to filter data based on the search query
  function filteredData() {
    const query = searchQuery.toLowerCase();
    return data.filter(
      (item) =>
        item.fornavn.toLowerCase().includes(query) ||
        item.etternavn.toLowerCase().includes(query)
    );
  }
</script>

<h1>Names of people i.e norwegian ethnicity in 1910 census</h1>

{#if error}
  <p>Error: {error}</p>
{:else}
  <!-- Toggle Button for Showing/Hiding Names -->
  <button on:click={() => (showNames = !showNames)}>
    {showNames ? "Hide Names" : "Show Names"}
  </button>

  <!-- Search Box -->
  <input type="text" placeholder="Search by name..." bind:value={searchQuery} />

  <!-- Displaying Filtered and Conditional Data -->
  {#if showNames}
    <ul>
      {#each filteredData() as item}
        <li>{item.fornavn} {item.etternavn}</li>
      {/each}
    </ul>
  {:else}
    <p>Names are hidden. Click "Show Names" to view them.</p>
  {/if}
{/if}
