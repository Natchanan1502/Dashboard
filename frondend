<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quality Defect Tracking Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: #fcfbf9;
            color: #292524;
        }
        
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

    <!-- Chosen Palette: Warm Neutrals with Semantic Accents -->
    <!-- Application Structure Plan: A single-page dashboard layout. Chosen for efficient quality monitoring. Key update: Added a 'Target' line to the bar chart to provide immediate visual context for performance against quality thresholds. -->
    <!-- Visualization & Content Choices: 
         1. KPIs (HTML/CSS): Core metrics.
         2. Mixed Bar/Line Chart (Chart.js): Goal: Compare volume while observing targets. Bars show absolute defect counts, while the line shows the Target % threshold on a secondary axis.
         3. Doughnut Chart (Chart.js): Overall source distribution.
         4. Data Table (HTML): Granular record tracking.
    -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        
        <header class="mb-8 flex flex-col md:flex-row md:items-end justify-between gap-4 border-b border-stone-200 pb-6">
            <div>
                <h1 class="text-3xl font-bold text-stone-900 tracking-tight mb-2">🏭 Quality Defect Tracker</h1>
                <p class="text-stone-500 max-w-2xl">
                    Monitoring production and customer defects. The chart below now includes a <span class="text-red-600 font-bold">— Target Line</span> for percentage comparison.
                </p>
            </div>
            <div class="flex flex-col gap-1 min-w-[200px]">
                <label for="monthFilter" class="text-sm font-semibold text-stone-600 uppercase tracking-wider">📅 Filter by Month</label>
                <select id="monthFilter" class="bg-white border border-stone-300 text-stone-900 text-base rounded-lg focus:ring-amber-500 focus:border-amber-500 block w-full p-2.5 shadow-sm cursor-pointer outline-none">
                    <option value="All">All Months</option>
                </select>
            </div>
        </header>

        <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-8" id="kpiContainer">
            <!-- Populated by JS -->
        </section>

        <section class="mb-8">
            <div class="mb-4">
                <h2 class="text-xl font-bold text-stone-800">Defect Analysis & Thresholds</h2>
                <p class="text-stone-500 text-sm mt-1">
                    The bars represent total defect quantities. The red line indicates the <strong>Target % Threshold</strong> for that category.
                </p>
            </div>
            
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm lg:col-span-2 flex flex-col">
                    <h3 class="text-sm font-bold text-stone-700 uppercase tracking-wide mb-4">Volume vs. Target Threshold</h3>
                    <div class="chart-container flex-grow">
                        <canvas id="categoryBarChart"></canvas>
                    </div>
                </div>

                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex flex-col">
                    <h3 class="text-sm font-bold text-stone-700 uppercase tracking-wide mb-4">Overall Origin Ratio</h3>
                    <div class="chart-container flex-grow">
                        <canvas id="sourcePieChart"></canvas>
                    </div>
                </div>
            </div>
        </section>

        <section class="bg-white rounded-xl border border-stone-200 shadow-sm overflow-hidden">
            <div class="p-5 border-b border-stone-100">
                <h2 class="text-xl font-bold text-stone-800">Category Detail Tracking</h2>
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
        const rawData = [
            { month: "Jan", category: "Bubbles", prodQty: 45, custQty: 2, totalQty: 47, actualPct: 31.76, targetPct: 3.00, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Jan", category: "Scratches", prodQty: 12, custQty: 8, totalQty: 20, actualPct: 13.51, targetPct: 1.50, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Jan", category: "Cateer", prodQty: 80, custQty: 1, totalQty: 81, actualPct: 54.73, targetPct: 0.50, status: "🔴 CRITICAL", rawStatus: "CRITICAL" },
            { month: "Feb", category: "Bluetooth Connection Issue", prodQty: 5, custQty: 12, totalQty: 17, actualPct: 80.95, targetPct: 1.00, status: "🔴 OVER", rawStatus: "OVER" },
            { month: "Feb", category: "Distortion", prodQty: 4, custQty: 0, totalQty: 4, actualPct: 19.05, targetPct: 2.00, status: "🟢 OK", rawStatus: "OK" }
        ];

        const state = {
            selectedMonth: 'All',
            filteredData: [...rawData]
        };

        let barChartInstance = null;
        let pieChartInstance = null;

        document.addEventListener('DOMContentLoaded', () => {
            initializeFilter();
            updateDashboard();

            document.getElementById('monthFilter').addEventListener('change', (e) => {
                state.selectedMonth = e.target.value;
                state.filteredData = state.selectedMonth === 'All' 
                    ? [...rawData] 
                    : rawData.filter(item => item.month === state.selectedMonth);
                updateDashboard();
            });
        });

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

        function updateDashboard() {
            renderKPIs();
            renderTable();
            updateCharts();
        }

        function renderKPIs() {
            const container = document.getElementById('kpiContainer');
            let totalDefects = 0, criticalCount = 0, overCount = 0, okCount = 0;

            state.filteredData.forEach(item => {
                totalDefects += item.totalQty;
                if (item.rawStatus === 'CRITICAL') criticalCount++;
                if (item.rawStatus === 'OVER') overCount++;
                if (item.rawStatus === 'OK') okCount++;
            });

            container.innerHTML = `
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-stone-100 flex items-center justify-center text-xl">📦</div>
                    <div><p class="text-sm font-medium text-stone-500">Total Defects</p><p class="text-2xl font-bold text-stone-900">${totalDefects}</p></div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-red-50 flex items-center justify-center text-xl">⚠️</div>
                    <div><p class="text-sm font-medium text-stone-500">Critical</p><p class="text-2xl font-bold text-red-700">${criticalCount}</p></div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-orange-50 flex items-center justify-center text-xl">📈</div>
                    <div><p class="text-sm font-medium text-stone-500">Over Target</p><p class="text-2xl font-bold text-orange-700">${overCount}</p></div>
                </div>
                <div class="bg-white p-5 rounded-xl border border-stone-200 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-green-50 flex items-center justify-center text-xl">✅</div>
                    <div><p class="text-sm font-medium text-stone-500">Target OK</p><p class="text-2xl font-bold text-green-700">${okCount}</p></div>
                </div>
            `;
        }

        function renderTable() {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = state.filteredData.length === 0 
                ? '<tr><td colspan="8" class="p-8 text-center text-stone-500">No data.</td></tr>' 
                : '';

            state.filteredData.forEach(row => {
                let badgeClass = row.rawStatus === 'CRITICAL' ? 'badge-critical' : (row.rawStatus === 'OVER' ? 'badge-over' : 'badge-ok');
                const tr = document.createElement('tr');
                tr.className = 'hover:bg-stone-50 transition-colors';
                tr.innerHTML = `
                    <td class="p-4 font-medium text-stone-900">${row.month}</td>
                    <td class="p-4 text-stone-800">${row.category}</td>
                    <td class="p-4 text-right text-stone-600">${row.prodQty}</td>
                    <td class="p-4 text-right text-stone-600">${row.custQty}</td>
                    <td class="p-4 text-right font-bold text-stone-800">${row.totalQty}</td>
                    <td class="p-4 text-right font-medium ${row.actualPct > row.targetPct ? 'text-red-600' : 'text-stone-800'}">${row.actualPct.toFixed(2)}%</td>
                    <td class="p-4 text-right text-stone-500 font-bold">${row.targetPct.toFixed(2)}%</td>
                    <td class="p-4"><span class="badge ${badgeClass}">${row.status}</span></td>
                `;
                tbody.appendChild(tr);
            });
        }

        function updateCharts() {
            const categoryLabels = [];
            const prodData = [];
            const custData = [];
            const targetData = [];
            let totalProd = 0, totalCust = 0;

            const aggregated = {};
            state.filteredData.forEach(item => {
                totalProd += item.prodQty; totalCust += item.custQty;
                if (!aggregated[item.category]) aggregated[item.category] = { prod: 0, cust: 0, target: item.targetPct };
                aggregated[item.category].prod += item.prodQty;
                aggregated[item.category].cust += item.custQty;
            });

            for (const [cat, val] of Object.entries(aggregated)) {
                categoryLabels.push(cat.length > 15 ? cat.substring(0, 12) + '...' : cat);
                prodData.push(val.prod);
                custData.push(val.cust);
                targetData.push(val.target);
            }

            const barCtx = document.getElementById('categoryBarChart').getContext('2d');
            if (barChartInstance) {
                barChartInstance.data.labels = categoryLabels;
                barChartInstance.data.datasets[0].data = prodData;
                barChartInstance.data.datasets[1].data = custData;
                barChartInstance.data.datasets[2].data = targetData;
                barChartInstance.update();
            } else {
                barChartInstance = new Chart(barCtx, {
                    type: 'bar',
                    data: {
                        labels: categoryLabels,
                        datasets: [
                            { label: 'Prod. Defect', data: prodData, backgroundColor: '#94a3b8', stack: 'stack0', yAxisID: 'y' },
                            { label: 'Cust. Defect', data: custData, backgroundColor: '#fbbf24', stack: 'stack0', yAxisID: 'y' },
                            { 
                                label: 'Max Target %', 
                                data: targetData, 
                                type: 'line', 
                                borderColor: '#ef4444', 
                                borderWidth: 3, 
                                pointBackgroundColor: '#ef4444',
                                tension: 0.1, 
                                yAxisID: 'y1',
                                fill: false
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: { legend: { position: 'bottom' } },
                        scales: {
                            y: { beginAtZero: true, title: { display: true, text: 'Defect Qty' } },
                            y1: { 
                                beginAtZero: true, 
                                position: 'right', 
                                title: { display: true, text: 'Target %' },
                                grid: { drawOnChartArea: false }
                            }
                        }
                    }
                });
            }

            const pieCtx = document.getElementById('sourcePieChart').getContext('2d');
            if (pieChartInstance) {
                pieChartInstance.data.datasets[0].data = [totalProd, totalCust];
                pieChartInstance.update();
            } else {
                pieChartInstance = new Chart(pieCtx, {
                    type: 'doughnut',
                    data: {
                        labels: ['Production', 'Customer'],
                        datasets: [{ data: [totalProd, totalCust], backgroundColor: ['#94a3b8', '#fbbf24'], borderWidth: 0 }]
                    },
                    options: { responsive: true, maintainAspectRatio: false, cutout: '70%', plugins: { legend: { position: 'bottom' } } }
                });
            }
        }
    </script>
</body>
</html>
