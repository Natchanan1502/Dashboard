<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quality Defect Tracking Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Plugin for data labels -->
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: #000000;
            color: #e2e8f0;
        }
        
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 100%;
            margin-left: auto;
            margin-right: auto;
            height: 420px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .table-scroll::-webkit-scrollbar {
            height: 8px;
        }
        .table-scroll::-webkit-scrollbar-track {
            background: #0a0a0a; 
            border-radius: 4px;
        }
        .table-scroll::-webkit-scrollbar-thumb {
            background: #333333; 
            border-radius: 4px;
        }
        .table-scroll::-webkit-scrollbar-thumb:hover {
            background: #444444; 
        }

        .dark-card {
            background-color: #0a0a0a;
            border: 1px solid #1f1f1f;
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
        .badge-critical { background-color: #450a0a; color: #fecaca; border: 1px solid #7f1d1d; }
        .badge-over { background-color: #431407; color: #fed7aa; border: 1px solid #7c2d12; }
        .badge-ok { background-color: #022c22; color: #d1fae5; border: 1px solid #064e3b; }

        /* Custom Table Styling for Black Background */
        .custom-table {
            background-color: #000000;
        }
        .custom-table thead tr {
            background-color: #0a0a0a;
        }
        .custom-table th {
            border-bottom: 1px solid #1f1f1f;
            color: #71717a;
        }
        .custom-table td {
            border-bottom: 1px solid #111111;
        }
        .custom-table tr:hover {
            background-color: #0a0a0a;
        }
    </style>
</head>
<body class="antialiased min-h-screen pb-12">

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
        
        <header class="mb-8 flex flex-col md:flex-row md:items-end justify-between gap-4 border-b border-zinc-800 pb-6">
            <div>
                <h1 class="text-3xl font-bold text-white tracking-tight mb-2">🏭 Quality Defect Tracker</h1>
                <p class="text-zinc-400 max-w-2xl">
                    Data analysis for <strong>Quality Defect Tracking</strong>. Full-width comparison of defect volumes against target thresholds in dark mode.
                </p>
            </div>
            <div class="flex flex-col gap-1 min-w-[200px]">
                <label for="monthFilter" class="text-sm font-semibold text-zinc-500 uppercase tracking-wider">📅 Filter by Month</label>
                <select id="monthFilter" class="bg-zinc-950 border border-zinc-800 text-white text-base rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 shadow-sm cursor-pointer outline-none transition-all">
                    <option value="All">All Months</option>
                </select>
            </div>
        </header>

        <!-- KPI Section -->
        <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-8" id="kpiContainer">
            <!-- Populated by JS -->
        </section>

        <!-- Charts Section -->
        <section class="mb-8">
            <div class="mb-4">
                <h2 class="text-xl font-bold text-zinc-100">Volume vs. Target Threshold</h2>
                <p class="text-zinc-500 text-sm mt-1">
                    Comparison of defect sources with direct quantity labels. The <span class="text-red-400 font-bold">--- Red Dash Line</span> represents the maximum quality threshold (Right Axis).
                </p>
            </div>
            
            <div class="dark-card p-5 rounded-xl shadow-sm flex flex-col w-full">
                <h3 class="text-sm font-bold text-zinc-400 uppercase tracking-wide mb-4">Defect Qty VS Target</h3>
                <div class="chart-container flex-grow">
                    <canvas id="categoryBarChart"></canvas>
                </div>
            </div>
        </section>

        <!-- Table Section -->
        <section class="dark-card rounded-xl shadow-sm overflow-hidden bg-black">
            <div class="p-5 border-b border-zinc-800 bg-zinc-950">
                <h2 class="text-xl font-bold text-zinc-100">Defect Data Log</h2>
            </div>
            <div class="table-scroll overflow-x-auto w-full">
                <table class="w-full text-left border-collapse whitespace-nowrap custom-table">
                    <thead>
                        <tr class="text-zinc-500 text-xs uppercase tracking-wider">
                            <th class="p-4 font-semibold">Month</th>
                            <th class="p-4 font-semibold">Category</th>
                            <th class="p-4 font-semibold text-right">Prod. Qty</th>
                            <th class="p-4 font-semibold text-right">Cust. Qty</th>
                            <th class="p-4 font-semibold text-right">Total</th>
                            <th class="p-4 font-semibold text-right">Actual %</th>
                            <th class="p-4 font-semibold text-right">Target %</th>
                            <th class="p-4 font-semibold">Status</th>
                        </tr>
                    </thead>
                    <tbody id="dataTableBody" class="text-sm text-zinc-300">
                        <!-- Populated by JS -->
                    </tbody>
                </table>
            </div>
        </section>
    </div>

    <script>
        Chart.register(ChartDataLabels);

        const rawData = [
            { month: "Jan", category: "Bubbles", prodQty: 45, custQty: 2, totalQty: 47, actualPct: 43.12, targetPct: 5.00, status: "Critical" },
            { month: "Jan", category: "Scratches", prodQty: 12, custQty: 8, totalQty: 20, actualPct: 18.35, targetPct: 12.00, status: "Over" },
            { month: "Jan", category: "Cateer", prodQty: 20, custQty: 1, totalQty: 21, actualPct: 19.27, targetPct: 5.00, status: "Over" },
            { month: "Jan", category: "Bluetooth Connection Issue", prodQty: 5, custQty: 12, totalQty: 17, actualPct: 15.60, targetPct: 7.00, status: "Over" },
            { month: "Jan", category: "Distortion", prodQty: 4, custQty: 0, totalQty: 4, actualPct: 3.67, targetPct: 10.00, status: "OK" },
            { month: "Feb", category: "Bubbles", prodQty: 10, custQty: 20, totalQty: 30, actualPct: 13.70, targetPct: 5.00, status: "Over" },
            { month: "Feb", category: "Scratches", prodQty: 2, custQty: 1, totalQty: 3, actualPct: 1.37, targetPct: 12.00, status: "OK" },
            { month: "Feb", category: "Cateer", prodQty: 40, custQty: 60, totalQty: 100, actualPct: 45.66, targetPct: 5.00, status: "Critical" },
            { month: "Feb", category: "Bluetooth Connection Issue", prodQty: 30, custQty: 50, totalQty: 80, actualPct: 36.53, targetPct: 7.00, status: "Critical" },
            { month: "Feb", category: "Distortion", prodQty: 5, custQty: 1, totalQty: 6, actualPct: 2.74, targetPct: 10.00, status: "OK" }
        ];

        const state = {
            selectedMonth: 'All',
            filteredData: [...rawData]
        };

        let barChartInstance = null;

        const THEME = {
            primary: '#3b82f6',
            secondary: '#60a5fa',
            target: '#f87171',
            grid: '#1f1f1f',
            text: '#94a3b8'
        };

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
                if (item.status === 'Critical') criticalCount++;
                if (item.status === 'Over') overCount++;
                if (item.status === 'OK') okCount++;
            });

            container.innerHTML = `
                <div class="bg-blue-900/20 p-5 rounded-xl border border-blue-900/40 shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-blue-600 flex items-center justify-center text-white text-xl">📦</div>
                    <div><p class="text-sm font-medium text-blue-400">Total Defects</p><p class="text-2xl font-bold text-white">${totalDefects}</p></div>
                </div>
                <div class="dark-card p-5 rounded-xl shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-red-900/20 border border-red-900/40 flex items-center justify-center text-xl">⚠️</div>
                    <div><p class="text-sm font-medium text-zinc-500">Critical</p><p class="text-2xl font-bold text-red-500">${criticalCount}</p></div>
                </div>
                <div class="dark-card p-5 rounded-xl shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-orange-900/20 border border-orange-900/40 flex items-center justify-center text-xl">📈</div>
                    <div><p class="text-sm font-medium text-zinc-500">Over Target</p><p class="text-2xl font-bold text-orange-500">${overCount}</p></div>
                </div>
                <div class="dark-card p-5 rounded-xl shadow-sm flex items-center gap-4">
                    <div class="w-12 h-12 rounded-full bg-emerald-900/20 border border-emerald-900/40 flex items-center justify-center text-xl">✅</div>
                    <div><p class="text-sm font-medium text-zinc-500">Target OK</p><p class="text-2xl font-bold text-emerald-500">${okCount}</p></div>
                </div>
            `;
        }

        function renderTable() {
            const tbody = document.getElementById('dataTableBody');
            tbody.innerHTML = '';

            state.filteredData.forEach(row => {
                const statusNorm = row.status.toUpperCase();
                let badgeClass = statusNorm === 'CRITICAL' ? 'badge-critical' : (statusNorm === 'OVER' ? 'badge-over' : 'badge-ok');
                
                const tr = document.createElement('tr');
                tr.className = 'transition-colors';
                tr.innerHTML = `
                    <td class="p-4 font-medium text-zinc-100">${row.month}</td>
                    <td class="p-4 text-zinc-300">${row.category}</td>
                    <td class="p-4 text-right text-zinc-400">${row.prodQty}</td>
                    <td class="p-4 text-right text-zinc-400">${row.custQty}</td>
                    <td class="p-4 text-right font-bold text-zinc-100">${row.totalQty}</td>
                    <td class="p-4 text-right font-medium ${row.actualPct > row.targetPct ? 'text-red-400' : 'text-zinc-200'}">${row.actualPct.toFixed(2)}%</td>
                    <td class="p-4 text-right text-zinc-500 font-bold">${row.targetPct.toFixed(2)}%</td>
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

            const aggregated = {};
            state.filteredData.forEach(item => {
                if (!aggregated[item.category]) aggregated[item.category] = { prod: 0, cust: 0, target: item.targetPct };
                aggregated[item.category].prod += item.prodQty;
                aggregated[item.category].cust += item.custQty;
            });

            for (const [cat, val] of Object.entries(aggregated)) {
                categoryLabels.push(cat.length > 25 ? cat.substring(0, 22) + '...' : cat);
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
                            { 
                                label: 'Production Defect', 
                                data: prodData, 
                                backgroundColor: THEME.primary, 
                                borderRadius: 4,
                                yAxisID: 'y',
                                datalabels: {
                                    anchor: 'end',
                                    align: 'top',
                                    offset: 4,
                                    color: '#ffffff',
                                    font: { weight: 'bold', size: 11 }
                                }
                            },
                            { 
                                label: 'Customer Defect', 
                                data: custData, 
                                backgroundColor: THEME.secondary, 
                                borderRadius: 4,
                                yAxisID: 'y',
                                datalabels: {
                                    anchor: 'end',
                                    align: 'top',
                                    offset: 4,
                                    color: '#cbd5e1',
                                    font: { weight: 'bold', size: 11 }
                                }
                            },
                            { 
                                label: 'Max Target Threshold %', 
                                data: targetData, 
                                type: 'line', 
                                borderColor: THEME.target, 
                                borderWidth: 3, 
                                pointBackgroundColor: THEME.target,
                                borderDash: [6, 4],
                                tension: 0.1, 
                                yAxisID: 'y1',
                                fill: false,
                                order: -1,
                                datalabels: { display: false }
                            }
                        ]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        layout: { padding: { top: 30 } },
                        plugins: { 
                            legend: { 
                                position: 'bottom', 
                                labels: { 
                                    usePointStyle: true, 
                                    padding: 25,
                                    color: '#cbd5e1'
                                } 
                            },
                            tooltip: { enabled: true }
                        },
                        scales: {
                            y: { 
                                beginAtZero: true, 
                                title: { display: true, text: 'Defect Qty', color: THEME.text },
                                grid: { color: THEME.grid },
                                ticks: { color: THEME.text }
                            },
                            y1: { 
                                beginAtZero: true, 
                                position: 'right', 
                                title: { display: true, text: 'Threshold %', color: THEME.target },
                                grid: { drawOnChartArea: false },
                                ticks: { 
                                    color: THEME.target,
                                    callback: function(value) { return value + '%'; } 
                                }
                            },
                            x: { 
                                grid: { display: false },
                                ticks: { 
                                    color: THEME.text,
                                    font: { weight: '600' } 
                                }
                            }
                        }
                    }
                });
            }
        }
    </script>
</body>
</html>>
