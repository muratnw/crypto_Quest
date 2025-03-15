<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CryptoQuest</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        primary: '#5D5CDE',
                        secondary: '#8584E4',
                        telegram: '#0088cc',
                        dark: {
                            bg: '#181818',
                            card: '#222222',
                            text: '#e2e2e2'
                        }
                    }
                }
            }
        }
        
        // Karanlık mod kontrolü
        if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
            document.documentElement.classList.add('dark');
        }
        window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', event => {
            if (event.matches) {
                document.documentElement.classList.add('dark');
            } else {
                document.documentElement.classList.remove('dark');
            }
        });
    </script>
    <style>
        @keyframes pulse-border {
            0% { box-shadow: 0 0 0 0 rgba(93, 92, 222, 0.7); }
            70% { box-shadow: 0 0 0 15px rgba(93, 92, 222, 0); }
            100% { box-shadow: 0 0 0 0 rgba(93, 92, 222, 0); }
        }
        
        @keyframes coin-float {
            0% { transform: translateY(0) rotate(0deg); opacity: 1; }
            100% { transform: translateY(-100px) rotate(360deg); opacity: 0; }
        }
        
        .pulse-animation {
            animation: pulse-border 1.5s infinite;
        }
        
        .coin {
            position: absolute;
            animation: coin-float 1s ease-out forwards;
            z-index: 10;
            user-select: none;
        }
        
        .progress-ring {
            transition: stroke-dashoffset 0.35s;
            transform: rotate(-90deg);
            transform-origin: 50% 50%;
        }
        
        .tab-content {
            display: none;
        }
        
        .tab-content.active {
            display: block;
        }
        
        .tap-area {
            touch-action: manipulation;
        }
        
        /* Glow efekti */
        .glow {
            box-shadow: 0 0 15px 3px rgba(93, 92, 222, 0.6);
        }
        
        .dark .glow {
            box-shadow: 0 0 15px 3px rgba(93, 92, 222, 0.8);
        }
        
        /* Ripple efekti */
        .ripple {
            position: relative;
            overflow: hidden;
        }
        
        .ripple:after {
            content: "";
            display: block;
            position: absolute;
            width: 100%;
            height: 100%;
            top: 0;
            left: 0;
            pointer-events: none;
            background-image: radial-gradient(circle, #fff 10%, transparent 10.01%);
            background-repeat: no-repeat;
            background-position: 50%;
            transform: scale(10, 10);
            opacity: 0;
            transition: transform 0.5s, opacity 1s;
        }
        
        .ripple:active:after {
            transform: scale(0, 0);
            opacity: 0.3;
            transition: 0s;
        }
        
        /* Token anim */
        @keyframes token-pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }
        
        .token-pulse {
            animation: token-pulse 0.5s ease;
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-dark-bg text-gray-800 dark:text-dark-text min-h-screen">
    <!-- Header -->
    <header class="bg-white dark:bg-dark-card shadow-md fixed top-0 left-0 right-0 z-50">
        <div class="container mx-auto p-4 flex justify-between items-center">
            <div class="flex items-center">
                <h1 class="text-xl font-bold text-primary">CryptoQuest</h1>
                <span class="ml-2 px-2 py-0.5 bg-primary text-white text-xs rounded-full">Beta</span>
            </div>
            <div class="flex items-center">
                <div class="mr-4 flex items-center bg-gray-100 dark:bg-gray-800 px-3 py-1 rounded-full">
                    <img src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEyIDIyQzE3LjUyMjggMjIgMjIgMTcuNTIyOCAyMiAxMkMyMiA2LjQ3NzE1IDE3LjUyMjggMiAxMiAyQzYuNDc3MTUgMiAyIDYuNDc3MTUgMiAxMkMyIDE3LjUyMjggNi40NzcxNSAyMiAxMiAyMloiIGZpbGw9IiM1RDVDREUiLz4KPHBhdGggZD0iTTE1IDlMMTIgMTJNMTIgMTJMOSA5TTEyIDEyTDE1IDE1TTEyIDEyTDkgMTUiIHN0cm9rZT0id2hpdGUiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIi8+Cjwvc3ZnPgo=" alt="CQX" class="w-5 h-5 mr-1" />
                    <span class="font-bold" id="tokenCount">0</span>
                    <span class="ml-1 text-xs text-gray-500 dark:text-gray-400">CQX</span>
                </div>
                <div class="text-sm font-semibold mr-2">Lvl <span id="userLevel">1</span></div>
            </div>
        </header>
    
    <!-- Main Content with padding top for fixed header -->
    <main class="container mx-auto p-4 pt-20 pb-24">
        <!-- Token Earning Section -->
        <div id="homeTab" class="tab-content active">
            <div class="text-center mb-6">
                <div class="relative mx-auto w-64 h-64 flex items-center justify-center mb-4">
                    <!-- Energy Ring -->
                    <svg class="absolute" width="240" height="240">
                        <circle cx="120" cy="120" r="100" fill="none" stroke="#e6e6e6" stroke-width="10" />
                        <circle id="energyRing" class="progress-ring" cx="120" cy="120" r="100" fill="none" stroke="#5D5CDE" stroke-width="10" stroke-dasharray="628" stroke-dashoffset="628" />
                    </svg>
                    
                    <!-- Tap Area -->
                    <div id="tapArea" class="tap-area ripple w-48 h-48 rounded-full bg-white dark:bg-dark-card flex items-center justify-center shadow-lg cursor-pointer z-10">
                        <div class="text-center">
                            <img src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNjQiIGhlaWdodD0iNjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEyIDIyQzE3LjUyMjggMjIgMjIgMTcuNTIyOCAyMiAxMkMyMiA2LjQ3NzE1IDE3LjUyMjggMiAxMiAyQzYuNDc3MTUgMiAyIDYuNDc3MTUgMiAxMkMyIDE3LjUyMjggNi40NzcxNSAyMiAxMiAyMloiIGZpbGw9IiM1RDVDREUiLz4KPHBhdGggZD0iTTE1IDlMMTIgMTJNMTIgMTJMOSA5TTEyIDEyTDE1IDE1TTEyIDEyTDkgMTUiIHN0cm9rZT0id2hpdGUiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIi8+Cjwvc3ZnPgo=" alt="CQX Token" class="w-16 h-16 mx-auto mb-2">
                            <p class="text-gray-500 dark:text-gray-400 text-sm">Dokunarak CQX kazan</p>
                        </div>
                    </div>
                </div>
                
                <div class="mb-6">
                    <div class="flex justify-between text-sm text-gray-600 dark:text-gray-400 mb-2">
                        <span>Kalan Enerji:</span>
                        <span id="energyText">100/100</span>
                    </div>
                    <div class="bg-gray-200 dark:bg-gray-700 rounded-full h-2">
                        <div id="energyBar" class="bg-primary h-2 rounded-full" style="width: 100%"></div>
                    </div>
                </div>
            </div>
            
            <!-- Daily Bonuses -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4 mb-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-calendar-day mr-2 text-primary"></i>
                    Günlük Bonuslar
                </h3>
                <div class="grid grid-cols-7 gap-2">
                    <div class="day-box bg-primary/10 rounded p-2 text-center relative">
                        <div class="text-xs">Pzt</div>
                        <div class="text-sm font-bold">+10</div>
                        <div class="absolute -top-1 -right-1 w-4 h-4 bg-primary rounded-full flex items-center justify-center">
                            <i class="fas fa-check text-white text-xs"></i>
                        </div>
                    </div>
                    <div class="day-box bg-primary/10 rounded p-2 text-center relative">
                        <div class="text-xs">Sal</div>
                        <div class="text-sm font-bold">+15</div>
                        <div class="absolute -top-1 -right-1 w-4 h-4 bg-primary rounded-full flex items-center justify-center">
                            <i class="fas fa-check text-white text-xs"></i>
                        </div>
                    </div>
                    <div class="day-box bg-gray-100 dark:bg-gray-700 rounded p-2 text-center">
                        <div class="text-xs">Çar</div>
                        <div class="text-sm font-bold">+20</div>
                    </div>
                    <div class="day-box bg-gray-100 dark:bg-gray-700 rounded p-2 text-center">
                        <div class="text-xs">Per</div>
                        <div class="text-sm font-bold">+25</div>
                    </div>
                    <div class="day-box bg-gray-100 dark:bg-gray-700 rounded p-2 text-center">
                        <div class="text-xs">Cum</div>
                        <div class="text-sm font-bold">+30</div>
                    </div>
                    <div class="day-box bg-gray-100 dark:bg-gray-700 rounded p-2 text-center">
                        <div class="text-xs">Cmt</div>
                        <div class="text-sm font-bold">+40</div>
                    </div>
                    <div class="day-box bg-gray-100 dark:bg-gray-700 rounded p-2 text-center relative overflow-hidden">
                        <div class="text-xs">Paz</div>
                        <div class="text-sm font-bold">+100</div>
                        <div class="absolute -right-4 -bottom-4 w-8 h-8 bg-yellow-500 rotate-45"></div>
                    </div>
                </div>
                <div class="mt-3 text-center">
                    <button id="claimDailyBtn" class="bg-primary text-white px-4 py-2 rounded-lg text-sm font-medium">
                        Günlük Bonusu Topla
                    </button>
                </div>
            </div>
            
            <!-- Recent Activity -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-history mr-2 text-primary"></i>
                    Son Aktiviteler
                </h3>
                <ul class="divide-y divide-gray-200 dark:divide-gray-700" id="activityList">
                    <li class="py-2 flex justify-between items-center">
                        <div class="flex items-center">
                            <div class="bg-green-100 dark:bg-green-900 text-green-800 dark:text-green-200 p-1 rounded">
                                <i class="fas fa-hand-pointer text-xs"></i>
                            </div>
                            <span class="ml-2 text-sm">Dokunarak kazanıldı</span>
                        </div>
                        <div class="text-green-600 dark:text-green-400 text-sm font-medium">+1 CQX</div>
                    </li>
                    <li class="py-2 flex justify-between items-center">
                        <div class="flex items-center">
                            <div class="bg-blue-100 dark:bg-blue-900 text-blue-800 dark:text-blue-200 p-1 rounded">
                                <i class="fas fa-star text-xs"></i>
                            </div>
                            <span class="ml-2 text-sm">Seviye atladınız</span>
                        </div>
                        <div class="text-blue-600 dark:text-blue-400 text-sm font-medium">Lvl 1</div>
                    </li>
                </ul>
            </div>
        </div>
        
        <!-- Quests Tab -->
        <div id="questsTab" class="tab-content">
            <div class="mb-4">
                <h2 class="text-xl font-bold mb-2">Görevler</h2>
                <p class="text-sm text-gray-600 dark:text-gray-400">Görevleri tamamlayarak daha fazla CQX kazanın!</p>
            </div>
            
            <!-- Daily Quests -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4 mb-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-calendar-day mr-2 text-primary"></i>
                    Günlük Görevler
                </h3>
                <ul class="divide-y divide-gray-200 dark:divide-gray-700">
                    <li class="py-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <h4 class="font-medium">Sosyal Medyada Paylaş</h4>
                                <p class="text-sm text-gray-600 dark:text-gray-400">CryptoQuest'i Twitter'da paylaş</p>
                            </div>
                            <div class="flex items-center">
                                <span class="mr-2 text-sm font-bold">+50 CQX</span>
                                <button class="quest-btn bg-primary text-white px-3 py-1 rounded text-sm">Tamamla</button>
                            </div>
                        </div>
                        <div class="mt-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5">
                            <div class="bg-primary h-1.5 rounded-full" style="width: 0%"></div>
                        </div>
                    </li>
                    <li class="py-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <h4 class="font-medium">Token Topla</h4>
                                <p class="text-sm text-gray-600 dark:text-gray-400">10 defa dokunarak token kazan</p>
                            </div>
                            <div class="flex items-center">
                                <span class="mr-2 text-sm font-bold">+20 CQX</span>
                                <div class="text-xs text-gray-500">0/10</div>
                            </div>
                        </div>
                        <div class="mt-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5">
                            <div class="bg-primary h-1.5 rounded-full" style="width: 0%"></div>
                        </div>
                    </li>
                    <li class="py-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <h4 class="font-medium">Arkadaş Davet Et</h4>
                                <p class="text-sm text-gray-600 dark:text-gray-400">Bir arkadaşını uygulamaya davet et</p>
                            </div>
                            <div class="flex items-center">
                                <span class="mr-2 text-sm font-bold">+100 CQX</span>
                                <button class="quest-btn bg-primary text-white px-3 py-1 rounded text-sm">Davet Et</button>
                            </div>
                        </div>
                        <div class="mt-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5">
                            <div class="bg-primary h-1.5 rounded-full" style="width: 0%"></div>
                        </div>
                    </li>
                </ul>
            </div>
            
            <!-- Weekly Quests -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-calendar-week mr-2 text-primary"></i>
                    Haftalık Görevler
                </h3>
                <ul class="divide-y divide-gray-200 dark:divide-gray-700">
                    <li class="py-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <h4 class="font-medium">7 Gün Bonusu</h4>
                                <p class="text-sm text-gray-600 dark:text-gray-400">7 gün üst üste giriş yap</p>
                                <div class="flex mt-1 space-x-1">
                                    <div class="w-4 h-4 rounded-full bg-primary"></div>
                                    <div class="w-4 h-4 rounded-full bg-primary"></div>
                                    <div class="w-4 h-4 rounded-full bg-gray-300 dark:bg-gray-700"></div>
                                    <div class="w-4 h-4 rounded-full bg-gray-300 dark:bg-gray-700"></div>
                                    <div class="w-4 h-4 rounded-full bg-gray-300 dark:bg-gray-700"></div>
                                    <div class="w-4 h-4 rounded-full bg-gray-300 dark:bg-gray-700"></div>
                                    <div class="w-4 h-4 rounded-full bg-gray-300 dark:bg-gray-700"></div>
                                </div>
                            </div>
                            <div class="flex items-center">
                                <span class="mr-2 text-sm font-bold">+500 CQX</span>
                                <div class="text-xs text-gray-500">2/7</div>
                            </div>
                        </div>
                        <div class="mt-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5">
                            <div class="bg-primary h-1.5 rounded-full" style="width: 28%"></div>
                        </div>
                    </li>
                    <li class="py-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <h4 class="font-medium">Mini Oyunda Rekor Kır</h4>
                                <p class="text-sm text-gray-600 dark:text-gray-400">Mini oyunda en az 100 puan al</p>
                            </div>
                            <div class="flex items-center">
                                <span class="mr-2 text-sm font-bold">+200 CQX</span>
                                <button class="quest-btn bg-primary text-white px-3 py-1 rounded text-sm">Oyna</button>
                            </div>
                        </div>
                        <div class="mt-1 w-full bg-gray-200 dark:bg-gray-700 rounded-full h-1.5">
                            <div class="bg-primary h-1.5 rounded-full" style="width: 0%"></div>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
        
        <!-- Leaderboard Tab -->
        <div id="leaderboardTab" class="tab-content">
            <div class="mb-4">
                <h2 class="text-xl font-bold mb-2">Liderlik Tablosu</h2>
                <p class="text-sm text-gray-600 dark:text-gray-400">En çok CQX toplayan kullanıcılar</p>
            </div>
            
            <!-- Rank Controls -->
            <div class="flex space-x-2 mb-4">
                <button class="rank-btn active bg-primary text-white px-3 py-1 rounded-lg text-sm font-medium" data-period="daily">Günlük</button>
                <button class="rank-btn bg-gray-200 dark:bg-gray-700 px-3 py-1 rounded-lg text-sm font-medium" data-period="weekly">Haftalık</button>
                <button class="rank-btn bg-gray-200 dark:bg-gray-700 px-3 py-1 rounded-lg text-sm font-medium" data-period="alltime">Tüm Zamanlar</button>
            </div>
            
            <!-- Top 3 Players -->
            <div class="flex justify-center items-end mb-8 space-x-4">
                <!-- 2nd Place -->
                <div class="text-center">
                    <div class="relative">
                        <div class="absolute -top-3 left-1/2 transform -translate-x-1/2 bg-gray-300 text-gray-800 rounded-full w-6 h-6 flex items-center justify-center text-sm font-bold">2</div>
                        <div class="w-16 h-16 bg-gray-200 dark:bg-gray-700 rounded-full flex items-center justify-center mb-2 mx-auto">
                            <i class="fas fa-user text-gray-500 text-xl"></i>
                        </div>
                    </div>
                    <p class="text-sm font-medium truncate w-20 mx-auto">Ahmet K.</p>
                    <p class="text-xs text-gray-500 dark:text-gray-400">1,250 CQX</p>
                </div>
                
                <!-- 1st Place -->
                <div class="text-center">
                    <div class="relative">
                        <div class="absolute -top-4 left-1/2 transform -translate-x-1/2 bg-yellow-400 text-yellow-800 rounded-full w-8 h-8 flex items-center justify-center text-lg font-bold">1</div>
                        <div class="w-20 h-20 bg-primary/20 rounded-full flex items-center justify-center mb-2 mx-auto border-2 border-primary">
                            <i class="fas fa-crown text-yellow-500 text-xl"></i>
                        </div>
                    </div>
                    <p class="text-sm font-medium truncate w-24 mx-auto">Mehmet Y.</p>
                    <p class="text-xs text-gray-500 dark:text-gray-400">2,540 CQX</p>
                </div>
                
                <!-- 3rd Place -->
                <div class="text-center">
                    <div class="relative">
                        <div class="absolute -top-3 left-1/2 transform -translate-x-1/2 bg-amber-700 text-amber-100 rounded-full w-6 h-6 flex items-center justify-center text-sm font-bold">3</div>
                        <div class="w-16 h-16 bg-gray-200 dark:bg-gray-700 rounded-full flex items-center justify-center mb-2 mx-auto">
                            <i class="fas fa-user text-gray-500 text-xl"></i>
                        </div>
                    </div>
                    <p class="text-sm font-medium truncate w-20 mx-auto">Ayşe T.</p>
                    <p class="text-xs text-gray-500 dark:text-gray-400">980 CQX</p>
                </div>
            </div>
            
            <!-- Leaderboard List -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md">
                <ul class="divide-y divide-gray-200 dark:divide-gray-700">
                    <li class="p-4 flex items-center">
                        <div class="w-6 text-center font-bold">4</div>
                        <div class="w-10 h-10 bg-gray-200 dark:bg-gray-700 rounded-full flex items-center justify-center mx-4">
                            <i class="fas fa-user text-gray-500"></i>
                        </div>
                        <div class="flex-1">
                            <p class="font-medium">Zeynep K.</p>
                            <p class="text-xs text-gray-500 dark:text-gray-400">Seviye 15</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold">870 CQX</p>
                        </div>
                    </li>
                    <li class="p-4 flex items-center">
                        <div class="w-6 text-center font-bold">5</div>
                        <div class="w-10 h-10 bg-gray-200 dark:bg-gray-700 rounded-full flex items-center justify-center mx-4">
                            <i class="fas fa-user text-gray-500"></i>
                        </div>
                        <div class="flex-1">
                            <p class="font-medium">Ali R.</p>
                            <p class="text-xs text-gray-500 dark:text-gray-400">Seviye 12</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold">720 CQX</p>
                        </div>
                    </li>
                    <li class="p-4 flex items-center bg-primary/10">
                        <div class="w-6 text-center font-bold">18</div>
                        <div class="w-10 h-10 bg-primary/20 rounded-full flex items-center justify-center mx-4 border border-primary">
                            <i class="fas fa-user text-primary"></i>
                        </div>
                        <div class="flex-1">
                            <p class="font-medium">Sen</p>
                            <p class="text-xs text-gray-500 dark:text-gray-400">Seviye 1</p>
                        </div>
                        <div class="text-right">
                            <p class="font-bold"><span id="leaderboardUserScore">0</span> CQX</p>
                        </div>
                    </li>
                </ul>
            </div>
        </div>
        
        <!-- Invite Tab -->
        <div id="inviteTab" class="tab-content">
            <div class="mb-4">
                <h2 class="text-xl font-bold mb-2">Arkadaş Davet Et</h2>
                <p class="text-sm text-gray-600 dark:text-gray-400">Arkadaşlarını davet et, daha fazla CQX kazan!</p>
            </div>
            
            <!-- Invite Card -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4 mb-4">
                <div class="text-center mb-4">
                    <img src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTI4IiBoZWlnaHQ9IjEyOCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgo8cGF0aCBkPSJNMTUgMTZDMTUgMTUuNTkyMiAxNC44NDIgMTUuMjAxIDE0LjU2MDcgMTQuOTA5OEMxNC4yNzk0IDE0LjYxODYgMTMuODk3OCAxNC40NSAxMy41IDE0LjQ1SDEwLjVDMTAuMTAyMiAxNC40NSA5LjcyMDY0IDE0LjYxODYgOS40MzkzNCAxNC45MDk4QzkuMTU4MDQgMTUuMjAxIDkgMTUuNTkyMiA5IDE2IiBzdHJva2U9IiM1RDVDREUiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW49InJvdW5kIi8+CjxwYXRoIGQ9Ik0xMiAxMkMxMy4xMDQ2IDEyIDE0IDExLjEwNDYgMTQgMTBDMTQgOC44OTU0MyAxMy4xMDQ2IDggMTIgOEMxMC44OTU0IDggMTAgOC44OTU0MyAxMCAxMEMxMCAxMS4xMDQ2IDEwLjg5NTQgMTIgMTIgMTJaIiBmaWxsPSIjNUQ1Q0RFIi8+CjxwYXRoIGQ9Ik0xMiAyMkMxNy41MjI4IDIyIDIyIDE3LjUyMjggMjIgMTJDMjIgNi40NzcxNSAxNy41MjI4IDIgMTIgMkM2LjQ3NzE1IDIgMiA2LjQ3NzE1IDIgMTJDMiAxNy41MjI4IDYuNDc3MTUgMjIgMTIgMjJaIiBzdHJva2U9IiM1RDVDREUiIHN0cm9rZS13aWR0aD0iMiIvPgo8cGF0aCBkPSJNMTguMjMwOSA3LjVMMTkuNSA4Ljc2OTA1IiBzdHJva2U9IiM1RDVDREUiIHN0cm9rZS13aWR0aD0iMiIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIi8+CjxwYXRoIGQ9Ik00Ljc2OTA1IDcuNUwzLjUgOC43NjkwNSIgc3Ryb2tlPSIjNUQ1Q0RFIiBzdHJva2Utd2lkdGg9IjIiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIvPgo8L3N2Zz4K" alt="Invite Friends" class="w-24 h-24 mx-auto mb-2">
                    <h3 class="text-lg font-semibold mb-1">Arkadaşlarını Davet Et, Birlikte Kazan!</h3>
                    <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">Her davet ettiğin arkadaş için kazancının %10'u senin olsun!</p>
                </div>
                
                <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded-lg mb-4">
                    <div class="text-sm mb-2">Davet kodun:</div>
                    <div class="flex">
                        <input type="text" value="CRYPT0Q-XR4F7Z" readonly class="flex-1 bg-white dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-l-lg px-3 py-2 text-base" />
                        <button id="copyInviteBtn" class="bg-primary text-white px-3 py-2 rounded-r-lg text-base">
                            <i class="fas fa-copy"></i>
                        </button>
                    </div>
                </div>
                
                <div class="flex justify-center space-x-3">
                    <button class="flex items-center justify-center bg-blue-500 text-white rounded-lg px-4 py-2 w-full">
                        <i class="fab fa-telegram text-xl mr-2"></i>
                        <span>Telegram</span>
                    </button>
                    <button class="flex items-center justify-center bg-green-500 text-white rounded-lg px-4 py-2 w-full">
                        <i class="fab fa-whatsapp text-xl mr-2"></i>
                        <span>WhatsApp</span>
                    </button>
                </div>
            </div>
            
            <!-- Invite Stats -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4">
                <h3 class="text-lg font-semibold mb-2">Davet İstatistikleri</h3>
                <div class="grid grid-cols-3 gap-4 mb-4">
                    <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded-lg text-center">
                        <div class="text-3xl font-bold text-primary">0</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Davet Edilen</div>
                    </div>
                    <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded-lg text-center">
                        <div class="text-3xl font-bold text-primary">0</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Katılan</div>
                    </div>
                    <div class="bg-gray-100 dark:bg-gray-800 p-3 rounded-lg text-center">
                        <div class="text-3xl font-bold text-primary">0</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Bonus CQX</div>
                    </div>
                </div>
                
                <p class="text-sm text-gray-600 dark:text-gray-400 mb-3">Davet ettiğin kişilerin listesi:</p>
                <div class="bg-gray-100 dark:bg-gray-800 rounded-lg p-4 text-center">
                    <p class="text-gray-500 dark:text-gray-400">Henüz kimseyi davet etmediniz.</p>
                </div>
            </div>
        </div>
        
        <!-- Profile/Settings Tab -->
        <div id="profileTab" class="tab-content">
            <div class="mb-4">
                <h2 class="text-xl font-bold mb-2">Profil</h2>
                <p class="text-sm text-gray-600 dark:text-gray-400">Seviye bilgilerin ve başarıların</p>
            </div>
            
            <!-- User Profile Card -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4 mb-4">
                <div class="flex items-center mb-4">
                    <div class="w-16 h-16 bg-primary/20 rounded-full flex items-center justify-center mr-4 border-2 border-primary">
                        <i class="fas fa-user text-primary text-xl"></i>
                    </div>
                    <div>
                        <h3 class="text-lg font-semibold">Kullanıcı</h3>
                        <div class="flex items-center">
                            <div class="text-xs bg-primary/10 text-primary px-2 py-0.5 rounded-full mr-2">Seviye 1</div>
                            <div class="text-xs bg-gray-100 dark:bg-gray-700 text-gray-600 dark:text-gray-400 px-2 py-0.5 rounded-full">Yeni Başlayan</div>
                        </div>
                    </div>
                </div>
                
                <!-- Level Progress -->
                <div class="mb-4">
                    <div class="flex justify-between text-sm mb-1">
                        <span>Seviye İlerlemesi</span>
                        <span><span id="levelProgress">0</span>/100 XP</span>
                    </div>
                    <div class="bg-gray-200 dark:bg-gray-700 rounded-full h-2">
                        <div id="levelProgressBar" class="bg-primary h-2 rounded-full" style="width: 0%"></div>
                    </div>
                    <div class="flex justify-between text-xs text-gray-500 mt-1">
                        <span>Seviye 1</span>
                        <span>Seviye 2</span>
                    </div>
                </div>
                
                <!-- Stats -->
                <div class="grid grid-cols-3 gap-3 mb-4">
                    <div class="bg-gray-100 dark:bg-gray-800 p-2 rounded-lg text-center">
                        <div class="text-xl font-bold text-primary" id="totalTapsCount">0</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Toplam Tık</div>
                    </div>
                    <div class="bg-gray-100 dark:bg-gray-800 p-2 rounded-lg text-center">
                        <div class="text-xl font-bold text-primary" id="totalDaysCount">1</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Aktif Günler</div>
                    </div>
                    <div class="bg-gray-100 dark:bg-gray-800 p-2 rounded-lg text-center">
                        <div class="text-xl font-bold text-primary" id="totalTokensCount">0</div>
                        <div class="text-xs text-gray-600 dark:text-gray-400">Toplam CQX</div>
                    </div>
                </div>
            </div>
            
            <!-- Achievements -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4 mb-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-trophy mr-2 text-primary"></i>
                    Başarılar
                </h3>
                <div class="grid grid-cols-4 gap-3">
                    <div class="text-center">
                        <div class="w-12 h-12 rounded-full bg-gray-200 dark:bg-gray-700 flex items-center justify-center mx-auto mb-1">
                            <i class="fas fa-hand-pointer text-gray-400"></i>
                        </div>
                        <div class="text-xs">İlk Tık</div>
                    </div>
                    <div class="text-center">
                        <div class="w-12 h-12 rounded-full bg-gray-200 dark:bg-gray-700 flex items-center justify-center mx-auto mb-1">
                            <i class="fas fa-users text-gray-400"></i>
                        </div>
                        <div class="text-xs">Davetçi</div>
                    </div>
                    <div class="text-center">
                        <div class="w-12 h-12 rounded-full bg-gray-200 dark:bg-gray-700 flex items-center justify-center mx-auto mb-1">
                            <i class="fas fa-calendar-check text-gray-400"></i>
                        </div>
                        <div class="text-xs">7 Gün</div>
                    </div>
                    <div class="text-center">
                        <div class="w-12 h-12 rounded-full bg-gray-200 dark:bg-gray-700 flex items-center justify-center mx-auto mb-1">
                            <i class="fas fa-gem text-gray-400"></i>
                        </div>
                        <div class="text-xs">100 CQX</div>
                    </div>
                </div>
            </div>
            
            <!-- Settings -->
            <div class="bg-white dark:bg-dark-card rounded-lg shadow-md p-4">
                <h3 class="text-lg font-semibold mb-2 flex items-center">
                    <i class="fas fa-cog mr-2 text-primary"></i>
                    Ayarlar
                </h3>
                <ul class="divide-y divide-gray-200 dark:divide-gray-700">
                    <li class="py-3 flex justify-between items-center">
                        <div class="flex items-center">
                            <i class="fas fa-bell text-gray-500 mr-3"></i>
                            <span>Bildirimler</span>
                        </div>
                        <div class="relative inline-block w-10 mr-2 align-middle select-none">
                            <input type="checkbox" name="toggle" id="notificationToggle" class="toggle-checkbox absolute block w-6 h-6 rounded-full bg-white border-4 appearance-none cursor-pointer"/>
                            <label for="notificationToggle" class="toggle-label block overflow-hidden h-6 rounded-full bg-gray-300 cursor-pointer"></label>
                        </div>
                    </li>
                    <li class="py-3 flex justify-between items-center">
                        <div class="flex items-center">
                            <i class="fas fa-volume-up text-gray-500 mr-3"></i>
                            <span>Ses Efektleri</span>
                        </div>
                        <div class="relative inline-block w-10 mr-2 align-middle select-none">
                            <input type="checkbox" name="toggle" id="soundToggle" checked class="toggle-checkbox absolute block w-6 h-6 rounded-full bg-white border-4 appearance-none cursor-pointer"/>
                            <label for="soundToggle" class="toggle-label block overflow-hidden h-6 rounded-full bg-gray-300 cursor-pointer"></label>
                        </div>
                    </li>
                    <li class="py-3">
                        <div class="flex items-center">
                            <i class="fas fa-info-circle text-gray-500 mr-3"></i>
                            <span>Uygulama Hakkında</span>
                        </div>
                        <div class="text-xs text-gray-500 mt-1">Versiyon 1.0.0 | CryptoQuest</div>
                    </li>
                </ul>
            </div>
        </div>
    </main>
    
    <!-- Bottom Navigation -->
    <nav class="bg-white dark:bg-dark-card shadow-lg fixed bottom-0 left-0 right-0 z-50">
        <div class="container mx-auto px-4">
            <div class="flex justify-between">
                <button class="tab-btn flex flex-col items-center py-2 w-full text-primary border-t-2 border-primary" data-tab="homeTab">
                    <i class="fas fa-home text-lg"></i>
                    <span class="text-xs mt-1">Ana Sayfa</span>
                </button>
                <button class="tab-btn flex flex-col items-center py-2 w-full text-gray-500 dark:text-gray-400" data-tab="questsTab">
                    <i class="fas fa-tasks text-lg"></i>
                    <span class="text-xs mt-1">Görevler</span>
                </button>
                <button class="tab-btn flex flex-col items-center py-2 w-full text-gray-500 dark:text-gray-400" data-tab="leaderboardTab">
                    <i class="fas fa-trophy text-lg"></i>
                    <span class="text-xs mt-1">Sıralama</span>
                </button>
                <button class="tab-btn flex flex-col items-center py-2 w-full text-gray-500 dark:text-gray-400" data-tab="inviteTab">
                    <i class="fas fa-user-plus text-lg"></i>
                    <span class="text-xs mt-1">Davet Et</span>
                </button>
                <button class="tab-btn flex flex-col items-center py-2 w-full text-gray-500 dark:text-gray-400" data-tab="profileTab">
                    <i class="fas fa-user text-lg"></i>
                    <span class="text-xs mt-1">Profil</span>
                </button>
            </div>
        </div>
    </nav>
    
    <!-- Level Up Modal -->
    <div id="levelUpModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
        <div class="bg-white dark:bg-dark-card p-6 rounded-lg shadow-lg max-w-sm w-full text-center">
            <div class="text-primary text-5xl mb-4">
                <i class="fas fa-star"></i>
            </div>
            <h3 class="text-xl font-bold mb-2">Seviye Atladın!</h3>
            <p class="mb-4">Tebrikler! Seviye <span id="newLevelNum">2</span> oldun.</p>
            <p class="text-sm text-gray-600 dark:text-gray-400 mb-4">Yeni seviye bonusu: <span class="font-bold text-primary">+50 CQX</span></p>
            <button id="closeLevelUpBtn" class="bg-primary hover:bg-opacity-90 text-white font-bold py-2 px-6 rounded-full text-lg">
                Harika!
            </button>
        </div>
    </div>
    
    <!-- Toast Notification -->
    <div id="toast" class="fixed top-4 right-4 bg-green-500 text-white px-4 py-2 rounded shadow-lg z-50 transform translate-x-full transition-transform duration-300">
        <div class="flex items-center">
            <i class="fas fa-check-circle mr-2"></i>
            <span id="toastMessage">İşlem başarılı!</span>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // Değişkenler
            let tokens = 0;
            let energy = 100;
            let maxEnergy = 100;
            let energyRegenRate = 1; // 10 saniyede 1 enerji
            let levelXP = 0;
            let level = 1;
            let xpToNextLevel = 100;
            let totalTaps = 0;
            let totalEarnedTokens = 0;
            
            // DOM elementleri
            const tokenCountEl = document.getElementById('tokenCount');
            const energyBarEl = document.getElementById('energyBar');
            const energyTextEl = document.getElementById('energyText');
            const energyRingEl = document.getElementById('energyRing');
            const tapAreaEl = document.getElementById('tapArea');
            const levelEl = document.getElementById('userLevel');
            const activityListEl = document.getElementById('activityList');
            const tabBtns = document.querySelectorAll('.tab-btn');
            const tabContents = document.querySelectorAll('.tab-content');
            const levelProgressEl = document.getElementById('levelProgress');
            const levelProgressBarEl = document.getElementById('levelProgressBar');
            const totalTapsCountEl = document.getElementById('totalTapsCount');
            const totalDaysCountEl = document.getElementById('totalDaysCount');
            const totalTokensCountEl = document.getElementById('totalTokensCount');
            const userLevelEl = document.getElementById('userLevel');
            const levelUpModalEl = document.getElementById('levelUpModal');
            const newLevelNumEl = document.getElementById('newLevelNum');
            const closeLevelUpBtn = document.getElementById('closeLevelUpBtn');
            const toastEl = document.getElementById('toast');
            const toastMessageEl = document.getElementById('toastMessage');
            const leaderboardUserScoreEl = document.getElementById('leaderboardUserScore');
            const claimDailyBtn = document.getElementById('claimDailyBtn');
            const rankBtns = document.querySelectorAll('.rank-btn');
            const copyInviteBtn = document.getElementById('copyInviteBtn');
            const questBtns = document.querySelectorAll('.quest-btn');
            
            // Enerji halkası güncelleme
            const updateEnergyRing = () => {
                const circumference = 2 * Math.PI * 100; // SVG daire çevresi (r=100)
                const offset = circumference - (energy / maxEnergy) * circumference;
                energyRingEl.style.strokeDasharray = circumference;
                energyRingEl.style.strokeDashoffset = offset;
            };
            
            // Token sayısını güncelleme
            const updateTokenDisplay = () => {
                tokenCountEl.textContent = tokens;
                leaderboardUserScoreEl.textContent = tokens;
                totalTokensCountEl.textContent = totalEarnedTokens;
            };
            
            // Enerji barını güncelleme
            const updateEnergyBar = () => {
                const percentage = (energy / maxEnergy) * 100;
                energyBarEl.style.width = `${percentage}%`;
                energyTextEl.textContent = `${Math.floor(energy)}/${maxEnergy}`;
                updateEnergyRing();
            };
            
            // Level ilerleme çubuğunu güncelleme
            const updateLevelProgress = () => {
                levelProgressEl.textContent = levelXP;
                const percentage = (levelXP / xpToNextLevel) * 100;
                levelProgressBarEl.style.width = `${percentage}%`;
            };
            
            // Token ekle
            const addTokens = (amount) => {
                tokens += amount;
                totalEarnedTokens += amount;
                updateTokenDisplay();
                
                // Token ekleme animasyonu
                tokenCountEl.classList.add('token-pulse');
                setTimeout(() => {
                    tokenCountEl.classList.remove('token-pulse');
                }, 500);
                
                // XP ekle ve seviye kontrolü
                addXP(amount * 5);
            };
            
            // Aktivite ekleme
            const addActivity = (text, amount, type = 'default') => {
                const li = document.createElement('li');
                li.className = 'py-2 flex justify-between items-center';
                
                let iconClass = 'fas fa-hand-pointer';
                let bgColorClass = 'bg-green-100 dark:bg-green-900';
                let textColorClass = 'text-green-800 dark:text-green-200';
                
                if (type === 'level') {
                    iconClass = 'fas fa-star';
                    bgColorClass = 'bg-blue-100 dark:bg-blue-900';
                    textColorClass = 'text-blue-800 dark:text-blue-200';
                } else if (type === 'quest') {
                    iconClass = 'fas fa-tasks';
                    bgColorClass = 'bg-purple-100 dark:bg-purple-900';
                    textColorClass = 'text-purple-800 dark:text-purple-200';
                } else if (type === 'daily') {
                    iconClass = 'fas fa-calendar-day';
                    bgColorClass = 'bg-yellow-100 dark:bg-yellow-900';
                    textColorClass = 'text-yellow-800 dark:text-yellow-200';
                }
                
                let amountText = '';
                if (amount > 0) {
                    amountText = `+${amount} CQX`;
                } else if (amount < 0) {
                    amountText = `${amount} CQX`;
                } else {
                    amountText = amount;
                }
                
                li.innerHTML = `
                    <div class="flex items-center">
                        <div class="${bgColorClass} ${textColorClass} p-1 rounded">
                            <i class="${iconClass} text-xs"></i>
                        </div>
                        <span class="ml-2 text-sm">${text}</span>
                    </div>
                    <div class="${amount > 0 ? 'text-green-600 dark:text-green-400' : 'text-gray-600 dark:text-gray-400'} text-sm font-medium">
                        ${amountText}
                    </div>
                `;
                
                activityListEl.insertBefore(li, activityListEl.firstChild);
                
                // Maksimum 10 aktivite
                if (activityListEl.children.length > 10) {
                    activityListEl.removeChild(activityListEl.lastChild);
                }
            };
            
            // XP ekle ve seviye atlama kontrolü
            const addXP = (amount) => {
                levelXP += amount;
                
                if (levelXP >= xpToNextLevel) {
                    // Seviye atla
                    levelXP = levelXP - xpToNextLevel;
                    level++;
                    xpToNextLevel = Math.floor(xpToNextLevel * 1.5);
                    userLevelEl.textContent = level;
                    
                    // Seviye bonusu
                    const levelBonus = level * 10;
                    addTokens(levelBonus);
                    
                    // Seviye atladı bildirimi
                    newLevelNumEl.textContent = level;
                    levelUpModalEl.classList.remove('hidden');
                    
                    // Aktivite ekle
                    addActivity('Seviye atladınız', `Lvl ${level}`, 'level');
                }
                
                updateLevelProgress();
            };
            
            // Coin animasyonu oluştur
            const createCoinAnimation = (x, y) => {
                const coin = document.createElement('div');
                coin.className = 'coin';
                coin.style.left = `${x}px`;
                coin.style.top = `${y}px`;
                coin.innerHTML = `
                    <div class="text-xl text-primary font-bold">+1</div>
                `;
                document.body.appendChild(coin);
                
                // Animasyon bitince elementi kaldır
                setTimeout(() => {
                    document.body.removeChild(coin);
                }, 1000);
            };
            
            // Toast mesajı göster
            const showToast = (message, type = 'success') => {
                toastMessageEl.textContent = message;
                toastEl.className = toastEl.className.replace(/bg-\w+-500/g, '');
                
                if (type === 'success') {
                    toastEl.classList.add('bg-green-500');
                } else if (type === 'error') {
                    toastEl.classList.add('bg-red-500');
                } else if (type === 'info') {
                    toastEl.classList.add('bg-blue-500');
                }
                
                toastEl.classList.remove('translate-x-full');
                
                setTimeout(() => {
                    toastEl.classList.add('translate-x-full');
                }, 3000);
            };
            
            // Tab değiştirme
            const switchTab = (tabId) => {
                tabContents.forEach(tab => {
                    tab.classList.remove('active');
                });
                
                tabBtns.forEach(btn => {
                    btn.classList.remove('text-primary', 'border-t-2', 'border-primary');
                    btn.classList.add('text-gray-500', 'dark:text-gray-400');
                });
                
                document.getElementById(tabId).classList.add('active');
                const activeBtn = document.querySelector(`.tab-btn[data-tab="${tabId}"]`);
                activeBtn.classList.remove('text-gray-500', 'dark:text-gray-400');
                activeBtn.classList.add('text-primary', 'border-t-2', 'border-primary');
            };
            
            // Event Listeners
            
            // Dokunma alanı
            tapAreaEl.addEventListener('click', (e) => {
                if (energy >= 5) {
                    // Enerji tüket
                    energy -= 5;
                    updateEnergyBar();
                    
                    // Token kazandır
                    addTokens(1);
                    
                    // Aktivite ekle
                    addActivity('Dokunarak kazanıldı', 1);
                    
                    // Animasyon
                    createCoinAnimation(e.clientX, e.clientY);
                    tapAreaEl.classList.add('pulse-animation');
                    setTimeout(() => {
                        tapAreaEl.classList.remove('pulse-animation');
                    }, 300);
                    
                    // Tıklama sayısını güncelle
                    totalTaps++;
                    totalTapsCountEl.textContent = totalTaps;
                } else {
                    // Enerji yetersiz
                    showToast('Yeterli enerji yok! Enerjinin dolmasını bekle.', 'error');
                }
            });
            
            // Tab butonları
            tabBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    const tabId = btn.getAttribute('data-tab');
                    switchTab(tabId);
                });
            });
            
            // Günlük bonus butonu
            claimDailyBtn.addEventListener('click', () => {
                showToast('Günlük bonusunu zaten aldın!', 'info');
            });
            
            // Seviye atlama modal kapatma
            closeLevelUpBtn.addEventListener('click', () => {
                levelUpModalEl.classList.add('hidden');
            });
            
            // Rank butonları
            rankBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    rankBtns.forEach(b => {
                        b.classList.remove('bg-primary', 'text-white');
                        b.classList.add('bg-gray-200', 'dark:bg-gray-700');
                    });
                    btn.classList.remove('bg-gray-200', 'dark:bg-gray-700');
                    btn.classList.add('bg-primary', 'text-white');
                });
            });
            
            // Davet kodu kopyalama
            copyInviteBtn.addEventListener('click', () => {
                showToast('Davet kodu kopyalandı!', 'success');
            });
            
            // Görev butonları
            questBtns.forEach(btn => {
                btn.addEventListener('click', () => {
                    showToast('Görev tamamlandı!', 'success');
                    addTokens(20);
                    addActivity('Görev tamamlandı', 20, 'quest');
                });
            });
            
            // Enerji yenileme zamanlayıcısı
            setInterval(() => {
                if (energy < maxEnergy) {
                    energy += energyRegenRate;
                    if (energy > maxEnergy) energy = maxEnergy;
                    updateEnergyBar();
                }
            }, 10000); // 10 saniyede bir enerji yenilenir
            
            // İlk yükleme
            updateTokenDisplay();
            updateEnergyBar();
            updateLevelProgress();
        });
    </script>
</body>
</html>
