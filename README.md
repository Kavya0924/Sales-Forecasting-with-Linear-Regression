# Sales-Forecasting-with-Linear-Regression
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sales Forecasting with Linear Regression</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/regression@2.0.1/dist/regression.min.js"></script>
    <style>
        .file-upload {
            position: relative;
            overflow: hidden;
            transition: all 0.3s;
        }
        .file-upload:hover {
            transform: translateY(-2px);
        }
        .file-upload input {
            position: absolute;
            top: 0;
            right: 0;
            margin: 0;
            padding: 0;
            font-size: 20px;
            cursor: pointer;
            opacity: 0;
            filter: alpha(opacity=0);
        }
        .predictions-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 1rem;
        }
        @media (max-width: 640px) {
            .predictions-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen">
    <div class="container mx-auto px-4 py-8">
        <header class="bg-white rounded-lg shadow-md p-6 mb-8">
            <div class="flex flex-col md:flex-row items-center justify-between gap-4">
                <div>
                    <h1 class="text-3xl font-bold text-indigo-700">Sales Forecasting with Linear Regression</h1>
                    <p class="text-gray-600 mt-2">Predict future sales based on historical performance</p>
                </div>
                <div class="file-upload bg-indigo-100 text-indigo-800 px-6 py-3 rounded-lg font-medium cursor-pointer hover:bg-indigo-200">
                    <span>Upload Sales Data (CSV)</span>
                    <input type="file" id="fileInput" accept=".csv" />
                </div>
            </div>
        </header>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
            <div id="uploadHelp" class="bg-white p-6 rounded-lg shadow-md">
                <div class="flex items-start gap-4 mb-4">
                    <div class="bg-indigo-100 p-3 rounded-full">
                        <img src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/a58bde30-b2b8-4881-a17e-b0ca75162f0f.png" alt="Document icon showing a white page with black text and lines" />
                    </div>
                    <div>
                        <h2 class="text-xl font-semibold text-gray-800">Expected CSV Format</h2>
                        <p class="text-gray-600 mt-1">Your file should include these columns with a header row:</p>
                    </div>
                </div>
                <ul class="list-disc pl-5 space-y-2 text-gray-700">
                    <li><span class="font-mono bg-gray-100 px-2 py-1 rounded">date</span> - Format: YYYY-MM-DD</li>
                    <li><span class="font-mono bg-gray-100 px-2 py-1 rounded">product</span> - Product name or ID</li>
                    <li><span class="font-mono bg-gray-100 px-2 py-1 rounded">quantity</span> - Numeric value</li>
                    <li><span class="font-mono bg-gray-100 px-2 py-1 rounded">revenue</span> - Numeric value</li>
                </ul>
                <div class="mt-6 bg-blue-50 border border-blue-200 p-4 rounded-md">
                    <img src="https://storage.googleapis.com/workspace-0f70711f-8b4e-4d94-86f1-2a93ccde5887/image/2ceb442e-beba-4541-a993-0a312117e6e4.png" alt="Information icon showing a white 'i' in a blue circle" class="inline-block mr-2" />
                    <span class="text-blue-700">Sample CSV data will be used if no file is uploaded</span>
                </div>
            </div>

            <div id="dataPreview" class="bg-white p-6 rounded-lg shadow-md hidden">
                <h2 class="text-xl font-semibold text-gray-800 mb-4">Data Preview</h2>
                <div class="overflow-x-auto">
                    <table id="previewTable" class="w-full border-collapse">
                        <thead>
                            <tr class="bg-gray-100">
                                <th class="px-4 py-2 text-left">Date</th>
                                <th class="px-4 py-2 text-left">Product</th>
                                <th class="px-4 py-2 text-left">Quantity</th>
                                <th class="px-4 py-2 text-left">Revenue</th>
                            </tr>
                        </thead>
                        <tbody>
                            <!-- Table content will be populated by JavaScript -->
                        </tbody>
                    </table>
                </div>
                <div id="nullCounts" class="mt-4 grid grid-cols-2 gap-2">
                    <!-- Null counts will be displayed here -->
                </div>
                <button id="processBtn" class="mt-4 w-full bg-indigo-600 hover:bg-indigo-700 text-white font-medium py-2 px-4 rounded-md transition duration-300">
                    Process Data & Generate Forecast
                </button>
            </div>
        </div>

        <div id="resultsSection" class="mt-8 hidden">
            <div class="bg-white p-6 rounded-lg shadow-md mb-8">
                <h2 class="text-xl font-semibold text-gray-800 mb-4">Model Accuracy</h2>
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div class="bg-indigo-50 p-4 rounded-lg">
                        <h3 class="text-sm font-medium text-indigo-700">R-Squared</h3>
                        <p id="rSquared" class="text-2xl font-bold mt-1">--</p>
                    </div>
                    <div class="bg-indigo-50 p-4 rounded-lg">
                        <h3 class="text-sm font-medium text-indigo-700">Mean Error</h3>
                        <p id="meanError" class="text-2xl font-bold mt-1">--</p>
                    </div>
                    <div class="bg-indigo-50 p-4 rounded-lg">
                        <h3 class="text-sm font-medium text-indigo-700">Forecast Period</h3>
                        <p id="forecastPeriod" class="text-2xl font-bold mt-1">-- days</p>
                    </div>
                </div>
            </div>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4">Sales Trend with Forecast</h2>
                    <div class="h-80">
                        <canvas id="forecastChart"></canvas>
                    </div>
                </div>

                <div class="bg-white p-6 rounded-lg shadow-md">
                    <h2 class="text-xl font-semibold text-gray-800 mb-4">Residuals Analysis</h2>
                    <div class="h-80">
                        <canvas id="residualsChart"></canvas>
                    </div>
                </div>
            </div>

            <div class="mt-8 bg-white p-6 rounded-lg shadow-md">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-semibold text-gray-800">Forecast Predictions</h2>
                    <div class="flex items-center gap-2">
                        <label for="forecastDays" class="text-sm font-medium">Days to forecast:</label>
                        <input type="number" id="forecastDays" min="1" max="90" value="30" class="w-20 px-2 py-1 border border-gray-300 rounded-md">
                        <button id="updateForecastBtn" class="bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium py-1 px-3 rounded-md transition duration-300">
                            Update
                        </button>
                    </div>
                </div>
                <div class="overflow-x-auto">
                    <table id="forecastTable" class="w-full border-collapse">
                        <thead>
                            <tr class="bg-gray-100">
                                <th class="px-4 py-2 text-left">Date</th>
                                <th class="px-4 py-2 text-left">Predicted Quantity</th>
                                <th class="px-4 py-2 text-left">Predicted Revenue</th>
                                <th class="px-4 py-2 text-left">Confidence</th>
                            </tr>
                        </thead>
                        <tbody>
                            <!-- Table content will be populated by JavaScript -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            // DOM elements
            const fileInput = document.getElementById('fileInput');
            const dataPreview = document.getElementById('dataPreview');
            const previewTable = document.getElementById('previewTable').querySelector('tbody');
            const nullCounts = document.getElementById('nullCounts');
            const processBtn = document.getElementById('processBtn');
            const resultsSection = document.getElementById('resultsSection');
            const rSquared = document.getElementById('rSquared');
            const meanError = document.getElementById('meanError');
            const forecastPeriod = document.getElementById('forecastPeriod');
            const forecastDaysInput = document.getElementById('forecastDays');
            const updateForecastBtn = document.getElementById('updateForecastBtn');
            const forecastTable = document.getElementById('forecastTable').querySelector('tbody');
            
            let rawData = [];
            let processedData = [];
            let quantityModel = null;
            let revenueModel = null;
            let forecastChart = null;
            let residualsChart = null;

            // Sample data
            const sampleData = `date,product,quantity,revenue
2023-01-01,Product A,150,3000
2023-01-02,Product A,165,3300
2023-01-03,Product A,174,3480
2023-01-04,Product A,182,3640
2023-01-05,Product A,190,3800
2023-01-06,Product A,158,3160
2023-01-07,Product A,143,2860
2023-01-08,Product A,167,3340
2023-01-09,Product A,175,3500
2023-01-10,Product A,189,3780
2023-01-11,Product A,193,3860
2023-01-12,Product A,201,4020
2023-01-13,Product A,195,3900
2023-01-14,Product A,158,3160
2023-01-15,Product A,149,2980
2023-01-16,Product A,178,3560
2023-01-17,Product A,187,3740
2023-01-18,Product A,196,3920
2023-01-19,Product A,205,4100
2023-01-20,Product A,212,4240
2023-01-21,Product A,168,3360
2023-01-22,Product A,157,3140
2023-01-23,Product A,185,3700
2023-01-24,Product A,194,3880
2023-01-25,Product A,203,4060
2023-01-26,Product A,211,4220
2023-01-27,Product A,219,4380
2023-01-28,Product A,226,4520
2023-01-29,Product A,170,3400
2023-01-30,Product A,162,3240`;

            // Event listeners
            fileInput.addEventListener('change', handleFileUpload);
            processBtn.addEventListener('click', processData);
            updateForecastBtn.addEventListener('click', updateForecast);

            // Initialize with sample data if no file is uploaded
            setTimeout(() => {
                if (rawData.length === 0) {
                    loadSampleData();
                }
            }, 500);

            function loadSampleData() {
                const parsed = Papa.parse(sampleData, {
                    header: true,
                    skipEmptyLines: true
                });
                rawData = parsed.data;
                showDataPreview();
            }

            function handleFileUpload(event) {
                const file = event.target.files[0];
                if (!file) return;

                Papa.parse(file, {
                    header: true,
                    skipEmptyLines: true,
                    complete: function(results) {
                        rawData = results.data;
                        showDataPreview();
                    }
                });
            }

            function showDataPreview() {
                // Clear previous content
                previewTable.innerHTML = '';
                nullCounts.innerHTML = '';

                // Show first 5 rows
                for (let i = 0; i < Math.min(5, rawData.length); i++) {
                    const row = document.createElement('tr');
                    row.className = i % 2 === 0 ? 'bg-white' : 'bg-gray-50';
                    
                    row.innerHTML = `
                        <td class="px-4 py-2">${rawData[i].date || '--'}</td>
                        <td class="px-4 py-2">${rawData[i].product || '--'}</td>
                        <td class="px-4 py-2">${rawData[i].quantity || '--'}</td>
                        <td class="px-4 py-2">${rawData[i].revenue || '--'}</td>
                    `;
                    previewTable.appendChild(row);
                }

                // Calculate null counts
                const columns = ['date', 'product', 'quantity', 'revenue'];
                const nullStats = columns.map(col => {
                    const count = rawData.filter(row => row[col] === '' || row[col] === undefined || row[col] === null).length;
                    return {
                        column: col,
                        count: count,
                        percentage: Math.round((count / rawData.length) * 100)
                    };
                });

                // Display null counts
                nullStats.forEach(stat => {
                    const statElement = document.createElement('div');
                    statElement.className = 'bg-gray-100 p-2 rounded-md';
                    statElement.innerHTML = `
                        <div class="flex justify-between items-center">
                            <span class="text-sm font-medium">${stat.column}</span>
                            <span class="text-sm ${stat.count > 0 ? 'text-red-600' : 'text-green-600'}">${stat.count} (${stat.percentage}%) null</span>
                        </div>
                    `;
                    nullCounts.appendChild(statElement);
                });

                dataPreview.classList.remove('hidden');
            }

            function processData() {
                // Data preprocessing
                processedData = rawData
                    .filter(row => row.date && row.quantity && row.revenue) // Remove rows with missing key fields
                    .map(row => {
                        return {
                            date: new Date(row.date),
                            product: row.product || 'Unknown',
                            quantity: parseFloat(row.quantity),
                            revenue: parseFloat(row.revenue)
                        };
                    })
                    .sort((a, b) => a.date - b.date); // Sort by date

                // Map data for regression (days since first date)
                const firstDate = processedData[0].date;
                const quantityPoints = processedData.map((row, index) => [
                            index,
                            row.quantity
                        ]);
                const revenuePoints = processedData.map((row, index) => [
                            index,
                            row.revenue
                        ]);

                // Train models
                quantityModel = regression.linear(quantityPoints);
                revenueModel = regression.linear(revenuePoints);

                // Calculate metrics
                const quantityRSquared = calculateRSquared(processedData.map(row => row.quantity), quantityPoints.map(p => quantityModel.predict(p[0])[1]));
                const revenueRSquared = calculateRSquared(processedData.map(row => row.revenue), revenuePoints.map(p => revenueModel.predict(p[0])[1]));
                const avgRSquared = (quantityRSquared + revenueRSquared) / 2;

                const quantityMeanError = calculateMeanAbsoluteError(processedData.map(row => row.quantity), quantityPoints.map(p => quantityModel.predict(p[0])[1]));
                const revenueMeanError = calculateMeanAbsoluteError(processedData.map(row => row.revenue), revenuePoints.map(p => revenueModel.predict(p[0])[1]));
                const avgMeanError = ((quantityMeanError + revenueMeanError) / 2).toFixed(2);

                // Display metrics
                rSquared.textContent = avgRSquared.toFixed(4);
                meanError.textContent = `$${avgMeanError}`;
                forecastPeriod.textContent = forecastDaysInput.value;

                // Generate charts
                createForecastChart(firstDate, processedData, quantityModel, revenueModel);
                createResidualsChart(processedData, quantityPoints, revenuePoints);

                // Show results and generate forecast
                resultsSection.classList.remove('hidden');
                updateForecast();
            }

            function calculateRSquared(actual, predicted) {
                const mean = actual.reduce((a, b) => a + b, 0) / actual.length;
                const ssRes = actual.reduce((sum, val, i) => sum + Math.pow(val - predicted[i], 2), 0);
                const ssTot = actual.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0);
                return 1 - (ssRes / ssTot);
            }

            function calculateMeanAbsoluteError(actual, predicted) {
                let sum = 0;
                for (let i = 0; i < actual.length; i++) {
                    sum += Math.abs(actual[i] - predicted[i]);
                }
                return sum / actual.length;
            }

            function createForecastChart(firstDate, data, qModel, rModel) {
                const ctx = document.getElementById('forecastChart').getContext('2d');
                
                // Prepare data for chart
                const dates = data.map(row => row.date.toISOString().split('T')[0]);
                const lastDate = new Date(dates[dates.length - 1]);
                
                // Generate forecast dates
                const forecastDays = parseInt(forecastDaysInput.value);
                const forecastDates = [];
                for (let i = 1; i <= forecastDays; i++) {
                    const date = new Date(lastDate);
                    date.setDate(lastDate.getDate() + i);
                    forecastDates.push(date.toISOString().split('T')[0]);
                }
                const allDates = [...dates, ...forecastDates];
                
                // Generate predicted values
                const quantityHistory = data.map(row => row.quantity);
                const quantityPredictions = [];
                for (let i = 0; i < quantityHistory.length + forecastDays; i++) {
                    quantityPredictions.push(qModel.predict(i)[1]);
                }
                
                const revenueHistory = data.map(row => row.revenue);
                const revenuePredictions = [];
                for (let i = 0; i < revenueHistory.length + forecastDays; i++) {
                    revenuePredictions.push(rModel.predict(i)[1]);
                }
                
                // Destroy previous chart if exists
                if (forecastChart) {
                    forecastChart.destroy();
                }
                
                forecastChart = new Chart(ctx, {
                    type: 'line',
                    data: {
                        labels: allDates,
                        datasets: [
                            {
                                label: 'Actual Quantity',
                                data: [...quantityHistory, ...Array(forecastDays).fill(null)],
                                borderColor: 'rgba(79

