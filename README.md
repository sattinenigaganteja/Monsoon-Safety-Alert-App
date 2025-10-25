<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monsoon Safety Net</title>

    <script src="https://cdn.tailwindcss.com"></script>

    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin="" />

    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
        }
        .weather-card {
            background: linear-gradient(135deg, #0052D4, #4364F7, #6FB1FC);
            color: white;
            transition: transform 0.3s ease-in-out;
        }
        .main-icon {
            font-size: 4.5rem;
        }
        .alert-high {
            background-color: #ef4444; /* red-500 */
            color: white;
        }
        .alert-medium {
            background-color: #f97316; /* orange-500 */
            color: white;
        }
        .alert-low {
            background-color: #eab308; /* yellow-500 */
            color: white;
        }
        .nav-link-active {
            color: #4364F7;
            background-color: #eef2ff;
            border-bottom: 2px solid #4364F7;
        }
        .nav-link {
             border-bottom: 2px solid transparent;
        }
        .btn-primary {
            background-color: #4364F7;
            transition: background-color 0.3s;
        }
        .btn-primary:hover:not(:disabled) {
            background-color: #0052D4;
        }
        .btn-primary:disabled {
            background-color: #a5b4fc;
            cursor: not-allowed;
        }
        .btn-secondary {
            background-color: #6772e5;
            transition: background-color 0.3s;
        }
        .btn-secondary:hover {
            background-color: #5469d4;
        }
        /* Custom scrollbar for a better look */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 3px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
        .modal-backdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
            z-index: 50;
            display: flex; /* Changed */
        }
        /* Form transition */
        .form-container {
            transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
        }
        .form-hidden {
            opacity: 0;
            transform: scale(0.95);
            position: absolute;
            pointer-events: none;
        }
        #weather-map {
            height: 500px;
            border-radius: 1rem;
            z-index: 1;
        }
        .leaflet-control-layers-toggle {
            background-image: url(https://unpkg.com/leaflet@1.9.4/dist/images/layers.png);
        }
        .legend {
            background-color: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 5px;
            box-shadow: 0 0 15px rgba(0,0,0,0.2);
            line-height: 1.5;
        }
        .legend h4 {
            margin-top: 0;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .legend i {
            width: 18px;
            height: 18px;
            float: left;
            margin-right: 8px;
            opacity: 0.7;
        }
        /* Scrolling Ticker Animation */
        #alert-ticker-content {
            animation: scroll-left 40s linear infinite;
        }

        @keyframes scroll-left {
            0% {
                transform: translateX(100%);
            }
            100% {
                transform: translateX(-100%);
            }
        }
    </style>
</head>
<body class="antialiased">

    <div id="app">
        
        <div id="auth-screen" class="min-h-screen flex items-center justify-center p-4">
            <div class="w-full max-w-md bg-white p-8 rounded-2xl shadow-lg relative overflow-hidden">
                <div class="text-center mb-8">
                    <i class="fas fa-cloud-showers-heavy text-5xl text-blue-600"></i>
                    <h1 class="text-3xl font-bold text-gray-800 mt-2">Monsoon Safety Net</h1>
                    <p class="text-gray-500">Your real-time rain alert and safety companion.</p>
                </div>

                <div id="auth-container" class="relative">
                    <form id="login-form" class="space-y-4 form-container">
                        <h2 class="text-xl font-semibold text-center text-gray-700">Login</h2>
                        <div>
                            <label for="login-email" class="block text-sm font-medium text-gray-600">Email</label>
                            <input type="email" id="login-email" required class="w-full mt-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        </div>
                        <div>
                            <label for="login-password" class="block text-sm font-medium text-gray-600">Password</label>
                            <input type="password" id="login-password" required class="w-full mt-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        </div>
                        <p id="login-error" class="text-red-500 text-sm hidden min-h-[20px]"></p>
                        <button type="submit" class="w-full btn-primary text-white font-semibold py-2 rounded-lg flex items-center justify-center">
                            <span class="btn-text">Login</span>
                            <i class="fas fa-spinner fa-spin ml-2 hidden"></i>
                        </button>
                        <p class="text-center text-sm text-gray-600">
                            Don't have an account? <a href="#" id="show-register" class="font-semibold text-blue-600 hover:underline">Register here</a>
                        </p>

                        <div class="relative flex py-2 items-center">
                            <div class="flex-grow border-t border-gray-300"></div>
                            <span class="flex-shrink mx-4 text-gray-400 text-sm">OR</span>
                            <div class="flex-grow border-t border-gray-300"></div>
                        </div>

                        <button type="button" id="google-signin-btn-login" class="w-full bg-white border border-gray-300 text-gray-700 font-semibold py-2 rounded-lg flex items-center justify-center hover:bg-gray-50 transition">
                            <img src="https://upload.wikimedia.org/wikipedia/commons/c/c1/Google_%22G%22_logo.svg" alt="Google logo" class="h-5 mr-3">
                            Sign in with Google
                        </button>
                    </form>

                    <form id="register-form" class="space-y-4 form-container form-hidden">
                         <h2 class="text-xl font-semibold text-center text-gray-700">Create Account</h2>
                        <div>
                            <label for="register-email" class="block text-sm font-medium text-gray-600">Email</label>
                            <input type="email" id="register-email" required class="w-full mt-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        </div>
                        <div>
                            <label for="register-password" class="block text-sm font-medium text-gray-600">Password (min. 6 characters)</label>
                            <input type="password" id="register-password" required class="w-full mt-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                        </div>
                        <p id="register-error" class="text-red-500 text-sm hidden min-h-[20px]"></p>
                        <button type="submit" class="w-full btn-primary text-white font-semibold py-2 rounded-lg flex items-center justify-center">
                            <span class="btn-text">Register</span>
                            <i class="fas fa-spinner fa-spin ml-2 hidden"></i>
                        </button>
                        <p class="text-center text-sm text-gray-600">
                            Already have an account? <a href="#" id="show-login" class="font-semibold text-blue-600 hover:underline">Login here</a>
                        </p>

                         <div class="relative flex py-2 items-center">
                            <div class="flex-grow border-t border-gray-300"></div>
                            <span class="flex-shrink mx-4 text-gray-400 text-sm">OR</span>
                            <div class="flex-grow border-t border-gray-300"></div>
                        </div>

                        <button type="button" id="google-signin-btn-register" class="w-full bg-white border border-gray-300 text-gray-700 font-semibold py-2 rounded-lg flex items-center justify-center hover:bg-gray-50 transition">
                            <img src="https://upload.wikimedia.org/wikipedia/commons/c/c1/Google_%22G%22_logo.svg" alt="Google logo" class="h-5 mr-3">
                            Sign up with Google
                        </button>
                    </form>
                </div>
            </div>
        </div>

        <main id="main-app" class="hidden bg-white shadow-lg">
            <header class="bg-gray-800 text-white p-4 flex justify-between items-center sticky top-0 z-30">
                <h1 class="text-xl font-bold"><i class="fas fa-cloud-showers-heavy"></i> Monsoon Safety Net</h1>
                <div class="flex items-center gap-4">
                    <button id="sos-btn" class="bg-red-600 hover:bg-red-700 text-white font-bold py-2 px-4 rounded-lg text-sm transition">
                        <i class="fas fa-exclamation-triangle"></i> SOS
                    </button>
                    <button id="logout-btn" class="text-sm text-gray-300 hover:text-white"><i class="fas fa-sign-out-alt"></i> Logout</button>
                </div>
            </header>

            <div id="weather-ticker" class="bg-blue-600 text-white p-2 flex justify-center items-center text-sm sticky top-[64px] z-20 shadow-md">
                <span id="ticker-location" class="font-bold">Hyderabad</span>:
                <span id="ticker-temp" class="ml-2">24°C</span>
                <span id="ticker-desc" class="ml-2 hidden sm:inline">, Light Rain</span>
            </div>
            
            <div id="alert-ticker" class="bg-yellow-400 text-yellow-900 p-2 text-sm shadow-md overflow-hidden whitespace-nowrap sticky top-[104px] z-20">
                <div id="alert-ticker-content" class="inline-block">
                    </div>
            </div>

            <nav class="bg-white shadow-md flex flex-wrap justify-around border-b sticky top-[136px] z-20">
                <button data-page="dashboard" class="nav-link nav-link-active font-semibold py-3 px-2 text-center text-gray-700 hover:bg-gray-100 w-1/2 md:w-1/5 transition-colors duration-200">
                    <i class="fas fa-th-large block mb-1"></i> Dashboard
                </button>
                 <button data-page="map" class="nav-link font-semibold py-3 px-2 text-center text-gray-500 hover:bg-gray-100 w-1/2 md:w-1/5 transition-colors duration-200">
                    <i class="fas fa-map-marked-alt block mb-1"></i> Weather Map
                </button>
                <button data-page="alerts" class="nav-link font-semibold py-3 px-2 text-center text-gray-500 hover:bg-gray-100 w-1/2 md:w-1/5 transition-colors duration-200">
                    <i class="fas fa-shield-alt block mb-1"></i> Alerts & Guides
                </button>
                <button data-page="community" class="nav-link font-semibold py-3 px-2 text-center text-gray-500 hover:bg-gray-100 w-1/2 md:w-1/5 transition-colors duration-200">
                    <i class="fas fa-users block mb-1"></i> Community
                </button>
                <button data-page="emergency" class="nav-link font-semibold py-3 px-2 text-center text-gray-500 hover:bg-gray-100 w-full md:w-1/5 transition-colors duration-200">
                    <i class="fas fa-first-aid block mb-1"></i> Emergency
                </button>
            </nav>
            
            <div id="page-content" class="p-4 md:p-6 lg:p-8">
                <div id="dashboard-page" class="page">
                    <section id="weather-section">
                        <div class="weather-card p-6 rounded-2xl shadow-lg mb-6 max-w-4xl mx-auto">
                            <div class="flex justify-between items-start flex-wrap gap-2">
                                <div>
                                    <h2 id="location" class="text-2xl md:text-3xl font-bold">Hyderabad</h2>
                                    <p id="weather-description" class="text-lg">Light Rain</p>
                                </div>
                                <div class="text-left sm:text-right">
                                    <p id="current-date" class="font-medium"></p>
                                    <p id="current-time" class="font-medium"></p>
                                </div>
                            </div>
                            <div class="flex items-center justify-between mt-4">
                                 <div class="flex items-center">
                                    <i id="weather-icon" class="wi wi-day-rain main-icon"></i>
                                    <p id="temperature" class="text-5xl md:text-6xl font-bold ml-4">24°C</p>
                                </div>
                                <div class="text-right space-y-2">
                                    <p id="feels-like"><i class="fas fa-temperature-half fa-fw mr-1"></i> Feels Like: 25°C</p>
                                    <p id="humidity"><i class="fas fa-tint fa-fw mr-1"></i> Humidity: 94%</p>
                                    <p id="wind-speed"><i class="fas fa-wind fa-fw mr-1"></i> Wind: 10 km/h</p>
                                </div>
                            </div>
                            <form id="location-search-form" class="mt-6 flex gap-2">
                                <input id="location-input" type="text" placeholder="Search for a city or pincode..." class="w-full px-4 py-2 rounded-lg bg-white/20 placeholder-white/70 text-white focus:outline-none focus:ring-2 focus:ring-white">
                                <button type="submit" class="bg-white/30 hover:bg-white/40 text-white font-bold py-2 px-4 rounded-lg">
                                    <i class="fas fa-search"></i>
                                </button>
                            </form>
                        </div>

                         <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4 text-center max-w-4xl mx-auto mb-6">
                            <div class="bg-white p-4 rounded-lg shadow">
                                <i class="fas fa-sun text-yellow-500 text-2xl mb-2"></i>
                                <h3 class="font-bold text-gray-700">Sunrise</h3>
                                <p id="sunrise-time" class="text-gray-600">6:05 AM</p>
                            </div>
                            <div class="bg-white p-4 rounded-lg shadow">
                                <i class="fas fa-moon text-blue-800 text-2xl mb-2"></i>
                                <h3 class="font-bold text-gray-700">Sunset</h3>
                                <p id="sunset-time" class="text-gray-600">6:45 PM</p>
                            </div>
                             <div class="bg-white p-4 rounded-lg shadow">
                                <i class="fas fa-eye text-gray-500 text-2xl mb-2"></i>
                                <h3 class="font-bold text-gray-700">Visibility</h3>
                                <p id="visibility" class="text-gray-600">5 km</p>
                            </div>
                             <div class="bg-white p-4 rounded-lg shadow">
                                <i class="fas fa-tachometer-alt text-gray-500 text-2xl mb-2"></i>
                                <h3 class="font-bold text-gray-700">Pressure</h3>
                                <p id="pressure" class="text-gray-600">1002 hPa</p>
                            </div>
                             <div class="bg-white p-4 rounded-lg shadow flex flex-col items-center justify-center">
                                <h3 class="font-bold text-gray-700 mb-2">Wind Direction</h3>
                                <div class="relative w-16 h-16">
                                    <i id="wind-direction-arrow" class="fas fa-arrow-up text-3xl text-gray-600 transition-transform duration-500"></i>
                                </div>
                                <p id="wind-direction" class="text-gray-600 font-semibold mt-1">NW</p>
                            </div>
                        </div>

                        <div class="max-w-4xl mx-auto">
                            <button id="hourly-outlook-btn" class="w-full btn-secondary text-white font-semibold py-3 rounded-lg shadow-md">
                                <i class="fas fa-clock"></i> Get AI-Powered Hourly Outlook ✨
                            </button>
                        </div>
                    </section>
                </div>

                {/* */}
                <div id="map-page" class="page hidden max-w-6xl mx-auto">
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">Live Weather Map</h2>
                    <p class="text-gray-600 mb-4">Use the controls to toggle live precipitation and wind speed overlays. The map will automatically center on your searched location.</p>
                    <button id="find-location-btn" class="mb-4 btn-primary text-white font-semibold py-2 px-4 rounded-lg">
                        <i class="fas fa-location-arrow"></i> Find My Location
                    </button>
                    <div id="weather-map" class="shadow-lg"></div>
                </div>

                {/* */}
                <div id="alerts-page" class="page hidden max-w-4xl mx-auto">
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">Alerts & Safety Guides</h2>
                    <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                        <div>
                            <h3 class="text-xl font-semibold text-gray-800 mb-3">Local Safety Alerts for Hyderabad</h3>
                            <div id="geo-alerts-container" class="space-y-3 mb-8">
                               {/* */}
                            </div>
                            
                            <div class="bg-gray-100 p-4 rounded-lg">
                                <h4 class="font-bold text-lg text-gray-800 mb-2"><i class="fas fa-info-circle mr-2"></i>Understanding Alerts</h4>
                                <p class="text-sm text-gray-700">Alert levels indicate potential severity. <span class="font-bold text-yellow-600">Low</span> suggests caution, <span class="font-bold text-orange-600">Medium</span> indicates potential for disruption, and <span class="font-bold text-red-600">High</span> warns of significant danger.</p>
                            </div>
                        </div>
                        
                        <div>
                            <h3 class="text-xl font-semibold text-gray-800 mb-3">Essential Safety Guides</h3>
                            <div class="space-y-4">
                                <div class="bg-gray-100 p-4 rounded-lg">
                                    <h4 class="font-bold text-lg text-gray-800 mb-2"><i class="fas fa-house-flood-water mr-2"></i>Flood Safety Guide</h4>
                                    <p class="text-sm text-gray-600 mb-2">Flooding is a primary monsoon hazard. Know how to react to stay safe.</p>
                                    <ul class="list-disc list-inside text-gray-700 text-sm space-y-1">
                                        <li>Do not walk or drive through flowing water. Six inches of moving water can knock you down.</li>
                                        <li>Move to higher ground immediately if a flood warning is issued.</li>
                                        <li>Avoid contact with floodwater; it may be contaminated or contain dangerous debris.</li>
                                    </ul>
                                </div>
                                <div class="bg-gray-100 p-4 rounded-lg">
                                    <h4 class="font-bold text-lg text-gray-800 mb-2"><i class="fas fa-car-battery mr-2"></i>Power Outage Guide</h4>
                                    <p class="text-sm text-gray-600 mb-2">Power outages are common. Staying prepared can ensure your safety and comfort.</p>
                                    <ul class="list-disc list-inside text-gray-700 text-sm space-y-1">
                                        <li>Have flashlights with extra batteries. Avoid candles to prevent fire hazards.</li>
                                        <li>Keep a power bank fully charged for your mobile devices.</li>
                                        <li>Unplug sensitive electronics to protect from power surges.</li>
                                    </ul>
                                </div>
                            </div>
                        </div>
                    </div>
                     <div class="mt-8">
                        <h3 class="text-xl font-semibold text-gray-800 mb-3">AI Health & Weather Tools</h3>
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                            <button id="safety-tips-btn" class="w-full btn-secondary text-white font-semibold py-3 rounded-lg shadow-md">
                                <i class="fas fa-brain"></i> AI Safety Tips ✨
                            </button>
                            <button id="forecast-btn" class="w-full btn-secondary text-white font-semibold py-3 rounded-lg shadow-md">
                                <i class="fas fa-calendar-alt"></i> 3-Day Forecast ✨
                            </button>
                            <button id="allergy-outlook-btn" class="w-full btn-secondary text-white font-semibold py-3 rounded-lg shadow-md">
                                <i class="fas fa-seedling"></i> Allergy Outlook ✨
                            </button>
                        </div>
                    </div>
                </div>
                
                {/* */}
                <div id="emergency-page" class="page hidden max-w-4xl mx-auto">
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">Emergency Contacts</h2>
                    <p class="text-gray-600 mb-6">In an emergency, quick access to the right numbers is critical. This list contains national and local helplines. Click any contact to call them directly if you are on a mobile device.</p>
                    <div id="emergency-contacts-container" class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        {/* */}
                    </div>
                </div>
                {/* */}
                <div id="community-page" class="page hidden max-w-4xl mx-auto">
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">Community Feed</h2>
                    <p class="text-gray-600 mb-6">Share and view real-time, on-the-ground updates from fellow citizens. Report waterlogging, traffic jams, or other hazards to help keep your community informed and safe.</p>
                    <div class="grid grid-cols-1 md:grid-cols-4 gap-6">
                        <div class="md:col-span-3">
                            <div class="bg-gray-50 p-4 rounded-lg">
                                 <form id="post-form" class="flex items-start space-x-3">
                                     <textarea id="post-content" required class="w-full p-2 border rounded-lg focus:ring-blue-500 focus:border-blue-500" rows="2" placeholder="Share an update (e.g., waterlogging at Ameerpet)"></textarea>
                                     <button type="submit" class="btn-primary text-white font-semibold px-4 py-2 rounded-lg h-full"><i class="fas fa-paper-plane"></i></button>
                                 </form>
                            </div>
                            <div class="my-4">
                                <button id="summarize-feed-btn" class="w-full btn-secondary text-white font-semibold py-2 rounded-lg text-sm">
                                    <i class="fas fa-wand-magic-sparkles"></i> Summarize Feed with AI ✨
                                </button>
                            </div>
                            <div id="feed-container" class="mt-4 space-y-4 custom-scrollbar h-96 overflow-y-auto p-1">
                                </div>
                        </div>
                        <div class="md:col-span-1">
                             <h3 class="text-xl font-semibold text-gray-800 mb-3">Trending Topics</h3>
                             <div class="bg-gray-100 p-4 rounded-lg space-y-2">
                                <a href="#" target="_blank" rel="noopener noreferrer" class="block text-blue-600 hover:underline">#HyderabadRains</a>
                                <a href="#" target="_blank" rel="noopener noreferrer" class="block text-blue-600 hover:underline">#TrafficAlert</a>
                                <a href="#" target="_blank" rel="noopener noreferrer" class="block text-blue-600 hover:underline">#Waterlogging</a>
                                <a href="#" target="_blank" rel="noopener noreferrer" class="block text-blue-600 hover:underline">#KPHB</a>
                             </div>
                        </div>
                    </div>
                </div>
            </div>

             <footer class="bg-gray-800 text-white pt-10 pb-6 mt-8">
                <div class="max-w-6xl mx-auto grid grid-cols-2 md:grid-cols-4 gap-8 px-4">
                    <div>
                        <h4 class="font-bold mb-3">Company</h4>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">About Us</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Contact</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Careers</a>
                    </div>
                     <div>
                        <h4 class="font-bold mb-3">Services</h4>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Weather API</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">AI Tools</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Partnerships</a>
                    </div>
                     <div>
                        <h4 class="font-bold mb-3">Follow Us</h4>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2"><i class="fab fa-twitter mr-2"></i>Twitter</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2"><i class="fab fa-facebook mr-2"></i>Facebook</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2"><i class="fab fa-instagram mr-2"></i>Instagram</a>
                    </div>
                     <div>
                        <h4 class="font-bold mb-3">Legal</h4>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Privacy Policy</a>
                        <a href="#" target="_blank" rel="noopener noreferrer" class="block text-gray-400 hover:text-white text-sm mb-2">Terms of Use</a>
                    </div>
                </div>
                 <div class="text-center text-gray-500 text-sm mt-8 border-t border-gray-700 pt-4">
                    &copy; 2025 Monsoon Safety Net. All rights reserved.
                </div>
            </footer>
        </main>
        
        <div id="gemini-modal" class="modal-backdrop hidden items-center justify-center p-4">
            <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-lg relative">
                <button id="close-gemini-modal" class="absolute top-2 right-3 text-gray-500 hover:text-gray-800 text-2xl">&times;</button>
                <h2 id="gemini-modal-title" class="text-xl font-bold text-gray-800 mb-4">AI Generated Response</h2>
                <div id="gemini-modal-content" class="text-gray-600 space-y-2">
                    <div id="gemini-spinner" class="text-center py-8">
                        <i class="fas fa-spinner fa-spin text-4xl text-blue-600"></i>
                        <p class="mt-2 text-lg">Generating response...</p>
                    </div>
                    <div id="gemini-response-text"></div>
                </div>
            </div>
        </div>

    </div>

    {/* */}
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>

    {/* */}
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
        import { getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword, onAuthStateChanged, signOut, GoogleAuthProvider, signInWithPopup } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, orderBy, onSnapshot, serverTimestamp, getDocs, limit } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore.js";

        // For Firebase JS SDK v7.20.0 and later, measurementId is optional
        const firebaseConfig = {
            apiKey: "AIzaSyDIwTDUeRWxid3f7tKLNX4m5v-PnTClMes",
            authDomain: "monsoon-safety-app.firebaseapp.com",
            projectId: "monsoon-safety-app",
            storageBucket: "monsoon-safety-app.firebasestorage.app",
            messagingSenderId: "418895526259",
            appId: "1:418895526259:web:724dcd5e6cf3645f682d06",
            measurementId: "G-K44QB5XGCG"
        };
        
        // --- App Initialization ---
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // --- App State & Constants ---
        const OWM_API_KEY = "19f2c9f658d3b41ad4643857eb350e34";
        const GEMINI_API_KEY = "AIzaSyBuVKaPh6AalgvfmuwIzyNYTED0kr1ARyw";
        let map = null; // To hold the map instance
        let userMarker = null; // To hold the user's location marker
        
        const floodProneAreas = {
            'Begumpet': { level: 'high', message: 'High flood risk near the airport. Avoid Begumpet flyover.' },
            'Kukatpally': { level: 'medium', message: 'Waterlogging reported near JNTU junction. Proceed with caution.'},
            'Secunderabad': { level: 'low', message: 'Minor waterlogging near the railway station. Traffic is slow.'},
            'Madhapur': { level: 'medium', message: 'Reports of waterlogging in low-lying areas. Be careful.'}
        };

        const emergencyContacts = [
            { name: 'National Emergency', number: '112', icon: 'fa-bullhorn' },
            { name: 'Disaster Management', number: '1077', icon: 'fa-house-tsunami' },
            { name: 'Traffic Police Helpline', number: '103', icon: 'fa-traffic-light' },
            { name: 'Electricity Dept.', number: '1912', icon: 'fa-bolt' },
            { name: 'GHMC Control Room', number: '040-2322-5397', icon: 'fa-city' },
            { name: 'Gas Leak Helpline', number: '1906', icon: 'fa-fire-extinguisher' },
            { name: 'Apollo Hospital', number: '1860-500-1066', icon: 'fa-hospital' },
            { name: 'Animal Rescue', number: '040-2713-3540', icon: 'fa-paw' },
        ];

        // --- UI Element References ---
        const authScreen = document.getElementById('auth-screen');
        const mainApp = document.getElementById('main-app');
        const geminiModal = document.getElementById('gemini-modal');
        const loginForm = document.getElementById('login-form');
        const registerForm = document.getElementById('register-form');

        // --- Helper Functions ---
        const toggleButtonLoading = (button, isLoading) => {
            const text = button.querySelector('.btn-text');
            const spinner = button.querySelector('i');
            if (isLoading) {
                button.disabled = true;
                text.classList.add('hidden');
                spinner.classList.remove('hidden');
            } else {
                button.disabled = false;
                text.classList.remove('hidden');
                spinner.classList.add('hidden');
            }
        };

        const degreesToCardinal = (deg) => {
            const directions = ['N', 'NNE', 'NE', 'ENE', 'E', 'ESE', 'SE', 'SSE', 'S', 'SSW', 'SW', 'WSW', 'W', 'WNW', 'NW', 'NNW'];
            const index = Math.round((deg % 360) / 22.5) % 16;
            return directions[index];
        }

        // --- Authentication Logic ---
        onAuthStateChanged(auth, user => {
            if (user) {
                authScreen.classList.add('hidden');
                mainApp.classList.remove('hidden');
                initializeAppState();
            } else {
                mainApp.classList.add('hidden');
                authScreen.classList.remove('hidden');
            }
        });

        const handleGoogleSignIn = async () => {
            const provider = new GoogleAuthProvider();
            try {
                await signInWithPopup(auth, provider);
            } catch (error) {
                console.error("Google Sign-In Error:", error);
                const errorEl = document.getElementById('login-error');
                errorEl.textContent = 'Google sign-in failed. Please try again.';
                errorEl.classList.remove('hidden');
            }
        };

        document.getElementById('google-signin-btn-login').addEventListener('click', handleGoogleSignIn);
        document.getElementById('google-signin-btn-register').addEventListener('click', handleGoogleSignIn);


        loginForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const button = loginForm.querySelector('button[type="submit"]');
            toggleButtonLoading(button, true);
            const email = document.getElementById('login-email').value;
            const password = document.getElementById('login-password').value;
            const errorEl = document.getElementById('login-error');
            errorEl.classList.add('hidden');

            try {
                await signInWithEmailAndPassword(auth, email, password);
            } catch (error) {
                errorEl.classList.remove('hidden');
                if (error.code === 'auth/user-not-found' || error.code === 'auth/invalid-credential') {
                     errorEl.innerHTML = 'User not found. <a href="#" id="error-show-register" class="font-semibold text-blue-600 hover:underline">Create an account?</a>';
                     document.getElementById('error-show-register').addEventListener('click', (e) => {
                         e.preventDefault();
                         switchAuthForms();
                     });
                } else {
                    errorEl.textContent = error.message;
                }
            } finally {
                toggleButtonLoading(button, false);
            }
        });

        registerForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const button = registerForm.querySelector('button[type="submit"]');
            toggleButtonLoading(button, true);
            const email = document.getElementById('register-email').value;
            const password = document.getElementById('register-password').value;
            const errorEl = document.getElementById('register-error');
            errorEl.classList.add('hidden');
            try {
                await createUserWithEmailAndPassword(auth, email, password);
            // --- CORRECTION 1: Fixed stray brace syntax error ---
            } catch (error) {
                errorEl.textContent = error.message;
                errorEl.classList.remove('hidden');
            } finally {
                toggleButtonLoading(button, false);
            }
        });

        const switchAuthForms = () => {
             document.getElementById('login-error').classList.add('hidden');
             document.getElementById('register-error').classList.add('hidden');
             loginForm.classList.toggle('form-hidden');
             registerForm.classList.toggle('form-hidden');
        }

        document.getElementById('show-register').addEventListener('click', (e) => { e.preventDefault(); switchAuthForms(); });
        document.getElementById('show-login').addEventListener('click', (e) => { e.preventDefault(); switchAuthForms(); });
        document.getElementById('logout-btn').addEventListener('click', () => signOut(auth));
        
        // --- Core Application Logic ---
        function initializeAppState() {
            getCurrentLocation(); // Try to get user location on start
            displayGeoAlerts();
            displayEmergencyContacts();
            setupCommunityFeed();
            initializeMap();
            displayScrollingAlerts();
            updateTime();
            setInterval(updateTime, 1000 * 30);
        }

        document.getElementById('location-search-form').addEventListener('submit', (e) => {
            e.preventDefault();
            const query = document.getElementById('location-input').value.trim();
            if(query) {
                fetchWeather(query);
            }
        });

        async function fetchWeather(query) {
            let url;
            // Check if query is a pincode (for India)
            if (/^\d{6}$/.test(query)) {
                // --- CORRECTION 2: Removed semicolon from inside URL string ---
                url = `https://api.openweathermap.org/data/2.5/weather?zip=${query},IN&appid=${OWM_API_KEY}&units=metric`;
            } else if (typeof query === 'string') {
                 // --- CORRECTION 3: Removed semicolon from inside URL string ---
                 url = `https://api.openweathermap.org/data/2.5/weather?q=${query}&appid=${OWM_API_KEY}&units=metric`;
            } else { // Assumes query is a coordinate object
                 // --- CORRECTION 4: Removed semicolon from inside URL string ---
                 url = `https://api.openweathermap.org/data/2.5/weather?lat=${query.lat}&lon=${query.lon}&appid=${OWM_API_KEY}&units=metric`;
            }
            
            try {
                const response = await fetch(url);
                if(!response.ok) throw new Error("Weather data not found for this location.");
                const data = await response.json();
                
                // Update main dashboard card
                document.getElementById('location').textContent = data.name;
                document.getElementById('weather-description').textContent = data.weather[0].main;
                document.getElementById('temperature').innerHTML = `${Math.round(data.main.temp)}&deg;C`;
                document.getElementById('feels-like').innerHTML = `<i class="fas fa-temperature-half fa-fw mr-1"></i> Feels Like: ${Math.round(data.main.feels_like)}&deg;C`;
                document.getElementById('humidity').innerHTML = `<i class="fas fa-tint fa-fw mr-1"></i> Humidity: ${data.main.humidity}%;`;
                document.getElementById('wind-speed').innerHTML = `<i class="fas fa-wind fa-fw mr-1"></i> Wind: ${Math.round(data.wind.speed * 3.6)} km/h`;
                document.getElementById('weather-icon').className = 'wi wi-day-rain main-icon';
                document.getElementById('sunrise-time').textContent = new Date(data.sys.sunrise * 1000).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
                document.getElementById('sunset-time').textContent = new Date(data.sys.sunset * 1000).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
                document.getElementById('visibility').textContent = `${data.visibility / 1000} km`;
                document.getElementById('pressure').textContent = `${data.main.pressure} hPa`;
                
                const windDir = degreesToCardinal(data.wind.deg);
                document.getElementById('wind-direction').textContent = windDir;
                document.getElementById('wind-direction-arrow').style.transform = `rotate(${data.wind.deg}deg)`;


                // Update weather ticker
                document.getElementById('ticker-location').textContent = data.name;
                document.getElementById('ticker-temp').textContent = `${Math.round(data.main.temp)}°C`;
                document.getElementById('ticker-desc').textContent = `, ${data.weather[0].main}`;
                
                // Update map view
                if(map) {
                    const coords = [data.coord.lat, data.coord.lon];
                    map.setView(coords, 10);
                    if (userMarker) {
                        userMarker.setLatLng(coords);
                    } else {
                        userMarker = L.marker(coords).addTo(map);
                    }
                    userMarker.bindPopup(`<b>${data.name}</b>`).openPopup();
                }

            } catch (error) {
                console.error("Error fetching weather:", error);
                document.getElementById('weather-description').textContent = error.message;
            }
        }
        function displayGeoAlerts() {
            const container = document.getElementById('geo-alerts-container');
            container.innerHTML = ''; 
            for (const area in floodProneAreas) {
                const alertData = floodProneAreas[area];
                const alertEl = document.createElement('div');
                alertEl.className = `p-4 rounded-lg flex items-start space-x-3 alert-${alertData.level}`;
                
                const iconClass = alertData.level === 'high' ? 'fa-triangle-exclamation' : (alertData.level === 'medium' ? 'fa-cloud-showers-heavy' : 'fa-info-circle');

                alertEl.innerHTML = `<i class="fas ${iconClass} mt-1"></i><div><h3 class="font-bold">${area}</h3><p class="text-sm">${alertData.message}</p></div>`;
                container.appendChild(alertEl);
            }
        }
        function displayScrollingAlerts() {
            const container = document.getElementById('alert-ticker-content');
            container.innerHTML = ''; 

            let alertText = '';
            for (const area in floodProneAreas) {
                const alertData = floodProneAreas[area];
                alertText += `<span class="mx-4"><i class="fas fa-exclamation-triangle"></i> <strong>${area}:</strong> ${alertData.message}</span>`;
            }
            container.innerHTML = alertText;
        }
        function displayEmergencyContacts() {
            const container = document.getElementById('emergency-contacts-container');
            container.innerHTML = '';
            emergencyContacts.forEach(contact => {
                const contactEl = document.createElement('a');
                contactEl.href = `tel:${contact.number}`;
                contactEl.target = "_blank";
                contactEl.rel = "noopener noreferrer";
                contactEl.className = 'bg-gray-100 p-4 rounded-lg flex items-center space-x-4 hover:bg-gray-200 transition';
                contactEl.innerHTML = `<div class="bg-blue-100 text-blue-600 rounded-full h-12 w-12 flex items-center justify-center shrink-0"><i class="fas ${contact.icon} text-xl"></i></div><div><p class="font-semibold text-gray-800">${contact.name}</p><p class="text-gray-600">${contact.number}</p></div>`;
                container.appendChild(contactEl);
            });
        }
        function setupCommunityFeed() {
            const feedContainer = document.getElementById('feed-container');
            const postForm = document.getElementById('post-form');
            const postContent = document.getElementById('post-content');

            const q = query(collection(db, "posts"), orderBy("timestamp", "desc"));
            onSnapshot(q, (querySnapshot) => {
                feedContainer.innerHTML = '';
                if (querySnapshot.empty) {
                    feedContainer.innerHTML = `<p class="text-center text-gray-500">No community updates yet. Be the first to post!</p>`;
                }
                querySnapshot.forEach((doc) => {
                    const post = doc.data();
                    const postEl = document.createElement('div');
                    postEl.className = 'bg-white p-4 rounded-lg shadow-sm';
                    postEl.innerHTML = `<p class="text-gray-800 break-words">${post.content}</p><div class="text-right text-xs text-gray-400 mt-2"><span>${post.user.substring(0, post.user.indexOf('@'))}</span> | <span>${formatTimeAgo(post.timestamp?.toDate())}</span></div>`;
                    feedContainer.appendChild(postEl);
                });
            });

            postForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const content = postContent.value.trim();
                if (content && auth.currentUser) {
                    await addDoc(collection(db, "posts"), { content: content, user: auth.currentUser.email, timestamp: serverTimestamp() });
                    postContent.value = '';
                }
            });
        }
        function updateTime() {
            const now = new Date();
            document.getElementById('current-date').textContent = now.toLocaleDateString(undefined, { weekday: 'long', month: 'long', day: 'numeric' });
            document.getElementById('current-time').textContent = now.toLocaleTimeString(undefined, { hour: '2-digit', minute:'2-digit' });
        }
        function formatTimeAgo(date) { if (!date) return 'just now'; const seconds = Math.floor((new Date() - date) / 1000); if(seconds < 5) return 'just now'; let interval = seconds / 60; if (interval < 60) return Math.floor(interval) + " min ago"; interval = seconds / 3600; if (interval < 24) return Math.floor(interval) + " hr ago"; interval = seconds / 86400; if (interval < 30) return Math.floor(interval) + " days ago"; return date.toLocaleDateString(); }

        // --- Geolocation ---
        function getCurrentLocation() {
            if (navigator.geolocation) {
                navigator.geolocation.getCurrentPosition(position => {
                    const { latitude, longitude } = position.coords;
                    fetchWeather({ lat: latitude, lon: longitude });
                }, () => {
                    // If user denies location, fetch default
                    fetchWeather('Hyderabad');
                    console.log("User denied geolocation access.");
                });
            } else {
                 fetchWeather('Hyderabad');
                 console.log("Geolocation is not supported by this browser.");
            }
        }
        document.getElementById('find-location-btn').addEventListener('click', getCurrentLocation);

        // --- Map Logic ---
        function initializeMap() {
            if (map) return; // Don't re-initialize
            map = L.map('weather-map').setView([17.3850, 78.4867], 10); // Default to Hyderabad

            const baseLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
            }).addTo(map);
            
            // --- CORRECTION 5: Added missing backtick for template literal ---
            const precipitationLayer = L.tileLayer(`https://tile.openweathermap.org/map/precipitation_new/{z}/{x}/{y}.png?appid=${OWM_API_KEY}`);
            // --- CORRECTION 6: Added missing backtick for template literal ---
            const windLayer = L.tileLayer(`https://tile.openweathermap.org/map/wind_new/{z}/{x}/{y}.png?appid=${OWM_API_KEY}`);
            
            const overlayMaps = {
                "Precipitation": precipitationLayer,
                "Wind Speed": windLayer
            };

            L.control.layers({ "Base Map": baseLayer }, overlayMaps, { position: 'topright' }).addTo(map);
            precipitationLayer.addTo(map); // Show precipitation by default
            
            // Add legend
            const legend = L.control({ position: 'bottomright' });
            legend.onAdd = function (map) {
                const div = L.DomUtil.create('div', 'info legend');
                div.innerHTML = `
                    <h4>Precipitation (mm/h)</h4>
                    <i style="background: #a0d2f0"></i> 0-1 (Light)<br>
                    <i style="background: #4682b4"></i> 1-5 (Moderate)<br>
                    <i style="background: #1e3a8a"></i> 5-10 (Heavy)<br>
                    <i style="background: #9333ea"></i> >10 (Extreme)<br>
                    <h4 class="mt-2">Wind (km/h)</h4>
                    <i style="background: #90ee90"></i> < 20 (Light)<br>
                    <i style="background: #ffd700"></i> 20-50 (Moderate)<br>
                    <i style="background: #ff4500"></i> > 50 (Strong)<br>
                `;
                return div;
            };
            legend.addTo(map);
        }

        // --- Gemini API Integration ---
        const safetyTipsBtn = document.getElementById('safety-tips-btn');
        const summarizeFeedBtn = document.getElementById('summarize-feed-btn');
        const forecastBtn = document.getElementById('forecast-btn');
        const hourlyOutlookBtn = document.getElementById('hourly-outlook-btn');
        const allergyOutlookBtn = document.getElementById('allergy-outlook-btn');
        const closeGeminiModalBtn = document.getElementById('close-gemini-modal');
        closeGeminiModalBtn.addEventListener('click', () => geminiModal.classList.add('hidden'));

        async function callGeminiAPI(prompt) {
            // --- CORRECTION 7: Removed semicolon from inside URL string ---
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${GEMINI_API_KEY}`;
            const payload = { contents: [{ parts: [{ text: prompt }] }] };
            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                if (!response.ok) throw new Error(`API Error: ${response.statusText}`);
                const result = await response.json();
                return result.candidates[0].content.parts[0].text;
            } catch (error) {
                console.error("Gemini API call failed:", error);
                return `An error occurred while contacting the AI model: ${error.message}`;
            }
        }

        function showGeminiModal(title) {
            geminiModal.classList.remove('hidden');
            document.getElementById('gemini-modal-title').textContent = title;
            document.getElementById('gemini-spinner').classList.remove('hidden');
            document.getElementById('gemini-response-text').innerHTML = '';
        }

        function displayGeminiResponse(htmlContent) {
            document.getElementById('gemini-spinner').classList.add('hidden');
            // --- CORRECTION 8: Fixed regex to correctly match **bold** text without conflicting with * list items ---
            const formattedContent = htmlContent
                .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>') // Bold (for **text**)
                .replace(/\* (.*?)(?:\n|<br>|$)/g, '<li class="ml-4 list-disc">$1</li>') // List items
                .replace(/\n/g, '<br>');
            document.getElementById('gemini-response-text').innerHTML = formattedContent;
        }

        safetyTipsBtn.addEventListener('click', async () => {
            showGeminiModal('AI-Powered Safety Tips ✨');
            const city = document.getElementById('location').textContent;
            const weather = document.getElementById('weather-description').textContent;
            const temp = document.getElementById('temperature').textContent;
            const prompt = `Act as a disaster management expert for ${city}. The current weather is "${weather}" with a temperature of ${temp}. Provide 5 concise, actionable safety tips for residents. Focus on immediate safety during these conditions. Format as a simple list.`;
            
            const response = await callGeminiAPI(prompt);
            displayGeminiResponse(response);
        });
        
        hourlyOutlookBtn.addEventListener('click', async () => {
             showGeminiModal('AI-Powered Hourly Outlook ✨');
            const city = document.getElementById('location').textContent;
            const weather = document.getElementById('weather-description').textContent;
            const temp = document.getElementById('temperature').textContent;
            const prompt = `Act as a professional meteorologist. The current weather in ${city} is "${weather}" with a temperature of ${temp}. Provide a narrative-style hourly outlook for the next 4-6 hours. Describe expected changes in weather, temperature, and wind. Be descriptive and reassuring.`;
            
            const response = await callGeminiAPI(prompt);
            displayGeminiResponse(response);
        });
        
        allergyOutlookBtn.addEventListener('click', async () => {
            showGeminiModal('AI-Powered Allergy Outlook ✨');
            const city = document.getElementById('location').textContent;
            const weather = document.getElementById('weather-description').textContent;
            const humidity = document.getElementById('humidity').textContent;
            const wind = document.getElementById('wind-speed').textContent;
            const prompt = `Act as an allergist providing a forecast for ${city}. Current conditions are: ${weather}, ${humidity}, and ${wind}. Based on this, provide an allergy outlook for today. Mention primary allergens like pollen and mold, and give a risk level (Low, Moderate, High, Very High). Offer one practical tip.`;
            
            const response = await callGeminiAPI(prompt);
            displayGeminiResponse(response);
        });

        forecastBtn.addEventListener('click', async () => {
            showGeminiModal('AI-Powered 3-Day Forecast ✨');
            const city = document.getElementById('location').textContent;
            const prompt = `Provide a simple, user-friendly 3-day weather forecast for ${city}. For each day (Today, Tomorrow, Day After), give a short description of the expected weather, the approximate high and low temperatures in Celsius, and a brief "Advice" section with one practical tip (e.g., 'Carry an umbrella,' 'Plan for indoor activities'). Format it clearly with bold headings for each day.`;

            const response = await callGeminiAPI(prompt);
            displayGeminiResponse(response);
        });

        summarizeFeedBtn.addEventListener('click', async () => {
            showGeminiModal('AI Community Feed Summary ✨');
            
            const q = query(collection(db, "posts"), orderBy("timestamp", "desc"), limit(15));
            const querySnapshot = await getDocs(q);
            let postsContent = "";
            querySnapshot.forEach(doc => { postsContent += `- ${doc.data().content}\n`; });

            if (postsContent.trim() === "") {
                displayGeminiResponse("There are no posts in the feed to summarize yet.");
                return;
            }

            const prompt = `You are a helpful city safety assistant for Hyderabad. The following are real-time updates from citizens during a monsoon. Read all the updates and provide a concise summary (2-3 sentences) of the overall situation, mentioning key affected areas or recurring problems (like waterlogging or traffic). Here are the updates:\n\n${postsContent}`;
            
            const response = await callGeminiAPI(prompt);
            displayGeminiResponse(response);
        });

        // --- SOS Button ---
        document.getElementById('sos-btn').addEventListener('click', () => {
            document.querySelector('[data-page="emergency"]').click();
        });

        // --- Page Navigation ---
        const navLinks = document.querySelectorAll('.nav-link');
        const pages = document.querySelectorAll('.page');
        navLinks.forEach(link => {
            link.addEventListener('click', () => {
                const pageId = link.dataset.page;

                navLinks.forEach(l => l.classList.remove('nav-link-active', 'text-gray-700') || l.classList.add('text-gray-500'));
                link.classList.add('nav-link-active', 'text-gray-700');
                link.classList.remove('text-gray-500');
                
                pages.forEach(page => {
                    if (page.id === `${pageId}-page`) {
                        page.classList.remove('hidden');
                    } else {
                        page.classList.add('hidden');
                    }
                });
                
                // Invalidate map size to fix rendering issues when showing the map page
                if (pageId === 'map' && map) {
                    setTimeout(() => map.invalidateSize(), 10);
                }
            });
        });

    </script>
</body>
</html>
