<script>
    import { onMount } from "svelte";
    import Chart from "chart.js/auto";
    let data = [];
    let error = "";
    let chartContainer;
    let selectedOption = "gender"; // Default option
    let chartInstance;
    const options = {
        gender: {
            url: "api/population-by-age-gender-group",
            map: {
                "0": "Other",
                "1": "Male",
                "2": "Female",
            },
        },
        marital_status: {
            url: "api/population-by-age-marital-group",
            map: {
                "1": "Married, spouse present",
                "2": "Married, spouse absent",
                "3": "Separated",
                "4": "Divorced",
                "5": "Widowed",
                "6": "Single/never married",
                "9": "Unknown",
            },
        },
        total_population: {
            url: "api/population-by-age-group",
            map: null, // No mapping needed for total population
        },
        maritalStatusBasedOnGender: {
            url: "api/population-by-age-gender-marital-group",
            map: {
                "0_1": "Other, Married, spouse present",
                "0_2": "Other, Married, spouse absent",
                "0_3": "Other, Separated",
                "0_4": "Other, Divorced",
                "0_5": "Other, Widowed",
                "0_6": "Other, Single/never married",
                "0_9": "Other, Unknown",
                "1_1": "Male, Married, spouse present",
                "1_2": "Male, Married, spouse absent",
                "1_3": "Male, Separated",
                "1_4": "Male, Divorced",
                "1_5": "Male, Widowed",
                "1_6": "Male, Single/never married",
                "1_9": "Male, Unknown",
                "2_1": "Female, Married, spouse present",
                "2_2": "Female, Married, spouse absent",
                "2_3": "Female, Separated",
                "2_4": "Female, Divorced",
                "2_5": "Female, Widowed",
                "2_6": "Female, Single/never married",
                "2_9": "Female, Unknown",
            },
        },
        single_gender: {
            url: "api/population-by-single-age-gender-group",
            map: {
                "0": "Other",
                "1": "Male",
                "2": "Female",
            },
        },
    };
    async function fetchData() {
        try {
            const response = await fetch(options[selectedOption].url);
            if (!response.ok) throw new Error("Network response was not ok.");
            data = await response.json();
            renderChart();
        } catch (e) {
            error = e.message;
        }
    }
    function renderChart() {
        const ctx = chartContainer.getContext("2d");
        // Destroy existing chart instance if it exists
        if (chartInstance) {
            chartInstance.destroy();
        }
        // Sort data by age group
        data.sort((a, b) => {
            const ageA = parseInt(a.age_group.split("-")[0]);
            const ageB = parseInt(b.age_group.split("-")[0]);
            return ageA - ageB;
        });
        const ageGroups = [...new Set(data.map((item) => item.age_group))];
        let datasets;
        if (selectedOption === "total_population") {
            const numbers = data.map((item) => item.number_of_people);
            datasets = [
                {
                    label: "Total Population",
                    data: numbers,
                    backgroundColor: "#42A5F5",
                },
            ];
        } else {
            const map = options[selectedOption].map;
            const keys = Object.keys(map);
            const colorPalette = [
                "#FF5733",
                "#33FF57",
                "#3357FF",
                "#FF33A6",
                "#33FFF5",
                "#FF8C33",
                "#8C33FF",
                "#FFD433",
                "#33FF8C",
                "#FF3333",
            ];
            datasets = keys.map((key, index) => {
                const groupData = ageGroups.map((group) => {
                    let item;
                    if (selectedOption === "maritalStatusBasedOnGender") {
                        const [gender, marital] = key.split("_");
                        item = data.find(
                            (d) =>
                                d.age_group === group &&
                                d.gender === gender &&
                                d.marital_status === marital,
                        );
                    } else if (selectedOption === "single_gender") {
                        item = data.find(
                            (d) => d.age_group === group && d.gender === key,
                        );
                    } else {
                        item = data.find(
                            (d) =>
                                d.age_group === group &&
                                d[selectedOption] === key,
                        );
                    }
                    return item ? item.number_of_people : 0;
                });
                return {
                    label: map[key],
                    data: groupData,
                    backgroundColor: colorPalette[index % colorPalette.length],
                };
            });
        }
        chartInstance = new Chart(ctx, {
            type: "bar",
            data: {
                labels: ageGroups,
                datasets: datasets,
            },
            options: {
                indexAxis: "y", // Horizontal bar chart
                responsive: true,
                scales: {
                    x: {
                        stacked: true,
                        grid: {
                            color: "rgba(220, 220, 220, 0.3)", // Subtle grid lines
                        },
                        ticks: {
                            font: {
                                size: 12, // Font size for x-axis labels
                            },
                            color: "#666", // Color for x-axis labels
                        },
                        title: {
                            display: true,
                            text: "Number of People", // Title for x-axis
                            font: {
                                size: 14,
                                weight: "bold",
                            },
                            color: "#333",
                        },
                    },
                    y: {
                        stacked: true,
                        reverse: true, // Flip the y-axis
                        grid: {
                            color: "rgba(220, 220, 220, 0.3)", // Subtle grid lines
                        },
                        ticks: {
                            font: {
                                size: 12, // Font size for y-axis labels
                            },
                            color: "#666", // Color for y-axis labels
                        },
                        title: {
                            display: true,
                            text: "Age Groups", // Title for y-axis
                            font: {
                                size: 14,
                                weight: "bold",
                            },
                            color: "#333",
                        },
                    },
                },
                plugins: {
                    legend: {
                        position: "top",
                    },
                    tooltip: {
                        callbacks: {
                            label: function (tooltipItem) {
                                return `Number of People: ${tooltipItem.raw}`;
                            },
                        },
                    },
                },
            },
        });
    }
    onMount(fetchData);
</script>

<h1>Population Pyramid by Age and {selectedOption.replace("_", " ")}</h1>
<select bind:value={selectedOption} on:change={fetchData}>
    {#each Object.keys(options) as key}
        <option value={key}>{key.replace("_", " ")}</option>
    {/each}
</select>
{#if error}
    <p>Error: {error}</p>
{:else}
    <canvas bind:this={chartContainer}></canvas>
{/if}
