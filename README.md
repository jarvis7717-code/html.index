<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Advanced Multimodal Route Finder</title>
  
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- Leaflet CSS & JS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
  
  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <!-- Google Font: Inter -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">

  <style>
    body {
      font-family: 'Inter', sans-serif;
    }
    /* Custom scrollbars */
    ::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    ::-webkit-scrollbar-track {
      background: #f1f5f9;
    }
    ::-webkit-scrollbar-thumb {
      background: #cbd5e1;
      border-radius: 4px;
    }
    ::-webkit-scrollbar-thumb:hover {
      background: #94a3b8;
    }
    .leaflet-popup-content-wrapper {
      border-radius: 12px;
      padding: 4px;
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
    }
    .pulse-pin {
      animation: pulse-ring 2s infinite;
    }
    @keyframes pulse-ring {
      0% { transform: scale(0.95); opacity: 0.8; }
      50% { transform: scale(1.15); opacity: 0.4; }
      100% { transform: scale(0.95); opacity: 0.8; }
    }
  </style>
</head>
<body class="bg-slate-100 text-slate-800 antialiased h-screen flex flex-col overflow-hidden">

  <!-- Top App Navigation -->
  <header class="bg-white border-b border-slate-200 z-20 px-4 py-3 flex items-center justify-between shadow-sm flex-shrink-0">
    <div class="flex items-center space-x-3">
      <div class="bg-gradient-to-tr from-blue-600 to-indigo-600 p-2 rounded-xl text-white shadow-md shadow-blue-500/20">
        <i data-lucide="compass" class="w-6 h-6"></i>
      </div>
      <div>
        <h1 class="text-lg font-bold text-slate-900 tracking-tight flex items-center gap-2">
          Multimodal Route Finder 
          <span class="text-xs font-semibold px-2 py-0.5 rounded-full bg-blue-100 text-blue-700">OSM Multi-Hierarchy Engine</span>
        </h1>
        <p class="text-xs text-slate-500 hidden sm:block">Granular street exploration from highways down to narrow residential & colony alleys</p>
      </div>
    </div>
    <div class="flex items-center gap-2">
      <span class="inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-medium bg-emerald-50 text-emerald-700 border border-emerald-200">
        <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
        GIS Topology Active
      </span>
      <button id="toggleMobileSidebarBtn" class="md:hidden p-2 rounded-lg border border-slate-200 text-slate-600 hover:bg-slate-50">
        <i data-lucide="sliders-horizontal" class="w-5 h-5"></i>
      </button>
    </div>
  </header>

  <!-- Main Container -->
  <div class="flex flex-1 overflow-hidden relative">

    <!-- Left Controls Sidebar -->
    <aside id="sidebarPanel" class="w-full md:w-96 lg:w-[420px] bg-white border-r border-slate-200 flex flex-col h-full z-10 transition-transform duration-300 absolute md:relative -translate-x-full md:translate-x-0 shadow-lg md:shadow-none">
      <div class="p-4 border-b border-slate-100 flex items-center justify-between">
        <div class="flex items-center gap-2">
          <i data-lucide="route" class="w-5 h-5 text-blue-600"></i>
          <h2 class="font-bold text-slate-800 text-sm uppercase tracking-wider">Navigation Parameters</h2>
        </div>
        <button id="closeMobileSidebarBtn" class="md:hidden text-slate-400 hover:text-slate-600">
          <i data-lucide="x" class="w-5 h-5"></i>
        </button>
      </div>

      <div class="flex-1 overflow-y-auto p-4 space-x-0 space-y-4 text-sm">
        
        <!-- Waypoints Section -->
        <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200 space-y-3">
          <div class="flex items-center justify-between">
            <span class="text-xs font-semibold text-slate-500 tracking-wide uppercase">Waypoints</span>
            <button id="swapBtn" class="text-blue-600 hover:text-blue-700 text-xs flex items-center gap-1 font-medium transition">
              <i data-lucide="arrow-up-down" class="w-3.5 h-3.5"></i> Invert
            </button>
          </div>

          <!-- Origin -->
          <div class="flex items-start gap-2.5">
            <div class="mt-2 w-3 h-3 rounded-full bg-emerald-500 ring-4 ring-emerald-100 flex-shrink-0"></div>
            <div class="flex-1">
              <label class="block text-xs font-medium text-slate-600 mb-0.5">Origin</label>
              <input type="text" id="originInput" value="Connaught Place, New Delhi" class="w-full text-xs px-3 py-2 bg-white rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-600 font-medium text-slate-800">
            </div>
          </div>

          <!-- Optional Intermediate Stop -->
          <div id="stopsContainer" class="space-y-2">
            <div class="flex items-start gap-2.5">
              <div class="mt-2 w-3 h-3 rounded-full bg-amber-500 ring-4 ring-amber-100 flex-shrink-0"></div>
              <div class="flex-1">
                <div class="flex justify-between items-center mb-0.5">
                  <label class="block text-xs font-medium text-slate-600">Via Stop 1</label>
                  <button id="removeStopBtn" class="text-slate-400 hover:text-red-500 text-xs">Remove</button>
                </div>
                <input type="text" id="stopInput" value="Janpath Market Corridor" class="w-full text-xs px-3 py-2 bg-white rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-600 font-medium text-slate-800">
              </div>
            </div>
          </div>

          <!-- Destination -->
          <div class="flex items-start gap-2.5">
            <div class="mt-2 w-3 h-3 rounded-full bg-rose-500 ring-4 ring-rose-100 flex-shrink-0"></div>
            <div class="flex-1">
              <label class="block text-xs font-medium text-slate-600 mb-0.5">Destination</label>
              <input type="text" id="destInput" value="India Gate, New Delhi" class="w-full text-xs px-3 py-2 bg-white rounded-lg border border-slate-300 focus:outline-none focus:ring-2 focus:ring-blue-500/20 focus:border-blue-600 font-medium text-slate-800">
            </div>
          </div>
        </div>

        <!-- Mode of Transport -->
        <div>
          <label class="block text-xs font-semibold text-slate-500 uppercase tracking-wide mb-2">Transport Mode</label>
          <div class="grid grid-cols-4 gap-1.5" id="modeSelector">
            <button data-mode="car" class="mode-btn active flex flex-col items-center justify-center p-2 rounded-xl border border-blue-600 bg-blue-50 text-blue-700 transition">
              <i data-lucide="car" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Car</span>
            </button>
            <button data-mode="taxi" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="badge-dollar-sign" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Taxi/Cab</span>
            </button>
            <button data-mode="motorcycle" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="zap" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Motorbike</span>
            </button>
            <button data-mode="auto_rickshaw" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="navigation-2" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Auto</span>
            </button>
            <button data-mode="bicycle" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="bike" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Bicycle</span>
            </button>
            <button data-mode="walking" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="footprints" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Walking</span>
            </button>
            <button data-mode="bus" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="bus" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Bus</span>
            </button>
            <button data-mode="multimodal" class="mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition">
              <i data-lucide="train" class="w-4 h-4 mb-1"></i>
              <span class="text-[10px] font-semibold">Metro/Rail</span>
            </button>
          </div>
        </div>

        <!-- Engine Routing Constraints -->
        <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200 space-y-3">
          <div class="flex items-center justify-between">
            <span class="text-xs font-semibold text-slate-500 uppercase tracking-wide">Discovery Constraints</span>
            <span class="text-[11px] bg-slate-200 text-slate-700 px-2 py-0.5 rounded-md font-mono" id="altCountBadge">5 Paths</span>
          </div>
          
          <div>
            <div class="flex justify-between text-xs text-slate-600 mb-1">
              <span>Max Alternatives</span>
              <span id="maxAltsVal" class="font-semibold text-blue-600">5</span>
            </div>
            <input id="maxAltsSlider" type="range" min="2" max="6" value="5" class="w-full h-1.5 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-blue-600">
          </div>

          <div>
            <div class="flex justify-between text-xs text-slate-600 mb-1">
              <span>Corridor Breadth (Buffer)</span>
              <span id="corridorVal" class="font-semibold text-blue-600">3.5 km</span>
            </div>
            <input id="corridorSlider" type="range" min="1.0" max="6.0" step="0.5" value="3.5" class="w-full h-1.5 bg-slate-200 rounded-lg appearance-none cursor-pointer accent-blue-600">
          </div>
        </div>

        <!-- Vehicle & Economic Cost Model -->
        <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200 space-y-3">
          <span class="text-xs font-semibold text-slate-500 uppercase tracking-wide block">Cost & Fuel Parameters</span>
          <div class="grid grid-cols-2 gap-2">
            <div>
              <label class="block text-[11px] font-medium text-slate-600 mb-1">Fuel Price ($/L)</label>
              <input type="number" id="fuelPriceInput" value="1.25" step="0.05" class="w-full text-xs px-2.5 py-1.5 bg-white rounded-lg border border-slate-300 focus:outline-none focus:ring-1 focus:ring-blue-500">
            </div>
            <div>
              <label class="block text-[11px] font-medium text-slate-600 mb-1">Mileage (km/L)</label>
              <input type="number" id="mileageInput" value="14.0" step="0.5" class="w-full text-xs px-2.5 py-1.5 bg-white rounded-lg border border-slate-300 focus:outline-none focus:ring-1 focus:ring-blue-500">
            </div>
          </div>
        </div>

      </div>

      <!-- Action Footer -->
      <div class="p-4 border-t border-slate-200 bg-white">
        <button id="discoverBtn" class="w-full py-3 px-4 bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-700 hover:to-indigo-700 text-white font-semibold rounded-xl shadow-lg shadow-blue-500/25 flex items-center justify-center gap-2 transition transform active:scale-[0.98]">
          <i data-lucide="sparkles" class="w-4 h-4"></i>
          <span>Discover All Routes</span>
        </button>
      </div>
    </aside>

    <!-- Center/Right Dashboard & Map Area -->
    <main class="flex-1 flex flex-col h-full overflow-hidden bg-slate-100">
      
      <!-- Top Metrics Bar -->
      <div class="bg-white border-b border-slate-200 px-5 py-3 flex-shrink-0 grid grid-cols-2 sm:grid-cols-4 gap-3">
        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-blue-50 text-blue-600">
            <i data-lucide="layers" class="w-5 h-5"></i>
          </div>
          <div>
            <div class="text-[11px] font-medium text-slate-500 uppercase tracking-wider">Alternatives</div>
            <div id="statAlternatives" class="text-base font-bold text-slate-900">5 Options</div>
          </div>
        </div>

        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-emerald-50 text-emerald-600">
            <i data-lucide="timer" class="w-5 h-5"></i>
          </div>
          <div>
            <div class="text-[11px] font-medium text-slate-500 uppercase tracking-wider">Fastest Time</div>
            <div id="statFastestTime" class="text-base font-bold text-slate-900">7.2 min</div>
          </div>
        </div>

        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-purple-50 text-purple-600">
            <i data-lucide="ruler" class="w-5 h-5"></i>
          </div>
          <div>
            <div class="text-[11px] font-medium text-slate-500 uppercase tracking-wider">Shortest Dist</div>
            <div id="statShortestDist" class="text-base font-bold text-slate-900">3.4 km</div>
          </div>
        </div>

        <div class="flex items-center gap-3">
          <div class="p-2 rounded-lg bg-amber-50 text-amber-600">
            <i data-lucide="coins" class="w-5 h-5"></i>
          </div>
          <div>
            <div class="text-[11px] font-medium text-slate-500 uppercase tracking-wider">Lowest Cost</div>
            <div id="statLowestCost" class="text-base font-bold text-slate-900">$0.30</div>
          </div>
        </div>
      </div>

      <!-- Split Area: Map and Comparative Analytics -->
      <div class="flex-1 flex flex-col lg:flex-row overflow-hidden relative">
        
        <!-- Interactive Map Container -->
        <div class="w-full lg:w-7/12 h-64 lg:h-full relative flex-shrink-0">
          <div id="map" class="w-full h-full"></div>

          <!-- Floating Map Mode Layer Switcher -->
          <div class="absolute top-3 right-3 z-[1000] bg-white/95 backdrop-blur-sm rounded-xl p-1.5 shadow-md border border-slate-200 flex gap-1 text-xs">
            <button id="mapStyleOsm" class="px-2.5 py-1 rounded-lg font-medium bg-blue-600 text-white transition">OSM Street</button>
            <button id="mapStyleDark" class="px-2.5 py-1 rounded-lg font-medium text-slate-600 hover:bg-slate-100 transition">Dark Matter</button>
            <button id="mapStyleTopo" class="px-2.5 py-1 rounded-lg font-medium text-slate-600 hover:bg-slate-100 transition">Terrain</button>
          </div>

          <!-- Map Legend Badge -->
          <div class="absolute bottom-4 left-4 z-[1000] bg-white/95 backdrop-blur-sm rounded-xl p-2.5 shadow-lg border border-slate-200 hidden sm:block max-w-xs text-xs">
            <div class="font-bold text-slate-700 mb-1.5 flex items-center gap-1.5">
              <i data-lucide="map-pin" class="w-3.5 h-3.5 text-blue-600"></i> Route Traversal Legend
            </div>
            <div class="space-y-1 text-[11px]" id="mapLegendItems">
              <!-- Rendered via JS -->
            </div>
          </div>
        </div>

        <!-- Comparative Analytics & Inspection Column -->
        <div class="w-full lg:w-5/12 h-full flex flex-col bg-white border-l border-slate-200 overflow-y-auto">
          
          <!-- Route Comparison Matrix -->
          <div class="p-4 border-b border-slate-200">
            <div class="flex items-center justify-between mb-3">
              <h3 class="text-xs font-bold text-slate-700 uppercase tracking-wider flex items-center gap-1.5">
                <i data-lucide="git-compare" class="w-4 h-4 text-blue-600"></i>
                Multi-Route Comparison Matrix
              </h3>
              <span class="text-[11px] text-slate-500">Click to focus</span>
            </div>

            <div class="overflow-x-auto rounded-xl border border-slate-200">
              <table class="w-full text-left text-xs" id="routesTable">
                <thead class="bg-slate-50 text-slate-600 font-semibold border-b border-slate-200">
                  <tr>
                    <th class="py-2.5 px-3">Route Strategy</th>
                    <th class="py-2.5 px-2">Dist</th>
                    <th class="py-2.5 px-2">Time</th>
                    <th class="py-2.5 px-2">Turns</th>
                    <th class="py-2.5 px-3">Est. Cost</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-slate-100 text-slate-700 font-medium" id="routesTableBody">
                  <!-- Populated by JS -->
                </tbody>
              </table>
            </div>
          </div>

          <!-- Active Route Granular Analysis -->
          <div class="p-4 space-y-4 flex-1">
            <div class="flex items-center justify-between">
              <div>
                <span class="text-[11px] font-semibold text-slate-400 uppercase tracking-wider block">Selected Inspection</span>
                <h4 id="selectedRouteTitle" class="text-base font-bold text-slate-900 flex items-center gap-2">
                  <span id="routeColorPill" class="w-3 h-3 rounded-full bg-blue-600 inline-block"></span>
                  Fastest Route
                </h4>
              </div>
              <span id="selectedRouteBadge" class="text-xs px-2.5 py-1 rounded-full font-semibold bg-blue-50 text-blue-700 border border-blue-200">Arterial Corridor</span>
            </div>

            <!-- Granular Metric Badges -->
            <div class="grid grid-cols-3 gap-2 text-center">
              <div class="p-2.5 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-[10px] uppercase text-slate-500 font-semibold">Fuel Burn</div>
                <div id="routeFuelLitres" class="text-sm font-bold text-slate-800">0.24 L</div>
              </div>
              <div class="p-2.5 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-[10px] uppercase text-slate-500 font-semibold">Carbon Impact</div>
                <div id="routeCO2" class="text-sm font-bold text-slate-800">0.56 kg</div>
              </div>
              <div class="p-2.5 rounded-xl bg-slate-50 border border-slate-200">
                <div class="text-[10px] uppercase text-slate-500 font-semibold">Tolls Count</div>
                <div id="routeTollInfo" class="text-sm font-bold text-slate-800">0 Tolls</div>
              </div>
            </div>

            <!-- Road Category Composition (OSM Hierarchies) -->
            <div class="bg-slate-50 p-3.5 rounded-xl border border-slate-200 space-y-2.5">
              <div class="flex items-center justify-between text-xs">
                <span class="font-bold text-slate-700 flex items-center gap-1.5">
                  <i data-lucide="split" class="w-4 h-4 text-blue-600"></i> Road Classification Breakdown
                </span>
                <span class="text-[11px] text-slate-500">OSM Hierarchy</span>
              </div>
              
              <!-- Stacked visual bar -->
              <div class="h-3 w-full bg-slate-200 rounded-full overflow-hidden flex" id="roadHierarchyBar">
                <!-- Stacked sections injected via JS -->
              </div>

              <!-- Hierarchy Key / Details -->
              <div class="grid grid-cols-2 gap-x-3 gap-y-1.5 text-[11px] pt-1" id="roadHierarchyList">
                <!-- Injected via JS -->
              </div>
            </div>

            <!-- Step by Step Turn Navigation -->
            <div class="border border-slate-200 rounded-xl overflow-hidden">
              <button id="toggleTurnsBtn" class="w-full px-3.5 py-2.5 bg-slate-50 flex items-center justify-between text-xs font-bold text-slate-700 hover:bg-slate-100 transition">
                <span class="flex items-center gap-1.5">
                  <i data-lucide="milestone" class="w-4 h-4 text-blue-600"></i> Traversed Corridors & Turns
                </span>
                <i data-lucide="chevron-down" id="turnsChevron" class="w-4 h-4 transition-transform duration-200"></i>
              </button>
              <div id="turnsList" class="p-3 divide-y divide-slate-100 max-h-52 overflow-y-auto space-y-2 text-xs text-slate-600">
                <!-- Turn rows injected via JS -->
              </div>
            </div>

          </div>

        </div>

      </div>

    </main>
  </div>

  <!-- APPLICATION LOGIC -->
  <script>
    // Initialize Lucide Icons
    lucide.createIcons();

    // -------------------------------------------------------------------------
    // 1. DATASETS & ROUTE DEFINITIONS (Delhi Central Corridor Geometries)
    // -------------------------------------------------------------------------
    
    // Core Waypoint Coordinates
    const WAYPOINTS = {
      origin: [28.6315, 77.2167], // Connaught Place Central Circle
      stop: [28.6234, 77.2185],   // Janpath Market
      dest: [28.6129, 77.2295]    // India Gate Hexagon
    };

    // Realistic road-following node trajectories preserving hierarchies
    const ROUTE_DATASETS = [
      {
        id: "fastest",
        label: "Fastest Route",
        badge: "Arterial Corridor",
        color: "#2563eb", // Royal Blue
        distanceKm: 3.8,
        durationMin: 7.2,
        turnsCount: 4,
        tolls: "None",
        roadBreakdown: {
          primary: 2.3,       // Kasturba Gandhi Marg
          secondary: 1.1,     // Janpath Radial
          tertiary: 0.4
        },
        majorStreets: [
          { text: "Head south on Outer Circle toward Janpath", turn: "Straight", dist: "0.6 km" },
          { text: "Turn slightly left onto Kasturba Gandhi Marg", turn: "Left", dist: "1.7 km" },
          { text: "Merge onto C-Hexagon arterial corridor", turn: "Right", dist: "1.1 km" },
          { text: "Arrive at India Gate Outer Circle", turn: "Destination", dist: "0.4 km" }
        ],
        // Coordinates: Outer circle -> KG Marg -> C-Hexagon
        path: [
          [28.6315, 77.2167],
          [28.6292, 77.2178],
          [28.6270, 77.2212],
          [28.6235, 77.2238],
          [28.6190, 77.2265],
          [28.6152, 77.2280],
          [28.6129, 77.2295]
        ]
      },
      {
        id: "shortest",
        label: "Shortest Distance",
        badge: "Direct Janpath Line",
        color: "#16a34a", // Green
        distanceKm: 3.4,
        durationMin: 8.5,
        turnsCount: 5,
        tolls: "None",
        roadBreakdown: {
          primary: 1.8,
          secondary: 1.2,
          residential: 0.4
        },
        majorStreets: [
          { text: "Head directly south down Janpath Road", turn: "Straight", dist: "1.5 km" },
          { text: "Cross Rajpath crossing junction", turn: "Straight", dist: "0.9 km" },
          { text: "Turn onto Maulana Azad Road local link", turn: "Left", dist: "0.6 km" },
          { text: "Join Man Singh Road toward India Gate", turn: "Right", dist: "0.4 km" }
        ],
        path: [
          [28.6315, 77.2167],
          [28.6275, 77.2180],
          [28.6234, 77.2185],
          [28.6180, 77.2195],
          [28.6145, 77.2240],
          [28.6129, 77.2295]
        ]
      },
      {
        id: "local_streets",
        label: "Local & Colony Streets",
        badge: "Low Speed / Neighborhood",
        color: "#ea580c", // Orange
        distanceKm: 4.1,
        durationMin: 11.4,
        turnsCount: 9,
        tolls: "None",
        roadBreakdown: {
          secondary: 0.8,
          residential: 1.9,
          service: 0.9,
          living_street: 0.5
        },
        majorStreets: [
          { text: "Exit Connaught Place via Tolstoy Lane alleys", turn: "Left", dist: "0.5 km" },
          { text: "Navigate Atul Grove Road residential sector", turn: "Right", dist: "0.8 km" },
          { text: "Pass through Ferozeshah Road government quarters", turn: "Left", dist: "1.2 km" },
          { text: "Take Copernicus Lane bypass corridor", turn: "Right", dist: "1.1 km" },
          { text: "Approach India Gate round-about via service slip", turn: "Left", dist: "0.5 km" }
        ],
        path: [
          [28.6315, 77.2167],
          [28.6285, 77.2140],
          [28.6260, 77.2155],
          [28.6240, 77.2205],
          [28.6215, 77.2220],
          [28.6185, 77.2240],
          [28.6160, 77.2270],
          [28.6129, 77.2295]
        ]
      },
      {
        id: "highway_bias",
        label: "Highway / Arterial Bias",
        badge: "Barakhamba Trunk",
        color: "#9333ea", // Purple
        distanceKm: 4.6,
        durationMin: 7.9,
        turnsCount: 3,
        tolls: "None",
        roadBreakdown: {
          trunk: 2.8,
          primary: 1.4,
          secondary: 0.4
        },
        majorStreets: [
          { text: "Take Barakhamba Expressway radial corridor", turn: "Straight", dist: "1.8 km" },
          { text: "Merge into Mandi House traffic rotary", turn: "Slight Right", dist: "0.8 km" },
          { text: "Follow Tilak Marg high-speed multilane highway", turn: "Straight", dist: "1.6 km" },
          { text: "Enter C-Hexagon main carriage entrance", turn: "Slight Left", dist: "0.4 km" }
        ],
        path: [
          [28.6315, 77.2167],
          [28.6300, 77.2230],
          [28.6255, 77.2340],
          [28.6200, 77.2330],
          [28.6165, 77.2315],
          [28.6129, 77.2295]
        ]
      },
      {
        id: "toll_free",
        label: "Toll-Free Alternative",
        badge: "Surface Free-Way",
        color: "#0891b2", // Cyan/Teal
        distanceKm: 3.9,
        durationMin: 8.8,
        turnsCount: 6,
        tolls: "$0.00 Guaranteed",
        roadBreakdown: {
          primary: 1.6,
          secondary: 1.4,
          tertiary: 0.9
        },
        majorStreets: [
          { text: "Proceed along Sansad Marg", turn: "Straight", dist: "1.1 km" },
          { text: "Turn east across Ashoka Road", turn: "Left", dist: "1.2 km" },
          { text: "Cross Windsor Place circular intersection", turn: "Right", dist: "0.9 km" },
          { text: "Approach C-Hexagon western perimeter", turn: "Straight", dist: "0.7 km" }
        ],
        path: [
          [28.6315, 77.2167],
          [28.6265, 77.2120],
          [28.6225, 77.2150],
          [28.6190, 77.2200],
          [28.6150, 77.2260],
          [28.6129, 77.2295]
        ]
      },
      {
        id: "multimodal",
        label: "Multimodal (Walk + Metro)",
        badge: "Yellow + Violet Line",
        color: "#4f46e5", // Indigo
        distanceKm: 4.2,
        durationMin: 14.0,
        turnsCount: 2,
        tolls: "Metro Fare Included",
        roadBreakdown: {
          railway: 3.1,
          pedestrian: 0.9,
          footway: 0.2
        },
        majorStreets: [
          { text: "Walk 250m to Rajiv Chowk Metro Gate 3", turn: "Walk", dist: "0.3 km" },
          { text: "Board Yellow Line towards Central Secretariat", turn: "Metro", dist: "2.4 km" },
          { text: "Interchange to Violet Line to Khan Market / Mandi House", turn: "Metro", dist: "0.7 km" },
          { text: "Walk east along Rajpath lawns to India Gate", turn: "Walk", dist: "0.8 km" }
        ],
        path: [
          [28.6315, 77.2167],
          [28.6305, 77.2185],
          [28.6240, 77.2170],
          [28.6180, 77.2160],
          [28.6150, 77.2230],
          [28.6129, 77.2295]
        ]
      }
    ];

    // Current State Management
    let currentMode = "car";
    let activeRouteIndex = 0;
    let map = null;
    let routePolylines = [];
    let waypointMarkers = [];
    let currentRoutes = [...ROUTE_DATASETS];

    // -------------------------------------------------------------------------
    // 2. LEAFLET MAP INITIALIZATION & TILE LAYERS
    // -------------------------------------------------------------------------
    
    const tileLayers = {
      osm: L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
      }),
      dark: L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
        attribution: '&copy; OpenStreetMap & CARTO'
      }),
      topo: L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenTopoMap contributors'
      })
    };

    function initMap() {
      map = L.map('map', {
        center: [28.6222, 77.2231],
        zoom: 14,
        zoomControl: false,
        layers: [tileLayers.osm]
      });

      L.control.zoom({ position: 'bottomright' }).add_to(map);

      // Render Initial Waypoint Markers
      renderMarkers();
      
      // Render Initial Route Polylines
      renderRoutesOnMap();

      // Setup map style toggles
      document.getElementById('mapStyleOsm').addEventListener('click', () => switchMapStyle('osm'));
      document.getElementById('mapStyleDark').addEventListener('click', () => switchMapStyle('dark'));
      document.getElementById('mapStyleTopo').addEventListener('click', () => switchMapStyle('topo'));
    }

    function switchMapStyle(styleKey) {
      Object.values(tileLayers).forEach(layer => map.removeLayer(layer));
      tileLayers[styleKey].addTo(map);

      const buttons = {
        osm: document.getElementById('mapStyleOsm'),
        dark: document.getElementById('mapStyleDark'),
        topo: document.getElementById('mapStyleTopo')
      };
      
      Object.keys(buttons).forEach(k => {
        if (k === styleKey) {
          buttons[k].className = "px-2.5 py-1 rounded-lg font-medium bg-blue-600 text-white transition";
        } else {
          buttons[k].className = "px-2.5 py-1 rounded-lg font-medium text-slate-600 hover:bg-slate-100 transition";
        }
      });
    }

    function renderMarkers() {
      waypointMarkers.forEach(m => map.removeLayer(m));
      waypointMarkers = [];

      // Custom HTML Pin Generator
      const createPin = (color, title, iconName) => {
        return L.divIcon({
          className: 'custom-pin',
          html: `
            <div class="relative flex items-center justify-center">
              <div class="w-8 h-8 rounded-full ${color} text-white flex items-center justify-center shadow-lg border-2 border-white ring-2 ring-black/10">
                <i data-lucide="${iconName}" style="width:16px;height:16px;"></i>
              </div>
            </div>
          `,
          iconSize: [32, 32],
          iconAnchor: [16, 16]
        });
      };

      // Start Marker
      const startMarker = L.marker(WAYPOINTS.origin, {
        icon: createPin('bg-emerald-600', 'Origin', 'map-pin')
      }).bindPopup("<b>Start:</b> Connaught Place Central Circle").addTo(map);

      // Stop Marker
      const stopMarker = L.marker(WAYPOINTS.stop, {
        icon: createPin('bg-amber-500', 'Via Stop', 'flag')
      }).bindPopup("<b>Via Stop:</b> Janpath Corridor").addTo(map);

      // Dest Marker
      const destMarker = L.marker(WAYPOINTS.dest, {
        icon: createPin('bg-rose-600', 'Destination', 'compass')
      }).bindPopup("<b>Destination:</b> India Gate").addTo(map);

      waypointMarkers.push(startMarker, stopMarker, destMarker);
      setTimeout(() => lucide.createIcons(), 50);
    }

    function renderRoutesOnMap() {
      // Clear existing polylines
      routePolylines.forEach(p => map.removeLayer(p));
      routePolylines = [];

      currentRoutes.forEach((route, idx) => {
        const isSelected = (idx === activeRouteIndex);
        
        const polyline = L.polyline(route.path, {
          color: route.color,
          weight: isSelected ? 7 : 3.5,
          opacity: isSelected ? 0.95 : 0.45,
          lineJoin: 'round',
          lineCap: 'round',
          dashArray: (route.id === 'multimodal') ? '8, 8' : null
        });

        polyline.on('click', () => {
          setActiveRoute(idx);
        });

        // Hover Tooltip
        polyline.bindTooltip(`<b>${route.label}</b><br>${route.distanceKm} km · ${route.durationMin} min`, {
          sticky: true,
          className: 'text-xs'
        });

        polyline.addTo(map);
        routePolylines.push(polyline);
      });

      // Fit map bounds to active polyline
      if (routePolylines[activeRouteIndex]) {
        map.fitBounds(routePolylines[activeRouteIndex].getBounds(), { padding: [50, 50] });
      }
    }

    // -------------------------------------------------------------------------
    // 3. COMPARATIVE DASHBOARD & METRICS CALCULATOR
    // -------------------------------------------------------------------------

    function recalculateMetrics() {
      const fuelPrice = parseFloat(document.getElementById('fuelPriceInput').value) || 1.25;
      const mileage = parseFloat(document.getElementById('mileageInput').value) || 14.0;
      const maxAlts = parseInt(document.getElementById('maxAltsSlider').value) || 5;

      // Filter dataset slice by slider
      currentRoutes = ROUTE_DATASETS.slice(0, maxAlts);

      // Mode-specific adjustment factors
      currentRoutes.forEach(r => {
        let speedFactor = 1.0;
        let costMode = 0.0;

        switch (currentMode) {
          case 'walking':
            speedFactor = 0.20; // 5 km/h
            costMode = 0.0;
            break;
          case 'bicycle':
            speedFactor = 0.45; // 15 km/h
            costMode = 0.0;
            break;
          case 'motorcycle':
            speedFactor = 1.15;
            costMode = (r.distanceKm / 40.0) * fuelPrice;
            break;
          case 'taxi':
            speedFactor = 0.95;
            costMode = 3.0 + (1.20 * r.distanceKm);
            break;
          case 'auto_rickshaw':
            speedFactor = 0.75;
            costMode = 1.5 + (0.60 * r.distanceKm);
            break;
          case 'bus':
            speedFactor = 0.60;
            costMode = 0.5 + (0.08 * r.distanceKm);
            break;
          case 'multimodal':
            speedFactor = 0.70;
            costMode = 0.8 + (0.10 * r.distanceKm);
            break;
          default: // car
            speedFactor = 1.0;
            costMode = (r.distanceKm / mileage) * fuelPrice;
        }

        r.calculatedTime = (r.durationMin / speedFactor).toFixed(1);
        r.calculatedCost = costMode.toFixed(2);
        r.fuelLiters = (currentMode === 'car' || currentMode === 'taxi') 
          ? (r.distanceKm / mileage).toFixed(2) 
          : (currentMode === 'motorcycle' ? (r.distanceKm / 40.0).toFixed(2) : "0.00");
        r.co2Kg = (parseFloat(r.fuelLiters) * 2.31).toFixed(2);
      });

      // Update KPI Cards
      const times = currentRoutes.map(r => parseFloat(r.calculatedTime));
      const dists = currentRoutes.map(r => r.distanceKm);
      const costs = currentRoutes.map(r => parseFloat(r.calculatedCost));

      document.getElementById('statAlternatives').innerText = `${currentRoutes.length} Options`;
      document.getElementById('statFastestTime').innerText = `${Math.min(...times)} min`;
      document.getElementById('statShortestDist').innerText = `${Math.min(...dists)} km`;
      document.getElementById('statLowestCost').innerText = `$${Math.min(...costs).toFixed(2)}`;

      renderComparisonTable();
      renderSelectedRouteDetails();
      renderLegend();
      renderRoutesOnMap();
    }

    function renderComparisonTable() {
      const tbody = document.getElementById('routesTableBody');
      tbody.innerHTML = '';

      currentRoutes.forEach((r, idx) => {
        const isSelected = (idx === activeRouteIndex);
        const tr = document.createElement('tr');
        tr.className = `cursor-pointer transition hover:bg-slate-50 ${isSelected ? 'bg-blue-50/70 font-semibold' : ''}`;
        
        tr.innerHTML = `
          <td class="py-2.5 px-3 flex items-center gap-2">
            <span class="w-2.5 h-2.5 rounded-full flex-shrink-0" style="background-color: ${r.color}"></span>
            <span class="truncate max-w-[120px] sm:max-w-none text-slate-900">${r.label}</span>
          </td>
          <td class="py-2.5 px-2 text-slate-600">${r.distanceKm} km</td>
          <td class="py-2.5 px-2 text-slate-600">${r.calculatedTime} m</td>
          <td class="py-2.5 px-2 text-slate-500">${r.turnsCount}</td>
          <td class="py-2.5 px-3 text-slate-900 font-bold">$${r.calculatedCost}</td>
        `;

        tr.addEventListener('click', () => setActiveRoute(idx));
        tbody.appendChild(tr);
      });
    }

    function renderSelectedRouteDetails() {
      const route = currentRoutes[activeRouteIndex] || currentRoutes[0];
      if (!route) return;

      document.getElementById('selectedRouteTitle').innerHTML = `
        <span class="w-3 h-3 rounded-full inline-block" style="background-color: ${route.color}"></span>
        ${route.label}
      `;
      document.getElementById('selectedRouteBadge').innerText = route.badge;
      document.getElementById('routeFuelLitres').innerText = `${route.fuelLiters} L`;
      document.getElementById('routeCO2').innerText = `${route.co2Kg} kg`;
      document.getElementById('routeTollInfo').innerText = route.tolls;

      // Render Stacked Road Hierarchy Bar
      const hierarchyBar = document.getElementById('roadHierarchyBar');
      const hierarchyList = document.getElementById('roadHierarchyList');
      hierarchyBar.innerHTML = '';
      hierarchyList.innerHTML = '';

      const colorMap = {
        trunk: '#7c3aed',
        primary: '#2563eb',
        secondary: '#0891b2',
        tertiary: '#059669',
        residential: '#ea580c',
        service: '#94a3b8',
        living_street: '#f59e0b',
        railway: '#4338ca',
        pedestrian: '#10b981',
        footway: '#34d399'
      };

      const totalDist = Object.values(route.roadBreakdown).reduce((a, b) => a + b, 0);

      Object.entries(route.roadBreakdown).forEach(([cat, km]) => {
        const pct = Math.round((km / totalDist) * 100);
        const col = colorMap[cat] || '#64748b';

        // Bar element
        const slice = document.createElement('div');
        slice.style.width = `${pct}%`;
        slice.style.backgroundColor = col;
        slice.title = `${cat}: ${km} km (${pct}%)`;
        hierarchyBar.appendChild(slice);

        // List item
        const item = document.createElement('div');
        item.className = "flex items-center justify-between text-slate-600";
        item.innerHTML = `
          <span class="flex items-center gap-1.5 truncate">
            <span class="w-2 h-2 rounded-full" style="background-color: ${col}"></span>
            <span class="capitalize">${cat.replace('_', ' ')}</span>
          </span>
          <span class="font-semibold text-slate-700">${km} km <span class="text-slate-400 font-normal">(${pct}%)</span></span>
        `;
        hierarchyList.appendChild(item);
      });

      // Render Step-by-Step Directions
      const turnsList = document.getElementById('turnsList');
      turnsList.innerHTML = '';
      route.majorStreets.forEach((step, i) => {
        const row = document.createElement('div');
        row.className = "py-1.5 flex items-start gap-2 text-[11px]";
        row.innerHTML = `
          <span class="px-1.5 py-0.5 rounded bg-slate-100 text-slate-700 font-mono text-[10px] font-bold mt-0.5">${i+1}</span>
          <div class="flex-1">
            <div class="font-medium text-slate-800">${step.text}</div>
            <div class="text-[10px] text-slate-400 flex items-center gap-1 mt-0.5">
              <span>${step.turn}</span> · <span>${step.dist}</span>
            </div>
          </div>
        `;
        turnsList.appendChild(row);
      });
    }

    function renderLegend() {
      const container = document.getElementById('mapLegendItems');
      container.innerHTML = '';
      currentRoutes.forEach((r, idx) => {
        const isSelected = (idx === activeRouteIndex);
        const item = document.createElement('div');
        item.className = `flex items-center justify-between gap-3 cursor-pointer py-0.5 ${isSelected ? 'font-bold' : 'text-slate-600'}`;
        item.innerHTML = `
          <span class="flex items-center gap-1.5">
            <span class="w-2.5 h-2.5 rounded-full" style="background-color: ${r.color}"></span>
            <span>${r.label}</span>
          </span>
          <span class="font-mono text-slate-500">${r.calculatedTime}m</span>
        `;
        item.addEventListener('click', () => setActiveRoute(idx));
        container.appendChild(item);
      });
    }

    function setActiveRoute(index) {
      if (index >= currentRoutes.length) return;
      activeRouteIndex = index;

      // Update polylines styling
      routePolylines.forEach((poly, idx) => {
        const isSelected = (idx === activeRouteIndex);
        poly.setStyle({
          weight: isSelected ? 7 : 3.5,
          opacity: isSelected ? 0.95 : 0.45
        });
        if (isSelected) poly.bringToFront();
      });

      renderComparisonTable();
      renderSelectedRouteDetails();
      renderLegend();
    }

    // -------------------------------------------------------------------------
    // 4. EVENT LISTENERS & USER CONTROLS
    // -------------------------------------------------------------------------

    // Mode Selector
    document.querySelectorAll('.mode-btn').forEach(btn => {
      btn.addEventListener('click', () => {
        document.querySelectorAll('.mode-btn').forEach(b => {
          b.className = "mode-btn flex flex-col items-center justify-center p-2 rounded-xl border border-slate-200 bg-white text-slate-600 hover:border-slate-300 transition";
        });
        btn.className = "mode-btn active flex flex-col items-center justify-center p-2 rounded-xl border border-blue-600 bg-blue-50 text-blue-700 transition";
        currentMode = btn.getAttribute('data-mode');
        recalculateMetrics();
      });
    });

    // Sliders
    const maxAltsSlider = document.getElementById('maxAltsSlider');
    maxAltsSlider.addEventListener('input', (e) => {
      document.getElementById('maxAltsVal').innerText = e.target.value;
      document.getElementById('altCountBadge').innerText = `${e.target.value} Paths`;
      if (activeRouteIndex >= parseInt(e.target.value)) {
        activeRouteIndex = 0;
      }
      recalculateMetrics();
    });

    const corridorSlider = document.getElementById('corridorSlider');
    corridorSlider.addEventListener('input', (e) => {
      document.getElementById('corridorVal').innerText = `${parseFloat(e.target.value).toFixed(1)} km`;
    });

    // Inputs
    document.getElementById('fuelPriceInput').addEventListener('input', recalculateMetrics);
    document.getElementById('mileageInput').addEventListener('input', recalculateMetrics);

    // Swap Waypoints
    document.getElementById('swapBtn').addEventListener('click', () => {
      const orig = document.getElementById('originInput');
      const dest = document.getElementById('destInput');
      const temp = orig.value;
      orig.value = dest.value;
      dest.value = temp;
      recalculateMetrics();
    });

    // Remove Stop Button
    document.getElementById('removeStopBtn').addEventListener('click', () => {
      const container = document.getElementById('stopsContainer');
      container.style.display = container.style.display === 'none' ? 'block' : 'none';
    });

    // Discover Action Button
    document.getElementById('discoverBtn').addEventListener('click', () => {
      const btn = document.getElementById('discoverBtn');
      btn.innerHTML = `<i data-lucide="loader-2" class="w-4 h-4 animate-spin"></i><span>Analyzing Network...</span>`;
      lucide.createIcons();

      setTimeout(() => {
        btn.innerHTML = `<i data-lucide="sparkles" class="w-4 h-4"></i><span>Discover All Routes</span>`;
        lucide.createIcons();
        recalculateMetrics();
      }, 400);
    });

    // Turn Directions Accordion Toggle
    document.getElementById('toggleTurnsBtn').addEventListener('click', () => {
      const list = document.getElementById('turnsList');
      const chevron = document.getElementById('turnsChevron');
      if (list.style.display === 'none') {
        list.style.display = 'block';
        chevron.style.transform = 'rotate(0deg)';
      } else {
        list.style.display = 'none';
        chevron.style.transform = 'rotate(-90deg)';
      }
    });

    // Mobile Sidebar Drawer
    const sidebar = document.getElementById('sidebarPanel');
    document.getElementById('toggleMobileSidebarBtn').addEventListener('click', () => {
      sidebar.classList.toggle('-translate-x-full');
    });
    document.getElementById('closeMobileSidebarBtn').addEventListener('click', () => {
      sidebar.classList.add('-translate-x-full');
    });

    // -------------------------------------------------------------------------
    // 5. APPLICATION LAUNCH
    // -------------------------------------------------------------------------
    window.addEventListener('DOMContentLoaded', () => {
      initMap();
      recalculateMetrics();
    });
  </script>
</body>
</html>
```
