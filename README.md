<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quality Defect Tracking Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Base typography and theme colors */
        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: #fcfbf9; /* Warm neutral light background */
            color: #292524; /* Stone 800 */
        }
        
        /* Required Chart Container Styling */
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 100%;
            margin-left: auto;
            margin-right: auto;
            height: 300px;
            max-height: 400px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 350px;
            }
        }
        @media (min-width: 1024px) {
            .chart-container.large {
                height: 400px;
            }
        }

        /* Custom scrollbar for table */
        .table-scroll::-webkit-scrollbar {
            height: 8px;
        }
        .table-scroll::-webkit-scrollbar-track {
            background: #f5f5f4; 
            border-radius: 4px;
        }
        .table-scroll::-webkit-scrollbar-thumb {
            background: #d6d3d1; 
            border-radius: 4px;
        }
        .table-scroll::-webkit-scrollbar-thumb:hover {
            background: #a8a29e; 
        }

        /* Status Badge Utilities */
        .badge {
            display: inline-flex;
            align-items: center;
            padding: 0.25rem 0.6rem;
            border-radius: 9999px;
            font-size: 0.75rem;
            font-weight: 600;
            white-space: nowrap;
        }
        .badge-critical { background-color: #fee2e2; color: #991b1b; }
        .badge-over { background-color: #ffedd5; color: #9a3412; }
        .badge-ok { background-color: #dcfce3; color: #166534; }
    </style>
</head>
<body class="antialiased min-h-screen pb-12">

    <!-- Chosen Palette: Warm Neutrals with Semantic Accents (Stone, Amber, Emerald, Rose) -->
    <!-- Application Structure Plan: A single-page dashboard layout. Chosen because the primary goal is monitoring and quick filtering of metrics based on time (Month). It features a persistent header with global filtering, a top row of KPI cards for immediate status checks, a middle section with visualizations to understand defect distribution and sources, and a bottom detail table for granular, row-level analysis. This hierarchical approach moves the user from high-level summary down to actionable details. -->
    <!-- Visualization & Content Choices: 
         1. KPIs (HTML/CSS): Goal: Inform. Shows immediate total counts and health status. Interaction: Auto-updates on month filter.
         2. Stacked Bar Chart (Chart.js): Goal: Compare. Shows defect volume by category, stacked by source (Prod/Cust). Justification: Best for comparing part-to-whole relationships across multiple categories simultaneously.
         3. Doughnut Chart (Chart.js): Goal: Proportions. Shows the overall ratio of Production vs Customer defects. Justification: Quick visual understanding of where defects originate globally.
         4. Data Table (HTML): Goal: Organize/Detail. Shows exact metrics and status against targets. Interaction: Scrollable, responsive.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. Relying on HTML structure, CSS, Unicode symbols, and Canvas for all visuals. -->

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        
        <!-- Header & Controls -->
        <header class="mb-8 flex flex-col md:flex-row md:items-end justify-between gap-4 border-b border-stone-200 pb-6">
            <div>
                <h1 class="text-3xl font-bold text-stone-900 tracking-tight mb-2">🏭 Quality Defect Tracker</h1>
                <p class="text-stone-500 max-w-2xl">
                    Interactive dashboard monitoring manufacturing and customer-reported defects against target thresholds. Use the filter below to isolate data for specific months.
                </p>
            </div>
            <div class="flex flex-col gap-1 min-w-[200px]">
                <label for="monthFilter" class="text-sm font-semibold text-stone-600 uppercase tracking-wider">📅 Filter by Month</label>
                <select id="monthFilter" class="bg-white border border-stone-300 text-stone-900 text-base rounded-lg focus:ring-amber-500 focus:border-amber-500 block w-full p-2.5 shadow-sm transition-colors cursor-pointer outline-none">
                    <option value="All">All Months</option>
                    <!-- Options populated by JS -->
                </select>
            </div>
        </header>

        <!-- Context Paragraph -->
        <div class="mb-6 bg-white p-5 rounded-xl border border-stone-200 shadow-sm">
            <h2 class="text-lg font-semibold text-stone-800 mb-2">Executive Summary</h2>
            <p class="text-stone-600 text-sm leading-relaxed">
                This section provides an immediate overview of product quality health. The Key Performance Indicators (KPIs) below aggregate the total defect volume and categorize the operational status of our production batches. "Critical" or "Over Target" statuses indicate categories requiring immediate corrective action. Data automatically recalculates based on your selected month.
            </p>
        </div>

        <!-- KPI Cards -->
        <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-8" id="kpiContainer">
            <!-- Populated by JS -->
        </section>

        <!-- Visualizations Section -->
        <section class="mb-8">
            <div class="mb-4">
                <h2 class="text-xl font-bold text-stone-800">Defect Distribution Analysis</h2>
                <p class="text-stone-500 text-sm mt-1">
                    Visual breakdown of defect quantities by category and their point of origin (Production Line vs. Customer Report). 
                    Hover over the chart segments to view specific quantities.
                </p>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Bar Chart: Category Breakdown -->
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm lg:col-span-2 flex flex-col">
                    <h3 class="text-sm font-bold text-stone-700 uppercase tracking-wide mb-4">Volume by Category & Source</h3>
                    <div class="chart-container flex-grow">
                        <canvas id="categoryBarChart"></canvas>
                    </div>
                </div>

                <!-- Donut Chart: Source Ratio -->
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex flex-col">
                    <h3 class="text-sm font-bold text-stone-700 uppercase tracking-wide mb-4">Overall Origin Ratio</h3>
                    <div class="chart-container flex-grow">
                        <canvas id="sourcePieChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <!-- Detailed Data Table -->
        <section class="bg-white rounded-xl border border-stone-200 shadow-sm overflow-hidden">
            <div class="p-5 border-b border-stone-100">
                <h2 class="text-xl font-bold text-stone-800">Category Detail Tracking</h2>
                <p class="text-stone-500 text-sm mt-1">
                    Comprehensive, row-level metrics comparing actual defect percentages against maximum allowable targets.
                </p>
            </div>
            <div class="table-scroll overflow-x-auto w-full">
                <table class="w-full text-left border-collapse whitespace-nowrap">
                    <thead>
                        <tr class="bg-stone-50 text-stone-500 text-xs uppercase tracking-wider">
                            <th class="p-4 font-semibold border-b border-stone-200">Month</th>
                            <th class="p-4 font-semibold border-b border-stone-200">Category</th>
                            <th class="p-4 font-semibold border-b border-stone-200 text-right">Prod. Qty</th>
                            <th class="p-4 font-semibold border-b border-stone-200 text-right">Cust. Qty</th>
                            <th class="p-4 font-semibold border-b border-stone-200 text-right">Total</th>
                            <th class="p-4 font-semibold border-b border-stone-200 text-right">Actual %</th>
                            <th class="p-4 font-semibold border-b border-stone-200 text-right">Target %</th>
                            <th class="p-4 font-semibold border-b border-stone-200">Status</th>
                        </tr>
                    </thead>
                    <tbody id="dataTableBody" class="text-sm text-stone-700 divide-y divide-stone-100">
                        <!-- Populated by JS -->
                    </tbody>
                </table>
            </div>
        </section>

    </div>

    <script>
        // 1. Data Storage: Extracted and structured from the provided Source Report CSV snippet
        const rawData = [
            { month: "Jan", category: "Bubbles", prodQty: 45, custQty: 2, totalQty: 47, actualPct: 31.76, targetPct: 3.00, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Jan", category: "Scratches", prodQty: 12, custQty: 8, totalQty: 20, actualPct: 13.51, targetPct: 1.50, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Jan", category: "Cateer", prodQty: 80, custQty: 1, totalQty: 81, actualPct: 54.73, targetPct: 0.50, status: "🔴 CRITICAL", rawStatus: "CRITICAL" },
            { month: "Feb", category: "Bluetooth Connection Issue", prodQty: 5, custQty: 12, totalQty: 17, actualPct: 80.95, targetPct: 1.00, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Feb", category: "Distortion", prodQty: 4, custQty: 0, totalQty: 4, actualPct: 19.05, targetPct: 2.00, status: "🟢 OK", rawStatus: "OK" }
        ];

        // 2. State Management
        const state = {
            selectedMonth: 'All',
            filteredData: [...rawData]
        };

        // Chart instances
        let barChartInstance = null;
        let pieChartInstance = null;

        // 3. Initialization & Core Logic
        document.addEventListener('DOMContentLoaded', () => {
            initializeFilter();
            updateDashboard(); // Initial render

            // Event Listener for the filter
            document.getElementById('monthFilter').addEventListener('change', (e) => {
                state.selectedMonth = e.target.value;
                if (state.selectedMonth === 'All') {
                    state.filteredData = [...rawData];
                } else {
                    state.filteredData = rawData.filter(item => item.month === state.selectedMonth);
                }
                updateDashboard();
            });
        });

        // Populate dropdown with unique months
        function initializeFilter() {
            const filterEl = document.getElementById('monthFilter');
            const months = [...new Set(rawData.map(item => item.month))];
            
            months.forEach(month => {
                const option = document.createElement('option');
                option.value = month;
                option.textContent = month;
                filterEl.appendChild(option);
            });
        }

        // Master update function
        function updateDashboard() {
            renderKPIs();
            renderTable();
            updateCharts();
        }

        // 4. Render Functions

        function renderKPIs() {
            const container = document.getElementById('kpiContainer');
            
            let totalDefects = 0;
            let criticalCount = 0;
            let overCount = 0;
            let okCount = 0;

            state.filteredData.forEach(item => {
                totalDefects += item.totalQty;
                if (item.rawStatus === 'CRITICAL') criticalCount++;
                if (item.rawStatus === 'OVER') overCount++;
                if (item.rawStatus === 'OK') okCount++;
            });

            const issuesCount = criticalCount + overCount;

            container.innerHTML = `
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-stone-100 flex items-center justify-center text-xl">📦</div>
                    <div>
                        <p class="text-sm font-medium text-stone-500">Total Defects</p>
                        <p class="text-2xl font-bold text-stone-900">${totalDefects}</p>
                    </div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-red-50 flex items-center justify-center text-xl">⚠️</div>
                    <div>
                        <p class="text-sm font-medium text-stone-500">Critical Categories</p>
                        <p class="text-2xl font-bold text-red-700">${criticalCount}</p>
                    </div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-orange-50 flex items-center justify-center text-xl">📈</div>
                    <div>
                        <p class="text-sm font-medium text-stone-500">Over Target</p>
                        <p class="text-2xl font-bold text-orange-700">${overCount}</p>
                    </div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-green-50 flex items-center justify-center text-xl">✅</div>
                    <div>
                        <p class="text-sm font-medium text-stone-500">Within Target (OK)</p>
                        <p class="text-2xl font-bold text-green-700">${okCount}</p>
                    </div>
                </div>
            `;
        }

        function renderTable() {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = '';

            if (state.filteredData.length === 0) {
                tbody.innerHTML = `<tr><td colspan="8" class="p-8 text-center text-stone-500">No data available for the selected period.</td></tr>`;
                return;
            }

            state.filteredData.forEach(row => {
                let badgeClass = '';
                if (row.rawStatus === 'CRITICAL') badgeClass = 'badge-critical';
                else if (row.rawStatus === 'OVER') badgeClass = 'badge-over';
                else if (row.rawStatus === 'OK') badgeClass = 'badge-ok';

                const tr = document.createElement('tr');
                tr.className = 'hover:bg-stone-50 transition-colors';
                tr.innerHTML = `
                    <td class="p-4 font-medium text-stone-900">${row.month}</td>
                    <td class="p-4 text-stone-800">${row.category}</td>
                    <td class="p-4 text-right text-stone-600">${row.prodQty}</td>
                    <td class="p-4 text-right text-stone-600">${row.custQty}</td>
                    <td class="p-4 text-right font-bold text-stone-800">${row.totalQty}</td>
                    <td class="p-4 text-right font-medium ${row.actualPct > row.targetPct ? 'text-red-600' : 'text-stone-800'}">${row.actualPct.toFixed(2)}%</td>
                    <td class="p-4 text-right text-stone-500">${row.targetPct.toFixed(2)}%</td>
                    <td class="p-4"><span class="badge ${badgeClass}">${row.status}</span></td>
                `;
                tbody.appendChild(tr);
            });
        }

        function updateCharts() {
            // Aggregate data for charts based on filtered state
            const categoryLabels = [];
            const prodData = [];
            const custData = [];
            
            let totalProd = 0;
            let totalCust = 0;

            // Group by category (in case a category appears multiple times in different months when 'All' is selected)
            const aggregatedCategories = {};

            state.filteredData.forEach(item => {
                totalProd += item.prodQty;
                totalCust += item.custQty;

                if (!aggregatedCategories[item.category]) {
                    aggregatedCategories[item.category] = { prod: 0, cust: 0 };
                }
                aggregatedCategories[item.category].prod += item.prodQty;
                aggregatedCategories[item.category].cust += item.custQty;
            });

            for (const [cat, counts] of Object.entries(aggregatedCategories)) {
                // Label wrapping logic (truncate or break if > 16 chars, though Chart.js handles arrays as multiline)
                let finalLabel = cat;
                if (cat.length > 16) {
                    // Split into two lines if it's too long, primarily handling "Bluetooth Connection Issue"
                    const words = cat.split(' ');
                    if(words.length > 1) {
                        finalLabel = [words.slice(0, Math.ceil(words.length/2)).join(' '), words.slice(Math.ceil(words.length/2)).join(' ')];
                    } else {
                        finalLabel = cat.substring(0, 13) + '...';
                    }
                }

                categoryLabels.push(finalLabel);
                prodData.push(counts.prod);
                custData.push(counts.cust);
            }

            // Common Tooltip Config
            const tooltipConfig = {
                backgroundColor: 'rgba(28, 25, 23, 0.95)',
                titleFont: { size: 13, family: "'Segoe UI', sans-serif" },
                bodyFont: { size: 13, family: "'Segoe UI', sans-serif" },
                padding: 12,
                cornerRadius: 8,
                displayColors: true
            };

            // 1. Update or Create Bar Chart
            const barCtx = document.getElementById('categoryBarChart').getContext('2d');
            if (barChartInstance) {
                barChartInstance.data.labels = categoryLabels;
                barChartInstance.data.datasets[0].data = prodData;
                barChartInstance.data.datasets[1].data = custData;
                barChartInstance.update();
            } else {
                barChartInstance = new Chart(barCtx, {
                    type: 'bar',
                    data: {
                        labels: categoryLabels,
                        datasets: [
                            {
                                label: 'Production Defect Qty',
                                data: prodData,
                                backgroundColor: '#94a3b8', // Slate 400
                                borderRadius: {topLeft: 0, topRight: 0, bottomLeft: 4, bottomRight: 4},
                                borderSkipped: false
                            },
                            {
                                label: 'Customer Defect Qty',
                                data: custData,
                                backgroundColor: '#d97706', // Amber 600
                                borderRadius: {topLeft: 4, topRight: 4, bottomLeft: 0, bottomRight: 0},
                                borderSkipped: false
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false, // CRITICAL for chart-container sizing
                        plugins: {
                            legend: {
                                position: 'bottom',
                                labels: { usePointStyle: true, boxWidth: 8, font: { family: "'Segoe UI', sans-serif" } }
                            },
                            tooltip: tooltipConfig
                        },
                        scales: {
                            x: {
                                stacked: true,
                                grid: { display: false }
                            },
                            y: {
                                stacked: true,
                                beginAtZero: true,
                                border: { display: false },
                                grid: { color: '#f5f5f4' }
                            }
                        }
                    }
                });
            }

            // 2. Update or Create Pie/Donut Chart
            const pieCtx = document.getElementById('sourcePieChart').getContext('2d');
            if (pieChartInstance) {
                pieChartInstance.data.datasets[0].data = [totalProd, totalCust];
                pieChartInstance.update();
            } else {
                pieChartInstance = new Chart(pieCtx, {
                    type: 'doughnut',
                    data: {
                        labels: ['Production', 'Customer'],
                        datasets: [{
                            data: [totalProd, totalCust],
                            backgroundColor: ['#94a3b8', '#d97706'],
                            borderWidth: 0,
                            hoverOffset: 4
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false, // CRITICAL for chart-container sizing
                        cutout: '65%',
                        plugins: {
                            legend: {
                                position: 'bottom',
                                labels: { usePointStyle: true, boxWidth: 8, font: { family: "'Segoe UI', sans-serif" } }
                            },
                            tooltip: tooltipConfig
                        }
                    }
                });
            }
        }
    </script>
</body>
</html>
