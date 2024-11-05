<!--Code gotten from ChatGPT using the query "make this compostiton api and make it work"-->
<template>
    <div>
        <!-- Form for user input to fetch weather data -->
        <form @submit.prevent="fetchRemoteData">
            <div class="form-group">
                <h2>Weather Data</h2>
                <p>Please fill in the form and use the submit button to load data.</p>
                <label for="">Location</label><br>
                <!-- Input for location, bound to formInput.location -->
                <input class="form-control rounded" v-model="formInput.location" type="text" required />
            </div>

            <div class="form-group">
                <label for="">Start Date</label><br>
                <!-- Input for start date, bound to formInput.startDt -->
                <input class="form-control rounded" type="date" v-model="formInput.startDt" required />
            </div>

            <div class="form-group">
                <label for="">End Date</label><br>
                <!-- Input for end date, bound to formInput.endDt -->
                <input class="form-control rounded" type="date" v-model="formInput.endDt" required />
            </div><br>

            <!-- Submit button -->
            <div>
                <button class="btn btn-danger" type="submit">Submit</button>
            </div>
        </form><br /><br />

        <!-- Chart component to display data, shown only if data is loaded -->
        <div v-if="dataLoaded">
            <Bar :data="chartData" :options="chartOptions" />
        </div>

        <!-- Table view for weather data, displayed if data is loaded -->
        <div v-if="dataLoaded">
            <hr>
            <h5>Weather Data in a Table</h5>
            <table class="table">
                <thead>
                    <tr>
                        <!-- Table header for each date, dynamic with v-for -->
                        <th v-for="adate in keys" :key="adate">{{ adate }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <!-- Table cells for each temperature value, dynamic with v-for -->
                        <td v-for="atemp in values" :key="atemp">{{ atemp }}</td>
                    </tr>
                </tbody>
            </table>            
        </div>
    </div>
</template>

<script>
import axios from "axios";
import { ref } from "vue";
import { Bar } from "vue-chartjs";
import { Chart as ChartJS, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js';

// Register Chart.js components to enable chart rendering
ChartJS.register(CategoryScale, LinearScale, BarElement, Title, Tooltip, Legend);

export default {
    components: {
        Bar, // Registering the Bar chart component from vue-chartjs
    },
    setup() {
        // Reactive state for form inputs (location, start and end dates)
        const formInput = ref({ startDt: "", endDt: "", location: "" });

        // Boolean reactive state to check if data is loaded
        const dataLoaded = ref(false);

        // Arrays to store date and temperature data for the table and chart
        const keys = ref([]); // Dates
        const values = ref([]); // Temperatures
        
        // Chart configuration options (axes labels, responsiveness)
        const chartOptions = {
            responsive: true,
            maintainAspectRatio: true,
            scales: {
                x: { title: { display: true, text: "Time" } },
                y: { title: { display: true, text: "Temperature (F)" } },
            },
        };

        // Reactive data structure for chart data
        const chartData = ref({
            labels: [], // Labels will be set dynamically
            datasets: [{ label: "Average Temperature", backgroundColor: "#f87979", data: [] }],
        });

        // Function to fetch weather data from API using axios
        const fetchRemoteData = async () => {
            // API request options, with parameters based on form input
            const options = {
                method: 'GET',
                url: 'https://weatherapi-com.p.rapidapi.com/history.json',
                params: {
                    q: formInput.value.location,
                    dt: formInput.value.startDt,
                    end_dt: formInput.value.endDt,
                    lang: 'en'
                },
                headers: {
                    'X-RapidAPI-Key': 'c5ba91ee15msh88e0253ce262ddap13b6e7jsnd353ddeec9a4',
                    'X-RapidAPI-Host': 'weatherapi-com.p.rapidapi.com'
                }
            };
            try {
                // Await API response
                const resp = await axios.request(options);
                
                // Parse response to extract weather data
                const w_data = resp.data.forecast.forecastday;
                keys.value = w_data.map((item) => item.date); // Set keys to dates
                values.value = w_data.map((item) => item.day.avgtemp_f); // Set values to temperatures

                // Update chartData with the fetched data
                chartData.value = {
                    labels: keys.value,
                    datasets: [{ label: "Average Temperature", backgroundColor: "#f87979", data: values.value }],
                };
                dataLoaded.value = true; // Indicate data is successfully loaded
            } catch (error) {
                console.error(error); // Log errors if request fails
            }
        };

        // Returning all state and methods to the template
        return {
            formInput,
            dataLoaded,
            keys,
            values,
            chartOptions,
            chartData,
            fetchRemoteData,
        };
    }
};
</script>

<style>
/* Basic table styling */
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
</style>


