<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  export let apiUrl = "";
  export let title = ""; // Chart title
  export let statusMap = {}; // Status mapping object (e.g., LanguageStatusMap)
  export let label = ""; // Label for the dataset
  export let labelField = ""; // Dynamic field to map labels (e.g., "spraak", "age_group")
  export let itemMapping = (item) => item; // Function to map data (default is identity)

  let chartContainer;
  let error = "";

  // Function to generate a random color
  function getRandomColor() {
    const letters = "0123456789ABCDEF";
    let color = "#";
    for (let i = 0; i < 6; i++) {
      color += letters[Math.floor(Math.random() * 16)];
    }
    return color;
  }

  onMount(async () => {
    try {
      const response = await fetch(apiUrl);
      if (!response.ok) throw new Error("Network response was not ok.");
      const stats = await response.json();

      const ctx = chartContainer.getContext("2d");
      new Chart(ctx, {
        type: "bar",
        data: {
          labels: stats.map((item) => statusMap[item[labelField]] || "Unknown"),
          datasets: [
            {
              label: label,
              data: stats.map(itemMapping),
              backgroundColor: stats.map(() => getRandomColor()), // Unique color for each bar
              borderColor: "#ffffff",
              borderWidth: 2,
              hoverBackgroundColor: "#FFB74D",
              hoverBorderColor: "#ffffff",
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false, // Allow chart to resize properly
          plugins: {
            legend: {
              position: "top",
              labels: {
                color: "#333",
                font: {
                  size: 16,
                },
              },
            },
            tooltip: {
              backgroundColor: "#000",
              titleColor: "#fff",
              bodyColor: "#fff",
              callbacks: {
                label: function (tooltipItem) {
                  return `Number of Persons: ${tooltipItem.raw}`;
                },
              },
            },
          },
          scales: {
            x: {
              grid: {
                display: false,
              },
              title: {
                display: true,
                text: "Categories",
                color: "#333",
                font: {
                  size: 16,
                },
              },
              ticks: {
                color: "#333",
                maxRotation: 45,
                minRotation: 30,
              },
            },
            y: {
              grid: {
                color: "#ddd",
              },
              title: {
                display: true,
                text: "Number of Persons",
                color: "#333",
                font: {
                  size: 16,
                },
              },
              ticks: {
                color: "#333",
                stepSize: 10,
              },
            },
          },
          animation: {
            duration: 800,
            easing: "easeOutBounce",
          },
        },
      });
    } catch (e) {
      error = e.message;
      console.error(e.message);
    }
  });
</script>

<h1>{title}</h1>
{#if error}
  <p>Error: {error}</p>
{:else}
  <div class="chart-container">
    <canvas bind:this={chartContainer}></canvas>
  </div>
{/if}

<style>
  .chart-container {
    position: relative;
    width: 100%;
    height: 500px; /* Fixed height for better appearance */
    margin: 0 auto;
  }
  canvas {
    width: 100% !important;
    height: 100% !important;
    display: block;
  }
</style>
