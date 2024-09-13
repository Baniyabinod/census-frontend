<script>
    import { onMount, onDestroy } from "svelte";
    import Chart from "chart.js/auto";

    export let apiUrl = ""; // API endpoint
    export let title = ""; // Chart title
    export let itemMapping = (item) => item; // Function to map data (default is identity)

    let chartContainer;
    let error = "";
    let chart = null; // Variable to hold the Chart.js instance
    let groupBy = "religion"; // Default grouping field
    let statusMap = {};
    let label = "";
    let labelField = "";

    // Sample status maps for different groupings
    const religionMap = {
        "1100": "Lutheran",
        "2300": "Baptist",
        "3100": "Roman Catholic",
        "3300": "Greek Orthodox",
        "4100": "Jewish",
        "4700": "Muslim",
        "4950": "Other Non-Christian",
        "5200": "Other Protestant",
        "6300": "Other Christian",
        "7100": "Buddhist",
        "8310": "Hindu",
        "8440": "Shinto",
        "8510": "Confucian",
        "8700": "Taoist",
        "8815": "Sikh",
        "8875": "Jain",
        "8890": "Zoroastrian",
        "9100": "Bahá'í",
        "9130": "Traditional African Religions",
        "9710": "Native American Religions",
        "9880": "No religion",
        "9990": "Unknown",
        "9998": "Not stated",
    };

    const ethnicityMap = {
        "2100": "English",
        "3100": "German",
        "4050": "Norwegian",
        "4100": "Sami (lapp)",
        "4110": "Sami, nomads",
        "4120": "Sami, resident",
        "4200": "Mixed",
        "4210": "Norwegian-Sami",
        "4220": "Norwegian-Finnish",
        "4230": "Sami-Finnish",
        "5100": "Scandinavian",
        "5200": "Swedish",
        "5300": "Finnish",
        "5400": "Danish",
        "5500": "Icelander",
        "64109": "Bohemian",
        "6420": "Hungarian",
        "6430": "Polish or Mazurian",
        "6510": "Russian",
        "6520": "Russian Jew",
        "6730": "Italian",
        "7100": "American (Unspecified)",
        "9970": "Unknown",
        "9990": "No information",
    };

    const familyPositionMap = {
        "01": "Household head",
        "02": "Spouse",
        "03": "Child, stepchild",
        "05": "Fostered, adopted child",
        "07": "Child-in-law",
        "08": "Grandchild",
        "09": "Parent, parent-in-law",
        "11": "Sibling, sibling-in-law",
        "13": "Other relatives",
        "14": "Servants",
        "15": "Apprentice",
        "16": "Lodgers",
        "17": "Paupers",
        "18": "Living with others",
        "20": "Visitors",
        "21": "Retired people with farm benefits",
        "23": "Others",
        "99": "Unknown",
        "00": "No information",
    };
    const maritalStatusMap = {
        "1": "Married, spouse present",
        "2": "Married, spouse absent",
        "3": "Separated",
        "4": "Divorced",
        "5": "Widowed",
        "6": "Single/never married",
        "9": "Unknown",
    };

    const primaryOccupationStatusMap = {
        "00": "Not given",
        "01": "Farmer, owner - ordinary sized farm",
        "02": "Proprietor of estate",
        "03": "House or building owner in town or city",
        "04": "Tenant farmer, lease-holder",
        "05": "Farmer, unspecified",
        "06": "Retired agriculturalists,remaining on farm with formal contract with (usually) children",
        "07": "Cottager with land",
        "08": "Cottager without land",
        "09": "Cottager (unspecified)",
        "10": "Lodger",
        "12": "Servant",
        "13": "Helps in the family",
        "20": "Owns and/or runs a business",
        "21": "Civil servant, high level",
        "22": "Civil servant, middle status",
        "23": "Civil servant, low status",
        "24": "Foreman",
        "25": "Guard, watchman, housekeeper, farm manager",
        "26": "Master (position gained through formal training and certified; owns own shop)",
        "27": "Other work leader (officer in the army, naval officer at court)",
        "37": "Journeyman",
        "38": "Workers, Day labourers",
        "39": "Apprentice",
        "40": "Other",
        "41": "Agent, Representative of companies or firms",
        "50": "Student, pupil",
        "51": "Rentier, “capitalist”",
        "52": "Pensioned",
        "53": "No longer working",
        "54": "Orphan, foster child",
        "55": "Pauper (fattiglem)",
        "56": "On municipal support",
        "57": "Receiving private or other support",
        "58": "Beggar",
        "59": "Prisoner",
    };

    const secondaryOccupationStatusMap = {
        "00": "Not given",
        "01": "Farmer, owner - ordinary sized farm",
        "02": "Proprietor of estate",
        "03": "House or building owner in town or city",
        "04": "Tenant farmer, lease-holder",
        "05": "Farmer, unspecified",
        "06": "Retired agriculturalists, remaining on farm with formal contract with (usually) children",
        "07": "Cottager with land",
        "08": "Cottager without land",
        "09": "Cottager (unspecified)",
        "10": "Lodger",
        "12": "Servant",
        "13": "Helps in the family",
        "20": "Owns and/or runs a business",
        "21": "Civil servant, high level",
        "22": "Civil servant, middle status",
        "23": "Civil servant, low status",
        "24": "Foreman",
        "25": "Guard, watchman, housekeeper, farm manager",
        "26": "Master (position gained through formal training and certified; owns own shop)",
        "27": "Other work leader (officer in the army, naval officer at court)",
        "37": "Journeyman",
        "38": "Workers, Day labourers",
        "39": "Apprentice",
        "40": "Other",
        "41": "Agent, Representative of companies or firms",
        "50": "Student, pupil",
        "51": "Rentier, “capitalist”",
        "52": "Pensioned",
        "53": "No longer working",
        "54": "Orphan, foster child",
        "55": "Pauper (fattiglem)",
        "56": "On municipal support",
        "57": "Receiving private or other support",
        "58": "Beggar",
        "59": "Prisoner",
    };

    const medicalConditionStatusMap = {
        "21": "Mental disease",
        "24": "Blind",
        "25": "Deaf",
        "26": "Deaf and dumb",
        "99": "Other",
        "00": "No information",
        x: "no reference",
    };
    const LanguageStatusMap = {
        "0500": "Swedish",
        "0700": "Norwegian",
        "3300": "Finnish",
        "3520": "Sami",
        "9600": "Other",
        "9900": "No information",
    };

    const citizenshipStatusMap = {
        "404": "Norwegian",
        "405": "Swedish",
        "401": "Finnish",
        "400": "Danish",
        "4538": "German",
        "465": "Russian",
        "499": "Rest of Europe",
        "900": "Rest of World",
        "997": "Unknown",
        "999": "No information",
        x: "No reference",
    };

    // Function to generate a random color
    function getRandomColor() {
        const letters = "0123456789ABCDEF";
        let color = "#";
        for (let i = 0; i < 6; i++) {
            color += letters[Math.floor(Math.random() * 16)];
        }
        return color;
    }

    // Function to fetch and update the chart data
    async function fetchAndUpdateChart() {
        try {
            const response = await fetch(`${apiUrl}?group_by=${groupBy}`);
            if (!response.ok) throw new Error("Network response was not ok.");
            const stats = await response.json();

            // Destroy old chart if exists
            if (chart) {
                chart.destroy();
            }

            // Update statusMap, labelField, and label based on groupBy
            if (groupBy === "religion") {
                statusMap = religionMap;
                labelField = "religion";
                label = "Number of Persons by Religion";
            } else if (groupBy === "etnisitet_fars_etnisitet_i_1875") {
                statusMap = ethnicityMap;
                labelField = "etnisitet_fars_etnisitet_i_1875";
                label = "Number of Persons by Ethnicity";
            } else if (groupBy === "famstatus_pos_en") {
                statusMap = familyPositionMap;
                labelField = "famstatus_pos_en";
                label = "Number of Persons by Family Position";
            } else if (groupBy === "sivilstatus") {
                statusMap = maritalStatusMap;
                labelField = "sivilstatus";
                label = "Number of Persons by Marital Status";
            } else if (groupBy === "occ_hierarkisk_hovedyrke") {
                statusMap = primaryOccupationStatusMap;
                labelField = "occ_hierarkisk_hovedyrke";
                label = "Number of Persons by Primary Occupation";
            } else if (groupBy === "occ_hierarkisk_biyrke") {
                statusMap = secondaryOccupationStatusMap;
                labelField = "occ_hierarkisk_biyrke";
                label = "Number of Persons by Secondary Occupation";
            } else if (groupBy === "sykdom") {
                statusMap = medicalConditionStatusMap;
                labelField = "sykdom";
                label = "Number of Persons by Medical Condition";
            } else if (groupBy === "spraak") {
                statusMap = LanguageStatusMap;
                labelField = "spraak";
                label = "Number of Persons by Language";
            } else if (groupBy === "statsborgerskap") {
                statusMap = citizenshipStatusMap;
                labelField = "statsborgerskap";
                label = "Number of Persons by Citizenship";
            }
            const ctx = chartContainer.getContext("2d");
            chart = new Chart(ctx, {
                type: "bar",
                data: {
                    labels: stats.map(
                        (item) => statusMap[item[labelField]] || "Unknown",
                    ),
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
    }

    onMount(() => {
        fetchAndUpdateChart(); // Fetch data and initialize chart on component mount
    });

    // Watch for changes in the groupBy value
    function handleGroupByChange(event) {
        groupBy = event.target.value;
        fetchAndUpdateChart();
    }

    onDestroy(() => {
        if (chart) {
            chart.destroy(); // Destroy chart instance on component unmount
        }
    });
</script>

<h1>{title}</h1>
{#if error}
    <p>Error: {error}</p>
{:else}
    <div class="input-container">
        <label for="group-by">Group By:</label>
        <select
            id="group-by"
            bind:value={groupBy}
            on:change={handleGroupByChange}
        >
            <option value="religion">Religion</option>
            <option value="etnisitet_fars_etnisitet_i_1875">Ethnicity</option>
            <option value="famstatus_pos_en">family Position</option>
            <option value="sivilstatus">Marital Status</option>
            <option value="occ_hierarkisk_hovedyrke"
                >primary Occupation Status</option
            >
            <option value="occ_hierarkisk_biyrke"
                >secondary Occupation Status</option
            >
            <option value="sykdom">Medical Condition</option>
            <option value="spraak">Language</option>
            <option value="statsborgerskap">Citizenship</option>
        </select>
    </div>
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
    .input-container {
        margin-bottom: 20px;
        text-align: center;
    }
    select {
        padding: 5px;
        font-size: 16px;
    }
</style>
