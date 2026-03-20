# inventory-system
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Qunfeng Service | Система мониторинга запасных частей</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Inter', sans-serif; background: #0f172a; color: #e2e8f0; }
        .glass { background: rgba(30, 41, 59, 0.7); backdrop-filter: blur(12px); border: 1px solid rgba(255,255,255,0.1); }
        .glass-hover:hover { background: rgba(51, 65, 85, 0.8); transform: translateY(-2px); transition: all 0.3s ease; }
        .gradient-text { background: linear-gradient(135deg, #60a5fa, #a78bfa); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .status-critical { animation: pulse-red 2s infinite; }
        @keyframes pulse-red { 0%, 100% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.4); } 50% { box-shadow: 0 0 0 10px rgba(239, 68, 68, 0); } }
        .sidebar-item.active { background: linear-gradient(90deg, rgba(96, 165, 250, 0.2), transparent); border-left: 3px solid #60a5fa; }
        .stock-bar { transition: width 0.5s ease; }
        .modal-enter { animation: modalIn 0.3s ease; }
        @keyframes modalIn { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
        .table-row:hover { background: rgba(51, 65, 85, 0.5); }
        .badge { font-size: 0.75rem; padding: 0.25rem 0.75rem; border-radius: 9999px; font-weight: 600; }
        .badge-success { background: rgba(34, 197, 94, 0.2); color: #4ade80; }
        .badge-warning { background: rgba(234, 179, 8, 0.2); color: #facc15; }
        .badge-danger { background: rgba(239, 68, 68, 0.2); color: #f87171; }
        .badge-info { background: rgba(96, 165, 250, 0.2); color: #60a5fa; }
    </style>
</head>
<body class="antialiased overflow-hidden">

    <!-- App Container -->
    <div class="flex h-screen">
        
        <!-- Sidebar -->
        <aside class="w-64 glass flex flex-col border-r border-slate-700/50">
            <div class="p-6 border-b border-slate-700/50">
                <h1 class="text-2xl font-bold gradient-text">QUNFENG</h1>
                <p class="text-xs text-slate-400 mt-1">Service Department</p>
            </div>
            
            <nav class="flex-1 py-6 space-y-1">
                <button onclick="switchTab('dashboard')" id="nav-dashboard" class="sidebar-item active w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-chart-line w-5"></i> Дашборд
                </button>
                <button onclick="switchTab('inventory')" id="nav-inventory" class="sidebar-item w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-boxes w-5"></i> Запасные части
                </button>
                <button onclick="switchTab('equipment')" id="nav-equipment" class="sidebar-item w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-cogs w-5"></i> Оборудование
                </button>
                <button onclick="switchTab('suppliers')" id="nav-suppliers" class="sidebar-item w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-ship w-5"></i> Поставщики
                </button>
                <button onclick="switchTab('clients')" id="nav-clients" class="sidebar-item w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-users w-5"></i> Клиенты
                </button>
                <button onclick="switchTab('orders')" id="nav-orders" class="sidebar-item w-full text-left px-6 py-3 text-sm font-medium text-slate-300 hover:text-white transition-colors flex items-center gap-3">
                    <i class="fas fa-clipboard-list w-5"></i> Заказы
                </button>
            </nav>
            
            <div class="p-4 border-t border-slate-700/50">
                <div class="glass rounded-lg p-4">
                    <div class="flex items-center gap-3 mb-2">
                        <div class="w-2 h-2 rounded-full bg-green-500 animate-pulse"></div>
                        <span class="text-xs text-slate-400">Система активна</span>
                    </div>
                    <div class="text-xs text-slate-500">Обновлено: <span id="last-update">08:25</span></div>
                </div>
            </div>
        </aside>

        <!-- Main Content -->
        <main class="flex-1 overflow-hidden flex flex-col bg-slate-900">
            
            <!-- Header -->
            <header class="h-16 glass border-b border-slate-700/50 flex items-center justify-between px-8">
                <div class="flex items-center gap-4">
                    <h2 id="page-title" class="text-xl font-semibold text-white">Обзор системы</h2>
                    <span class="px-3 py-1 rounded-full bg-blue-500/20 text-blue-400 text-xs border border-blue-500/30">Tech Service</span>
                </div>
                <div class="flex items-center gap-4">
                    <button onclick="openModal('order-modal')" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-lg text-sm font-medium transition-colors flex items-center gap-2">
                        <i class="fas fa-plus"></i> Новый заказ
                    </button>
                    <div class="w-8 h-8 rounded-full bg-gradient-to-br from-blue-500 to-purple-600 flex items-center justify-center text-xs font-bold">
                        QS
                    </div>
                </div>
            </header>

            <!-- Content Scroll Area -->
            <div class="flex-1 overflow-y-auto p-8" id="content-area">
                
                <!-- Dashboard View -->
                <div id="view-dashboard" class="space-y-6">
                    <!-- Stats Grid -->
                    <div class="grid grid-cols-1 md:grid-cols-4 gap-6">
                        <div class="glass rounded-xl p-6 glass-hover">
                            <div class="flex justify-between items-start mb-4">
                                <div>
                                    <p class="text-slate-400 text-sm">Всего позиций</p>
                                    <h3 class="text-3xl font-bold text-white mt-1" id="total-items">0</h3>
                                </div>
                                <div class="w-10 h-10 rounded-lg bg-blue-500/20 flex items-center justify-center text-blue-400">
                                    <i class="fas fa-cube"></i>
                                </div>
                            </div>
                            <div class="text-xs text-green-400 flex items-center gap-1">
                                <i class="fas fa-arrow-up"></i> +12% к прошлому месяцу
                            </div>
                        </div>

                        <div class="glass rounded-xl p-6 glass-hover">
                            <div class="flex justify-between items-start mb-4">
                                <div>
                                    <p class="text-slate-400 text-sm">Критический запас</p>
                                    <h3 class="text-3xl font-bold text-red-400 mt-1" id="critical-count">0</h3>
                                </div>
                                <div class="w-10 h-10 rounded-lg bg-red-500/20 flex items-center justify-center text-red-400 status-critical">
                                    <i class="fas fa-exclamation-triangle"></i>
                                </div>
                            </div>
                            <div class="text-xs text-slate-400">Требуют срочной закупки</div>
                        </div>

                        <div class="glass rounded-xl p-6 glass-hover">
                            <div class="flex justify-between items-start mb-4">
                                <div>
                                    <p class="text-slate-400 text-sm">В пути из Китая</p>
                                    <h3 class="text-3xl font-bold text-yellow-400 mt-1" id="transit-count">0</h3>
                                </div>
                                <div class="w-10 h-10 rounded-lg bg-yellow-500/20 flex items-center justify-center text-yellow-400">
                                    <i class="fas fa-ship"></i>
                                </div>
                            </div>
                            <div class="text-xs text-slate-400">Средний срок: 45 дней</div>
                        </div>

                        <div class="glass rounded-xl p-6 glass-hover">
                            <div class="flex justify-between items-start mb-4">
                                <div>
                                    <p class="text-slate-400 text-sm">Продажи (мес)</p>
                                    <h3 class="text-3xl font-bold text-green-400 mt-1">₽2.4M</h3>
                                </div>
                                <div class="w-10 h-10 rounded-lg bg-green-500/20 flex items-center justify-center text-green-400">
                                    <i class="fas fa-ruble-sign"></i>
                                </div>
                            </div>
                            <div class="text-xs text-green-400 flex items-center gap-1">
                                <i class="fas fa-arrow-up"></i> +8% к прошлому месяцу
                            </div>
                        </div>
                    </div>

                    <!-- Charts & Alerts -->
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                        <div class="lg:col-span-2 glass rounded-xl p-6">
                            <h3 class="text-lg font-semibold mb-4">Динамика запасов</h3>
                            <canvas id="stockChart" height="200"></canvas>
                        </div>
                        
                        <div class="glass rounded-xl p-6">
                            <h3 class="text-lg font-semibold mb-4 flex items-center gap-2">
                                <i class="fas fa-bell text-yellow-400"></i> Уведомления
                            </h3>
                            <div class="space-y-3" id="alerts-list">
                                <!-- Alerts injected here -->
                            </div>
                        </div>
                    </div>

                    <!-- Quick Access -->
                    <div class="glass rounded-xl p-6">
                        <h3 class="text-lg font-semibold mb-4">Быстрый доступ</h3>
                        <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                            <button onclick="switchTab('inventory'); filterByEquipment('vibro')" class="p-4 rounded-lg bg-slate-800/50 hover:bg-slate-700/50 border border-slate-700 transition-all text-left group">
                                <i class="fas fa-industry text-2xl text-blue-400 mb-2 group-hover:scale-110 transition-transform"></i>
                                <div class="font-medium">Вибропрессы</div>
                                <div class="text-xs text-slate-400">Запчасти и узлы</div>
                            </button>
                            <button onclick="switchTab('inventory'); filterByEquipment('mixer')" class="p-4 rounded-lg bg-slate-800/50 hover:bg-slate-700/50 border border-slate-700 transition-all text-left group">
                                <i class="fas fa-blender text-2xl text-purple-400 mb-2 group-hover:scale-110 transition-transform"></i>
                                <div class="font-medium">Смесители</div>
                                <div class="text-xs text-slate-400">Запчасти и узлы</div>
                            </button>
                            <button onclick="openModal('supplier-modal')" class="p-4 rounded-lg bg-slate-800/50 hover:bg-slate-700/50 border border-slate-700 transition-all text-left group">
                                <i class="fas fa-plus-circle text-2xl text-green-400 mb-2 group-hover:scale-110 transition-transform"></i>
                                <div class="font-medium">Добавить поставщика</div>
                                <div class="text-xs text-slate-400">Китай/Европа</div>
                            </button>
                            <button onclick="generateReport()" class="p-4 rounded-lg bg-slate-800/50 hover:bg-slate-700/50 border border-slate-700 transition-all text-left group">
                                <i class="fas fa-file-export text-2xl text-orange-400 mb-2 group-hover:scale-110 transition-transform"></i>
                                <div class="font-medium">Отчёт по запасам</div>
                                <div class="text-xs text-slate-400">Excel/PDF</div>
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Inventory View -->
                <div id="view-inventory" class="hidden space-y-6">
                    <div class="flex justify-between items-center">
                        <div class="flex gap-2">
                            <button onclick="filterInventory('all')" class="px-4 py-2 rounded-lg bg-blue-600 text-white text-sm">Все</button>
                            <button onclick="filterInventory('vibro')" class="px-4 py-2 rounded-lg bg-slate-700 text-slate-300 hover:bg-slate-600 text-sm">Вибропрессы</button>
                            <button onclick="filterInventory('mixer')" class="px-4 py-2 rounded-lg bg-slate-700 text-slate-300 hover:bg-slate-600 text-sm">Смесители</button>
                            <button onclick="filterInventory('critical')" class="px-4 py-2 rounded-lg bg-red-900/30 text-red-400 hover:bg-red-900/50 text-sm border border-red-800">
                                <i class="fas fa-exclamation-circle mr-1"></i> Критические
                            </button>
                        </div>
                        <div class="flex gap-2">
                            <input type="text" id="search-parts" placeholder="Поиск по артикулу или названию..." 
                                   class="px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white text-sm w-64 focus:outline-none focus:border-blue-500">
                            <button onclick="openModal('part-modal')" class="px-4 py-2 bg-green-600 hover:bg-green-500 text-white rounded-lg text-sm">
                                <i class="fas fa-plus mr-1"></i> Добавить запчасть
                            </button>
                        </div>
                    </div>

                    <div class="glass rounded-xl overflow-hidden">
                        <table class="w-full text-left text-sm">
                            <thead class="bg-slate-800/50 text-slate-400">
                                <tr>
                                    <th class="px-6 py-4 font-medium">Артикул</th>
                                    <th class="px-6 py-4 font-medium">Наименование</th>
                                    <th class="px-6 py-4 font-medium">Оборудование</th>
                                    <th class="px-6 py-4 font-medium">Склад</th>
                                    <th class="px-6 py-4 font-medium">Мин. запас</th>
                                    <th class="px-6 py-4 font-medium">Статус</th>
                                    <th class="px-6 py-4 font-medium">Действия</th>
                                </tr>
                            </thead>
                            <tbody id="inventory-table" class="divide-y divide-slate-700/50">
                                <!-- Rows injected here -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Equipment View -->
                <div id="view-equipment" class="hidden space-y-6">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="glass rounded-xl p-6">
                            <div class="flex items-center justify-between mb-6">
                                <h3 class="text-xl font-bold text-blue-400">Вибропрессы Qunfeng</h3>
                                <button onclick="openModal('equipment-modal')" class="text-sm text-blue-400 hover:text-blue-300">+ Добавить модель</button>
                            </div>
                            <div class="space-y-4" id="vibro-list">
                                <!-- Equipment cards -->
                            </div>
                        </div>
                        
                        <div class="glass rounded-xl p-6">
                            <div class="flex items-center justify-between mb-6">
                                <h3 class="text-xl font-bold text-purple-400">Смесители Qunfeng</h3>
                                <button onclick="openModal('equipment-modal')" class="text-sm text-purple-400 hover:text-purple-300">+ Добавить модель</button>
                            </div>
                            <div class="space-y-4" id="mixer-list">
                                <!-- Equipment cards -->
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Suppliers View -->
                <div id="view-suppliers" class="hidden space-y-6">
                    <div class="flex justify-between items-center mb-6">
                        <h3 class="text-lg font-semibold">Поставщики из Китая и Европы</h3>
                        <button onclick="openModal('supplier-modal')" class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-lg text-sm">
                            <i class="fas fa-plus mr-1"></i> Добавить поставщика
                        </button>
                    </div>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6" id="suppliers-grid">
                        <!-- Supplier cards injected here -->
                    </div>
                </div>

                <!-- Clients View -->
                <div id="view-clients" class="hidden space-y-6">
                    <div class="glass rounded-xl p-6">
                        <div class="flex justify-between items-center mb-6">
                            <h3 class="text-lg font-semibold">Клиенты и история продаж</h3>
                            <button onclick="openModal('client-modal')" class="px-4 py-2 bg-green-600 hover:bg-green-500 text-white rounded-lg text-sm">
                                <i class="fas fa-user-plus mr-1"></i> Новый клиент
                            </button>
                        </div>
                        <table class="w-full text-left text-sm">
                            <thead class="bg-slate-800/50 text-slate-400">
                                <tr>
                                    <th class="px-6 py-4">Компания</th>
                                    <th class="px-6 py-4">Контакт</th>
                                    <th class="px-6 py-4">Оборудование</th>
                                    <th class="px-6 py-4">Последняя покупка</th>
                                    <th class="px-6 py-4">Общий объем</th>
                                </tr>
                            </thead>
                            <tbody id="clients-table" class="divide-y divide-slate-700/50">
                                <!-- Client rows -->
                            </tbody>
                        </table>
                    </div>
                </div>

                <!-- Orders View -->
                <div id="view-orders" class="hidden space-y-6">
                    <div class="glass rounded-xl p-6">
                        <h3 class="text-lg font-semibold mb-6">Заказы на поставку (в пути из Китая)</h3>
                        <div class="space-y-4" id="orders-list">
                            <!-- Order tracking -->
                        </div>
                    </div>
                </div>

            </div>
        </main>
    </div>

    <!-- Modal: Add Part -->
    <div id="part-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden items-center justify-center z-50">
        <div class="glass rounded-xl p-6 w-full max-w-2xl modal-enter">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-xl font-bold">Добавить запасную часть</h3>
                <button onclick="closeModal('part-modal')" class="text-slate-400 hover:text-white"><i class="fas fa-times text-xl"></i></button>
            </div>
            <form onsubmit="savePart(event)" class="grid grid-cols-2 gap-4">
                <div class="col-span-2">
                    <label class="block text-sm text-slate-400 mb-1">Наименование</label>
                    <input type="text" name="name" required class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Артикул (SKU)</label>
                    <input type="text" name="sku" required class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Тип оборудования</label>
                    <select name="type" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                        <option value="vibro">Вибропресс</option>
                        <option value="mixer">Смеситель</option>
                    </select>
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Модель оборудования</label>
                    <input type="text" name="model" placeholder="например: QF800" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Категория</label>
                    <select name="category" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                        <option>Гидравлика</option>
                        <option>Электрика</option>
                        <option>Механика</option>
                        <option>Износостойкие детали</option>
                        <option>Узлы и агрегаты</option>
                    </select>
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Количество на складе</label>
                    <input type="number" name="stock" min="0" value="0" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Минимальный запас</label>
                    <input type="number" name="min_stock" min="0" value="5" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Поставщик</label>
                    <select name="supplier" id="supplier-select" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                        <!-- Options injected -->
                    </select>
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Срок поставки (дней)</label>
                    <input type="number" name="lead_time" value="45" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div class="col-span-2 flex justify-end gap-3 mt-4">
                    <button type="button" onclick="closeModal('part-modal')" class="px-4 py-2 rounded-lg bg-slate-700 text-white hover:bg-slate-600">Отмена</button>
                    <button type="submit" class="px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-500">Сохранить</button>
                </div>
            </form>
        </div>
    </div>

    <!-- Modal: New Order -->
    <div id="order-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm hidden items-center justify-center z-50">
        <div class="glass rounded-xl p-6 w-full max-w-lg modal-enter">
            <div class="flex justify-between items-center mb-6">
                <h3 class="text-xl font-bold">Создать заказ клиента</h3>
                <button onclick="closeModal('order-modal')" class="text-slate-400 hover:text-white"><i class="fas fa-times text-xl"></i></button>
            </div>
            <form onsubmit="saveOrder(event)" class="space-y-4">
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Клиент</label>
                    <select name="client" id="client-select" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                        <!-- Options injected -->
                    </select>
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Запасная часть</label>
                    <select name="part" id="part-select" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                        <!-- Options injected -->
                    </select>
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Количество</label>
                    <input type="number" name="quantity" min="1" value="1" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm text-slate-400 mb-1">Примечания</label>
                    <textarea name="notes" rows="3" class="w-full px-4 py-2 rounded-lg bg-slate-800 border border-slate-700 text-white focus:border-blue-500 focus:outline-none"></textarea>
                </div>
                <div class="flex justify-end gap-3 mt-6">
                    <button type="button" onclick="closeModal('order-modal')" class="px-4 py-2 rounded-lg bg-slate-700 text-white hover:bg-slate-600">Отмена</button>
                    <button type="submit" class="px-4 py-2 rounded-lg bg-green-600 text-white hover:bg-green-500">Оформить продажу</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        // Data Store
        const store = {
            parts: [
                { id: 1, sku: 'QF-HC-001', name: 'Гидроцилиндр пресса 200т', type: 'vibro', model: 'QF800', category: 'Гидравлика', stock: 2, min_stock: 3, supplier: 'Qunfeng China', lead_time: 60, price: 450000 },
                { id: 2, sku: 'QF-MX-002', name: 'Вал смесителя JS1500', type: 'mixer', model: 'JS1500', category: 'Механика', stock: 1, min_stock: 2, supplier: 'Qunfeng China', lead_time: 45, price: 120000 },
                { id: 3, sku: 'QF-EL-003', name: 'Контроллер PLC S7-1200', type: 'vibro', model: 'Universal', category: 'Электрика', stock: 5, min_stock: 2, supplier: 'Siemens DE', lead_time: 14, price: 85000 },
                { id: 4, sku: 'QF-WP-004', name: 'Поддон формовочный 1200x800', type: 'vibro', model: 'QF600', category: 'Износостойкие детали', stock: 12, min_stock: 10, supplier: 'Qunfeng China', lead_time: 30, price: 15000 },
                { id: 5, sku: 'QF-HC-005', name: 'Гидронасос Rexroth A10VSO', type: 'vibro', model: 'QF1200', category: 'Гидравлика', stock: 0, min_stock: 2, supplier: 'Rexroth DE', lead_time: 21, price: 95000 },
                { id: 6, sku: 'QF-MX-006', name: 'Лопасть смесителя комплект', type: 'mixer', model: 'JS1000', category: 'Износостойкие детали', stock: 8, min_stock: 5, supplier: 'Qunfeng China', lead_time: 40, price: 22000 },
                { id: 7, sku: 'QF-EL-007', name: 'Датчик давления 0-400 bar', type: 'vibro', model: 'Universal', category: 'Электрика', stock: 3, min_stock: 5, supplier: 'Wika DE', lead_time: 10, price: 12000 },
                { id: 8, sku: 'QF-MC-008', name: 'Вибромотор 3.7 кВт', type: 'vibro', model: 'QF400', category: 'Узлы и агрегаты', stock: 4, min_stock: 3, supplier: 'Italvibras IT', lead_time: 18, price: 65000 }
            ],
            suppliers: [
                { id: 1, name: 'Qunfeng Machinery China', country: 'CN', city: 'Цюаньчжоу', lead_time: 45, type: 'Основной', contact: 'Li Wei' },
                { id: 2, name: 'Siemens AG', country: 'DE', city: 'Мюнхен', lead_time: 14, type: 'Электрика', contact: 'Hans Mueller' },
                { id: 3, name: 'Bosch Rexroth', country: 'DE', city: 'Лор-ам-Майн', lead_time: 21, type: 'Гидравлика', contact: 'Klaus Schmidt' },
                { id: 4, name: 'Italvibras', country: 'IT', city: 'Милан', lead_time: 18, type: 'Вибраторы', contact: 'Marco Rossi' }
            ],
            clients: [
                { id: 1, name: 'СтройБетон ООО', contact: 'Иванов А.П.', phone: '+7 (495) 123-45-67', equipment: 'QF800, JS1500', last_order: '2026-03-15', total: 1250000 },
                { id: 2, name: 'БетонПром Групп', contact: 'Петров С.В.', phone: '+7 (812) 987-65-43', equipment: 'QF1200', last_order: '2026-03-10', total: 2340000 },
                { id: 3, name: 'ЖБИ-Комплект', contact: 'Сидоров М.А.', phone: '+7 (383) 456-78-90', equipment: 'QF600, JS1000', last_order: '2026-02-28', total: 890000 }
            ],
            orders: [
                { id: 'ORD-2026-001', part_id: 5, quantity: 3, status: 'transit', date: '2026-02-15', eta: '2026-04-01', supplier: 'Rexroth DE' },
                { id: 'ORD-2026-002', part_id: 1, quantity: 5, status: 'transit', date: '2026-02-20', eta: '2026-04-15', supplier: 'Qunfeng China' },
                { id: 'ORD-2026-003', part_id: 7, quantity: 10, status: 'processing', date: '2026-03-18', eta: '2026-03-28', supplier: 'Wika DE' }
            ]
        };

        // Initialize
        document.addEventListener('DOMContentLoaded', () => {
            updateDashboard();
            renderInventory();
            renderSuppliers();
            renderClients();
            renderOrders();
            renderEquipment();
            initChart();
            populateSelects();
        });

        // Navigation
        function switchTab(tab) {
            // Hide all views
            document.querySelectorAll('[id^="view-"]').forEach(el => el.classList.add('hidden'));
            document.getElementById(`view-${tab}`).classList.remove('hidden');
            
            // Update sidebar
            document.querySelectorAll('.sidebar-item').forEach(el => el.classList.remove('active'));
            document.getElementById(`nav-${tab}`).classList.add('active');
            
            // Update title
            const titles = {
                dashboard: 'Обзор системы',
                inventory: 'Управление запасными частями',
                equipment: 'Оборудование',
                suppliers: 'Поставщики',
                clients: 'Клиенты',
                orders: 'Заказы и поставки'
            };
            document.getElementById('page-title').textContent = titles[tab];
        }

        // Dashboard Updates
        function updateDashboard() {
            document.getElementById('total-items').textContent = store.parts.length;
            
            const critical = store.parts.filter(p => p.stock <= p.min_stock);
            document.getElementById('critical-count').textContent = critical.length;
            
            const transit = store.orders.filter(o => o.status === 'transit').length;
            document.getElementById('transit-count').textContent = transit;
            
            // Alerts
            const alertsList = document.getElementById('alerts-list');
            alertsList.innerHTML = '';
            
            critical.slice(0, 5).forEach(part => {
                const div = document.createElement('div');
                div.className = 'flex items-center gap-3 p-3 rounded-lg bg-red-900/20 border border-red-800/50';
                div.innerHTML = `
                    <i class="fas fa-exclamation-triangle text-red-400"></i>
                    <div class="flex-1">
                        <div class="text-sm font-medium text-red-200">${part.name}</div>
                        <div class="text-xs text-red-400">Остаток: ${part.stock} (мин: ${part.min_stock})</div>
                    </div>
                    <button onclick="openOrderModal(${part.id})" class="px-3 py-1 rounded bg-red-600 hover:bg-red-500 text-white text-xs">Заказать</button>
                `;
                alertsList.appendChild(div);
            });
        }

        // Inventory Management
        function renderInventory(filter = 'all') {
            const tbody = document.getElementById('inventory-table');
            tbody.innerHTML = '';
            
            let parts = store.parts;
            if (filter === 'vibro') parts = parts.filter(p => p.type === 'vibro');
            if (filter === 'mixer') parts = parts.filter(p => p.type === 'mixer');
            if (filter === 'critical') parts = parts.filter(p => p.stock <= p.min_stock);
            
            parts.forEach(part => {
                const status = part.stock === 0 ? 'danger' : part.stock <= part.min_stock ? 'warning' : 'success';
                const statusText = part.stock === 0 ? 'Нет в наличии' : part.stock <= part.min_stock ? 'Низкий запас' : 'В наличии';
                
                const tr = document.createElement('tr');
                tr.className = 'table-row border-b border-slate-700/50';
                tr.innerHTML = `
                    <td class="px-6 py-4 font-mono text-blue-400">${part.sku}</td>
                    <td class="px-6 py-4">
                        <div class="font-medium">${part.name}</div>
                        <div class="text-xs text-slate-500">${part.category}</div>
                    </td>
                    <td class="px-6 py-4">
                        <span class="px-2 py-1 rounded bg-slate-700 text-xs">${part.model}</span>
                    </td>
                    <td class="px-6 py-4">
                        <div class="flex items-center gap-2">
                            <div class="w-16 h-2 bg-slate-700 rounded-full overflow-hidden">
                                <div class="h-full ${part.stock <= part.min_stock ? 'bg-red-500' : 'bg-green-500'} stock-bar" 
                                     style="width: ${Math.min((part.stock / (part.min_stock * 2)) * 100, 100)}%"></div>
                            </div>
                            <span class="text-sm ${part.stock <= part.min_stock ? 'text-red-400 font-bold' : 'text-slate-300'}">${part.stock}</span>
                        </div>
                    </td>
                    <td class="px-6 py-4 text-slate-400">${part.min_stock}</td>
                    <td class="px-6 py-4"><span class="badge badge-${status}">${statusText}</span></td>
                    <td class="px-6 py-4">
                        <button onclick="editPart(${part.id})" class="text-blue-400 hover:text-blue-300 mr-3"><i class="fas fa-edit"></i></button>
                        <button onclick="deletePart(${part.id})" class="text-red-400 hover:text-red-300"><i class="fas fa-trash"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        function filterInventory(type) {
            renderInventory(type);
        }

        function filterByEquipment(type) {
            switchTab('inventory');
            renderInventory(type);
        }

        // Suppliers
        function renderSuppliers() {
            const grid = document.getElementById('suppliers-grid');
            grid.innerHTML = '';
            
            store.suppliers.forEach(sup => {
                const card = document.createElement('div');
                card.className = 'glass rounded-xl p-6 glass-hover';
                card.innerHTML = `
                    <div class="flex justify-between items-start mb-4">
                        <div class="w-12 h-12 rounded-lg bg-slate-700 flex items-center justify-center text-2xl">
                            ${sup.country === 'CN' ? '🇨🇳' : sup.country === 'DE' ? '🇩🇪' : '🇮🇹'}
                        </div>
                        <span class="badge badge-info">${sup.type}</span>
                    </div>
                    <h4 class="font-bold text-lg mb-1">${sup.name}</h4>
                    <p class="text-sm text-slate-400 mb-4">${sup.city}, ${sup.country}</p>
                    <div class="space-y-2 text-sm">
                        <div class="flex justify-between"><span class="text-slate-500">Контакт:</span> <span>${sup.contact}</span></div>
                        <div class="flex justify-between"><span class="text-slate-500">Срок поставки:</span> <span class="text-yellow-400">${sup.lead_time} дней</span></div>
                    </div>
                    <div class="mt-4 pt-4 border-t border-slate-700 flex gap-2">
                        <button class="flex-1 py-2 rounded bg-blue-600/20 text-blue-400 hover:bg-blue-600/30 text-sm">Заказать</button>
                        <button class="flex-1 py-2 rounded bg-slate-700 text-slate-300 hover:bg-slate-600 text-sm">История</button>
                    </div>
                `;
                grid.appendChild(card);
            });
        }

        // Clients
        function renderClients() {
            const tbody = document.getElementById('clients-table');
            tbody.innerHTML = '';
            
            store.clients.forEach(client => {
                const tr = document.createElement('tr');
                tr.className = 'table-row';
                tr.innerHTML = `
                    <td class="px-6 py-4">
                        <div class="font-medium">${client.name}</div>
                    </td>
                    <td class="px-6 py-4">
                        <div>${client.contact}</div>
                        <div class="text-xs text-slate-500">${client.phone}</div>
                    </td>
                    <td class="px-6 py-4 text-slate-400">${client.equipment}</td>
                    <td class="px-6 py-4 text-slate-400">${client.last_order}</td>
                    <td class="px-6 py-4 font-medium text-green-400">₽${client.total.toLocaleString()}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        // Orders
        function renderOrders() {
            const container = document.getElementById('orders-list');
            container.innerHTML = '';
            
            store.orders.forEach(order => {
                const part = store.parts.find(p => p.id === order.part_id);
                const progress = order.status === 'transit' ? 60 : order.status === 'processing' ? 30 : 100;
                
                const div = document.createElement('div');
                div.className = 'glass rounded-lg p-4 border-l-4 border-blue-500';
                div.innerHTML = `
                    <div class="flex justify-between items-start mb-3">
                        <div>
                            <div class="font-bold text-lg">${order.id}</div>
                            <div class="text-sm text-slate-400">${part ? part.name : 'Unknown'}</div>
                        </div>
                        <span class="badge badge-${order.status === 'transit' ? 'warning' : 'info'}">
                            ${order.status === 'transit' ? 'В пути' : 'Обработка'}
                        </span>
                    </div>
                    <div class="flex justify-between text-sm mb-2">
                        <span class="text-slate-400">Количество: <span class="text-white">${order.quantity}</span></span>
                        <span class="text-slate-400">Поставщик: <span class="text-white">${order.supplier}</span></span>
                    </div>
                    <div class="space-y-1">
                        <div class="flex justify-between text-xs text-slate-500">
                            <span>Отправка: ${order.date}</span>
                            <span>Прибытие: ${order.eta}</span>
                        </div>
                        <div class="w-full h-2 bg-slate-700 rounded-full overflow-hidden">
                            <div class="h-full bg-blue-500 stock-bar" style="width: ${progress}%"></div>
                        </div>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        // Equipment
        function renderEquipment() {
            const vibroList = document.getElementById('vibro-list');
            const mixerList = document.getElementById('mixer-list');
            
            const vibros = ['QF400', 'QF600', 'QF800', 'QF1000', 'QF1200', 'QF2000'];
            const mixers = ['JS500', 'JS750', 'JS1000', 'JS1500', 'JS2000', 'JS3000'];
            
            vibroList.innerHTML = vibros.map(model => `
                <div class="flex items-center justify-between p-4 rounded-lg bg-slate-800/50 border border-slate-700 hover:border-blue-500/50 transition-colors cursor-pointer" onclick="filterByModel('${model}')">
                    <div class="flex items-center gap-4">
                        <div class="w-10 h-10 rounded bg-blue-500/20 flex items-center justify-center text-blue-400">
                            <i class="fas fa-industry"></i>
                        </div>
                        <div>
                            <div class="font-medium">Вибропресс ${model}</div>
                            <div class="text-xs text-slate-500">${store.parts.filter(p => p.model === model).length} позиций</div>
                        </div>
                    </div>
                    <i class="fas fa-chevron-right text-slate-600"></i>
                </div>
            `).join('');
            
            mixerList.innerHTML = mixers.map(model => `
                <div class="flex items-center justify-between p-4 rounded-lg bg-slate-800/50 border border-slate-700 hover:border-purple-500/50 transition-colors cursor-pointer" onclick="filterByModel('${model}')">
                    <div class="flex items-center gap-4">
                        <div class="w-10 h-10 rounded bg-purple-500/20 flex items-center justify-center text-purple-400">
                            <i class="fas fa-blender"></i>
                        </div>
                        <div>
                            <div class="font-medium">Смеситель ${model}</div>
                            <div class="text-xs text-slate-500">${store.parts.filter(p => p.model === model).length} позиций</div>
                        </div>
                    </div>
                    <i class="fas fa-chevron-right text-slate-600"></i>
                </div>
            `).join('');
        }

        function filterByModel(model) {
            switchTab('inventory');
            const tbody = document.getElementById('inventory-table');
            tbody.innerHTML = '';
            
            const parts = store.parts.filter(p => p.model === model);
            parts.forEach(part => {
                // Same render logic as renderInventory
                const status = part.stock === 0 ? 'danger' : part.stock <= part.min_stock ? 'warning' : 'success';
                const tr = document.createElement('tr');
                tr.className = 'table-row border-b border-slate-700/50';
                tr.innerHTML = `
                    <td class="px-6 py-4 font-mono text-blue-400">${part.sku}</td>
                    <td class="px-6 py-4"><div class="font-medium">${part.name}</div></td>
                    <td class="px-6 py-4"><span class="px-2 py-1 rounded bg-slate-700 text-xs">${part.model}</span></td>
                    <td class="px-6 py-4">${part.stock}</td>
                    <td class="px-6 py-4 text-slate-400">${part.min_stock}</td>
                    <td class="px-6 py-4"><span class="badge badge-${status}">${part.stock === 0 ? 'Нет' : part.stock <= part.min_stock ? 'Низкий' : 'OK'}</span></td>
                    <td class="px-6 py-4">
                        <button onclick="editPart(${part.id})" class="text-blue-400 hover:text-blue-300 mr-3"><i class="fas fa-edit"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        // Modal Functions
        function openModal(id) {
            document.getElementById(id).classList.remove('hidden');
            document.getElementById(id).classList.add('flex');
        }

        function closeModal(id) {
            document.getElementById(id).classList.add('hidden');
            document.getElementById(id).classList.remove('flex');
        }

        function populateSelects() {
            const supSelect = document.getElementById('supplier-select');
            supSelect.innerHTML = store.suppliers.map(s => `<option value="${s.name}">${s.name}</option>`).join('');
            
            const clientSelect = document.getElementById('client-select');
            clientSelect.innerHTML = store.clients.map(c => `<option value="${c.id}">${c.name}</option>`).join('');
            
            const partSelect = document.getElementById('part-select');
            partSelect.innerHTML = store.parts.map(p => `<option value="${p.id}">${p.sku} - ${p.name} (в наличии: ${p.stock})</option>`).join('');
        }

        // Form Handlers
        function savePart(e) {
            e.preventDefault();
            const form = e.target;
            const newPart = {
                id: store.parts.length + 1,
                sku: form.sku.value,
                name: form.name.value,
                type: form.type.value,
                model: form.model.value,
                category: form.category.value,
                stock: parseInt(form.stock.value),
                min_stock: parseInt(form.min_stock.value),
                supplier: form.supplier.value,
                lead_time: parseInt(form.lead_time.value)
            };
            store.parts.push(newPart);
            closeModal('part-modal');
            renderInventory();
            updateDashboard();
            populateSelects();
        }

        function saveOrder(e) {
            e.preventDefault();
            const form = e.target;
            const partId = parseInt(form.part.value);
            const quantity = parseInt(form.quantity.value);
            const part = store.parts.find(p => p.id === partId);
            
            if (part.stock >= quantity) {
                part.stock -= quantity;
                alert(`Продажа оформлена! Остаток ${part.name}: ${part.stock}`);
                closeModal('order-modal');
                renderInventory();
                updateDashboard();
            } else {
                alert('Недостаточно запасов! Необходимо заказать у поставщика.');
            }
        }

        function openOrderModal(partId) {
            // Pre-select part in order modal
            populateSelects();
            document.getElementById('part-select').value = partId;
            openModal('order-modal');
        }

        // Chart
        function initChart() {
            const ctx = document.getElementById('stockChart').getContext('2d');
            new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Янв', 'Фев', 'Мар', 'Апр', 'Май', 'Июн'],
                    datasets: [{
                        label: 'Остатки на складе',
                        data: [65, 59, 80, 81, 56, 55],
                        borderColor: '#60a5fa',
                        backgroundColor: 'rgba(96, 165, 250, 0.1)',
                        tension: 0.4,
                        fill: true
                    }, {
                        label: 'Продажи',
                        data: [28, 48, 40, 19, 86, 27],
                        borderColor: '#34d399',
                        backgroundColor: 'rgba(52, 211, 153, 0.1)',
                        tension: 0.4,
                        fill: true
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: { labels: { color: '#94a3b8' } }
                    },
                    scales: {
                        y: { grid: { color: '#334155' }, ticks: { color: '#94a3b8' } },
                        x: { grid: { color: '#334155' }, ticks: { color: '#94a3b8' } }
                    }
                }
            });
        }

        // Search
        document.getElementById('search-parts')?.addEventListener('input', (e) => {
            const term = e.target.value.toLowerCase();
            const rows = document.querySelectorAll('#inventory-table tr');
            rows.forEach(row => {
                const text = row.textContent.toLowerCase();
                row.style.display = text.includes(term) ? '' : 'none';
            });
        });

        function generateReport() {
            alert('Формирование отчета по запасам...\nExcel файл будет скачан автоматически.');
        }

        // Close modals on outside click
        window.onclick = function(event) {
            if (event.target.classList.contains('fixed')) {
                event.target.classList.add('hidden');
                event.target.classList.remove('flex');
            }
        }
    </script>
</body>
Initial commit: inventory system
