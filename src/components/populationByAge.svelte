<script>
    import { onMount } from 'svelte';

    // Props for dynamic input
    export let apiUrl = '';         // API endpoint passed as a prop
    export let dataMap = {};        // Key-value pairs for mapping (e.g., gender map)
    export let labelField = '';     // The field in the stats for the label (e.g., 'aaa')
    export let valueField = '';     // The field in the stats for the value (e.g., 'Number_of_Persons')

    let minAge = 20;
    let maxAge = 30;
    let stats = [];
    let error = '';

    // Function to fetch data from the backend
    async function fetchPopulationByAge() {
        try {
            // Construct the URL with age range parameters
            const url = `${apiUrl}?min_age=${minAge}&max_age=${maxAge}`;
            console.log(`Fetching data from: ${url}`);  // Log the URL for debugging

            const response = await fetch(url);
            if (!response.ok) throw new Error('Network response was not ok.');

            // Parse the response data
            stats = await response.json();  // Update the stats array, triggers UI update
            error = '';
        } catch (e) {
            console.error('Fetch error:', e.message);  // Log error
            error = e.message;
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

        <button on:click={fetchPopulationByAge}>Fetch Population</button>
    </div>

    {#if error}
        <p>Error: {error}</p>
    {:else if stats.length === 0}
        <p>No data available for the selected age range.</p>
    {:else}
        <div class="card-container">
            {#each stats as item}
                <div class="card">
                    <!-- Dynamic label mapping based on dataMap and labelField -->
                    <h3>{dataMap[item[labelField]] || 'Unknown'}</h3>  
                    
                    <!-- Dynamic value display based on valueField -->
                    <p>{item[valueField]}</p>
                </div>
            {/each}
        </div>
    {/if}
</div>

<style>
    .population-by-age {
        margin: 20px 0;
        text-align: center;
    }

    .input-container {
        display: flex;
        gap: 10px;
        justify-content: center;
        margin-bottom: 20px;
    }

    .input-container label {
        margin-right: 5px;
    }

    .input-container input {
        width: 60px;
        padding: 5px;
        border: 1px solid #ccc;
        border-radius: 4px;
    }

    .input-container button {
        padding: 5px 10px;
        background-color: #42A5F5;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
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
