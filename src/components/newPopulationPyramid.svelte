<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  let data = [];
  let error = "";
  let chartContainer;
  let selectedOption = "gender"; // Default option
  let selectedMaritalStatus = "married"; // Default marital status (Married)
  let chartInstance;

  const options = {
    gender: {
      url: "api/population-by-age-gender-group",
      map: {
        "0": "Male",
        "1": "Female",
      },
    },
    marital_status: {
      map: {
        married: "Married Spouse Present",
        single: "Single/Never Married",
        widowed: "Widowed",
        separated: "Separated",
      },
    },
    single_gender: {
      url: "/population-by-single-age-gender-group",
      map: {
        "1": "Male",
        "2": "Female",
      },
    },
  };

  async function fetchData() {
    try {
      let url;
      if (selectedOption === "gender") {
        url = options.gender.url;
      } else if (selectedOption === "single_gender") {
        url = options.single_gender.url;
      } else if (selectedOption === "marital_status") {
        url = `/api/population-by-age-gender-marital-group?marital_status=${selectedMaritalStatus}`;
      }

      const response = await fetch(url);
      if (!response.ok) throw new Error("Network response was not ok.");
      data = await response.json();

      console.log("Fetched Data:", data); // Log the fetched data
      renderChart();
    } catch (e) {
      error = e.message;
    }
  }

  function renderGenderChart() {
    const ctx = chartContainer.getContext("2d");

    if (chartInstance) {
      chartInstance.destroy();
    }

    const ageGroups = [...new Set(data.map((item) => item.age_group))];
    const maleData = ageGroups.map((group) => {
      const item = data.find((d) => d.gender === "1" && d.age_group === group); // Adjusted to match API response
      return item ? -item.number_of_people : 0; // Negative for males
    });
    const femaleData = ageGroups.map((group) => {
      const item = data.find((d) => d.gender === "2" && d.age_group === group); // Adjusted to match API response
      return item ? item.number_of_people : 0; // Positive for females
    });

    const datasets = [
      {
        label: "Male",
        data: maleData,
        backgroundColor: "#42A5F5",
      },
      {
        label: "Female",
        data: femaleData,
        backgroundColor: "#FF8C33",
      },
    ];

    chartInstance = new Chart(ctx, {
      type: "bar",
      data: {
        labels: ageGroups,
        datasets: datasets,
      },
      options: {
        indexAxis: "y",
        responsive: true,
        scales: {
          x: {
            min: Math.min(0, ...maleData, ...femaleData),
            max: Math.max(0, ...maleData, ...femaleData),
            beginAtZero: true,
            ticks: {
              callback: function (value) {
                return Math.abs(value); // Show positive values on x-axis
              },
            },
          },
          y: {
            stacked: true,
            reverse: true,
          },
        },
      },
    });
  }
  function renderMaritalStatusChart() {
    const ctx = chartContainer.getContext("2d");

    if (chartInstance) {
      chartInstance.destroy();
    }

    const ageGroups = [...new Set(data.map((item) => item.age_group))];

    // Initialize data arrays for males and females
    const maleData = ageGroups.map((group) => {
      const totalMale = data
        .filter((d) => d.gender === 1 && d.age_group === group) // Filter for males
        .reduce((sum, d) => sum + d.number_of_people, 0); // Sum up number_of_people

      return -totalMale; // Negative for males
    });

    const femaleData = ageGroups.map((group) => {
      const totalFemale = data
        .filter((d) => d.gender === 2 && d.age_group === group) // Filter for females
        .reduce((sum, d) => sum + d.number_of_people, 0); // Sum up number_of_people

      return totalFemale; // Positive for females
    });

    console.log("Male Data:", maleData); // Log male data
    console.log("Female Data:", femaleData); // Log female data

    const datasets = [
      {
        label: "Male",
        data: maleData,
        backgroundColor: "#42A5F5",
      },
      {
        label: "Female",
        data: femaleData,
        backgroundColor: "#FF8C33",
      },
    ];

    chartInstance = new Chart(ctx, {
      type: "bar",
      data: {
        labels: ageGroups,
        datasets: datasets,
      },
      options: {
        indexAxis: "y",
        responsive: true,
        scales: {
          x: {
            beginAtZero: true,
            ticks: {
              callback: function (value) {
                return Math.abs(value); // Show positive values on x-axis
              },
            },
          },
          y: {
            stacked: true,
            reverse: true,
          },
        },
      },
    });
  }
  function renderSingleGenderChart() {
    const ctx = chartContainer.getContext("2d");

    if (chartInstance) {
      chartInstance.destroy();
    }

    const ageGroups = [...new Set(data.map((item) => item.age_group))];
    const maleData = ageGroups.map((group) => {
      const item = data.find((d) => d.gender === "1" && d.age_group === group);
      return item ? -item.number_of_people : 0; // Negative for males
    });
    const femaleData = ageGroups.map((group) => {
      const item = data.find((d) => d.gender === "2" && d.age_group === group);
      return item ? item.number_of_people : 0; // Positive for females
    });

    const datasets = [
      {
        label: "Male",
        data: maleData,
        backgroundColor: "#42A5F5",
      },
      {
        label: "Female",
        data: femaleData,
        backgroundColor: "#FF8C33",
      },
    ];

    chartInstance = new Chart(ctx, {
      type: "bar",
      data: {
        labels: ageGroups,
        datasets: datasets,
      },
      options: {
        indexAxis: "y",
        responsive: true,
        scales: {
          x: {
            // Calculate min and max values from data
            min: Math.min(0, ...maleData, ...femaleData),
            max: Math.max(0, ...maleData, ...femaleData),
            ticks: {
              callback: function (value) {
                return Math.abs(value); // Show positive values on x-axis
              },
            },
          },
          y: {
            stacked: true,
            reverse: true,
          },
        },
      },
    });
  }

  function renderChart() {
    if (selectedOption === "gender") {
      renderGenderChart();
    } else if (selectedOption === "marital_status") {
      renderMaritalStatusChart();
    } else if (selectedOption === "single_gender") {
      renderSingleGenderChart();
    }
  }

  onMount(fetchData);
</script>

<div>
  <h1>Population Pyramid by Age and {selectedOption.replace("_", " ")}</h1>
  <div class="options-container">
    {#each Object.keys(options) as key}
      <div
        class="option {selectedOption === key ? 'selected' : ''}"
        on:click={() => {
          selectedOption = key;
          if (key === "marital_status") {
            selectedMaritalStatus = "married"; // Reset to default marital status
          }
          fetchData();
        }}
      >
        {key.replace("_", " ").toUpperCase()}
      </div>
    {/each}
  </div>

  {#if selectedOption === "marital_status"}
    <div class="marital-status-container">
      {#each Object.keys(options.marital_status.map) as key}
        <button
          class="marital-button {selectedMaritalStatus === key
            ? 'selected'
            : ''}"
          on:click={() => {
            selectedMaritalStatus = key;
            fetchData(); // Fetch data when marital status is changed
          }}
        >
          {options.marital_status.map[key]}
        </button>
      {/each}
    </div>
  {/if}

  {#if error}
    <p>Error: {error}</p>
  {:else}
    <div class="chart-container">
      <canvas bind:this={chartContainer}></canvas>
    </div>
  {/if}
</div>

<style>
  .options-container {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    align-items: center;
    justify-content: center;
  }

  .option,
  .marital-button {
    padding: 10px 20px;
    background-color: #f0f0f0;
    border-radius: 4px;
    cursor: pointer;
    transition: border-color 0.3s;
    border: none;
    color: #333;
  }

  .option.selected,
  .marital-button.selected {
    background-color: #007bff;
    color: white;
  }

  .option:hover,
  .marital-button:hover {
    background-color: #007bff;
    color: white;
  }

  .marital-status-container {
    display: flex;
    gap: 10px;
    margin-top: 20px; /* Adjusted margin */
    justify-content: center;
  }

  .chart-container {
    position: relative;
    height: 600px; /* Fixed height to stabilize the layout */
    margin-top: 20px;
  }

  canvas {
    width: 100% !important; /* Ensure canvas stretches to container width */
    height: auto !important; /* Ensure canvas stretches to container height */
  }
</style>
