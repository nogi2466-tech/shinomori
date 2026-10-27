<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>紫句守鉄道（しのもり鉄道）総合運行管理システム</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        brand: {
                            local: '#3b82f6',
                            semi: '#10b981',
                            rapid: '#f59e0b',
                            comrapid: '#d97706',
                            express: '#ef4444',
                            comexp: '#b91c1c',
                            ltdexp: '#8b5cf6'
                        }
                    }
                }
            }
        }
    </script>
    <!-- Firebase SDK (Compat) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    <style>
        * { box-sizing: border-box; }
        body { font-family: system-ui, -apple-system, sans-serif; transition: background-color 0.3s, color 0.3s; overflow-x: hidden; margin: 0; }
        
        .page-container {
            width: 100%;
            max-width: 1200px;
            margin: 0 auto;
        }

        .tab-content { display: none; }
        .tab-content.active { display: block; }

        /* 走行位置用スタイル（全駅縦方向配置・中央線） */
        .track-container-vertical {
            position: relative;
            min-height: 1400px;
            border-radius: 8px;
            padding: 40px 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            overflow: hidden;
            display: flex;
            justify-content: center;
            width: 100%;
        }
        .vertical-rail {
            position: absolute;
            top: 40px;
            bottom: 40px;
            left: 50%;
            width: 4px;
            background: #64748b;
            transform: translateX(-50%);
        }
        .v-station-node {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
            pointer-events: none;
            z-index: 5;
        }
        .v-station-dot {
            position: absolute;
            left: 50%;
            width: 12px;
            height: 12px;
            background: #94a3b8;
            border-radius: 50%;
            transform: translateX(-50%);
            z-index: 2;
        }
        .v-station-label {
            position: absolute;
            left: 50%;
            transform: translateX(-50%) translateY(-50%);
            font-size: 0.7rem;
            font-weight: bold;
            padding: 2px 6px;
            border-radius: 9999px;
            background: rgba(15, 23, 42, 0.95);
            border: 1px solid #475569;
            color: #e2e8f0;
            white-space: nowrap;
            z-index: 6;
            box-shadow: 0 2px 4px rgba(0,0,0,0.4);
        }
        .v-train {
            position: absolute;
            transform: translateY(-50%);
            cursor: pointer;
            z-index: 10;
        }
        .v-train.up-train {
            right: calc(50% + 25px);
            text-align: right;
        }
        .v-train.down-train {
            left: calc(50% + 25px);
            text-align: left;
        }
        .train-icon-v {
            width: 26px;
            height: 26px;
            border-radius: 6px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 10px;
            font-weight: bold;
            box-shadow: 0 2px 4px rgba(0,0,0,0.3);
            margin: 0 auto;
        }
        .train-card-v {
            color: white;
            border-radius: 6px;
            padding: 4px 8px;
            font-size: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.3);
            min-width: 90px;
        }
        .train-num-label {
            font-size: 9px;
            font-weight: bold;
            margin-bottom: 2px;
            white-space: nowrap;
        }

        /* 時刻表用スタイル */
        .tt-minute-item {
            cursor: pointer;
            display: inline-flex;
            flex-direction: column;
            align-items: center;
            margin-right: 10px;
            margin-bottom: 6px;
            transition: transform 0.1s;
        }
        .tt-minute-item:hover {
            transform: scale(1.15);
        }
        .tt-dest-char {
            font-size: 8px;
            line-height: 1;
            margin-bottom: 1px;
            opacity: 0.85;
        }
        .tt-minute-num {
            font-weight: bold;
            font-size: 13px;
            line-height: 1;
        }

        .modal-bg {
            background-color: rgba(0, 0, 0, 0.7);
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex flex-col relative dark">

    <!-- ヘッダー -->
    <header class="bg-indigo-950 border-b border-indigo-800 p-3 sm:p-4 shadow-lg flex justify-between items-center gap-2 relative z-50">
        <div class="page-container flex justify-between items-center px-1">
            <div class="flex items-center space-x-2 sm:space-x-3 cursor-pointer" onclick="switchTab('about')">
                <span class="text-2xl sm:text-3xl">🚄</span>
                <div>
                    <h1 class="text-base sm:text-xl font-bold tracking-wider text-indigo-200">紫句守鉄道 <span class="text-[10px] sm:text-xs font-normal text-indigo-400">しのもり鉄道</span></h1>
                    <p class="text-[10px] sm:text-xs text-slate-400">総合運行管理システム</p>
                </div>
            </div>

            <div class="flex items-center gap-2">
                <div id="live-datetime" class="hidden lg:block bg-slate-900 px-3 py-1.5 rounded-lg border border-slate-700 text-xs font-mono text-indigo-300">
                    2026/10/27(火) 00:00:00
                </div>
                <div id="event-banner-badge" class="hidden bg-rose-900 text-rose-200 px-2.5 py-1 rounded-lg text-xs font-bold border border-rose-700 animate-pulse">
                    🎉 イベント日
                </div>
                <div class="hidden sm:flex items-center space-x-2 bg-slate-900 px-2.5 py-1 rounded-lg border border-slate-700 text-[11px]">
                    <span id="sync-status-dot" class="w-2 h-2 rounded-full bg-amber-500 animate-pulse"></span>
                    <span id="sync-status-text">同期待機中</span>
                </div>
                <button onclick="toggleMenu()" class="bg-indigo-900 hover:bg-indigo-800 border border-indigo-700 p-2 rounded-lg text-white md:hidden transition flex items-center justify-center w-9 h-9 shadow">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
                    </svg>
                </button>
            </div>
        </div>
    </header>

    <!-- ナビゲーションタブ -->
    <nav class="bg-slate-900/95 border-b border-slate-800 px-4 py-2 sticky top-0 z-40 backdrop-blur shadow-md">
        <div id="nav-menu" class="page-container hidden md:flex overflow-x-auto space-x-1">
            <button onclick="switchTab('about')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition bg-indigo-600 text-white shadow whitespace-nowrap" data-tab="about">会社について</button>
            <button onclick="switchTab('timetable')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="timetable">時刻表</button>
            <button onclick="switchTab('operation')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="operation">走行位置</button>
            <button onclick="switchTab('traininfo')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="traininfo">列車情報</button>
            <button onclick="switchTab('addtrain')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="addtrain">列車追加</button>
            <button onclick="switchTab('consist')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="consist">編成表</button>
            <button onclick="switchTab('dia')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="dia">ダイヤ表</button>
            <button onclick="switchTab('settings')" class="tab-btn px-3 py-2 rounded-lg text-xs sm:text-sm font-medium transition text-slate-400 hover:text-white hover:bg-slate-800 whitespace-nowrap" data-tab="settings">設定</button>
        </div>
    </nav>

    <!-- メインコンテンツ -->
    <main class="flex-1 p-3 sm:p-6 page-container w-full relative z-10">

        <!-- 1. 会社について -->
        <div id="tab-about" class="tab-content active space-y-6">
            <div class="bg-gradient-to-r from-indigo-900 to-slate-800 p-5 sm:p-6 rounded-2xl border border-indigo-700/50 shadow-xl space-y-3">
                <h2 class="text-xl sm:text-2xl font-bold text-indigo-100">紫句守鉄道株式会社 <span class="text-xs sm:text-sm font-normal text-indigo-300">Shinomori Railway Co., Ltd.</span></h2>
                <p class="text-slate-300 text-xs sm:text-sm leading-relaxed">
                    紫句守鉄道（しのもり鉄道）は、紫雲本線を軸に、句守支線・紫霞観光線・星句高原線および他社直通路線を結ぶ基幹鉄道ネットワークです。
                </p>
            </div>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 sm:gap-6">
                <div class="bg-slate-800 p-4 sm:p-5 rounded-xl border border-slate-700 space-y-3">
                    <h3 class="text-indigo-400 font-bold border-b border-slate-700 pb-2 text-sm sm:text-base">路線一覧</h3>
                    <ul class="text-xs sm:text-sm text-slate-300 space-y-2 font-mono">
                        <li><span class="text-slate-400 inline-block w-28 sm:w-32 font-sans">紫雲本線</span> 1～30</li>
                        <li><span class="text-slate-400 inline-block w-28 sm:w-32 font-sans">句守支線</span> 41～55</li>
                        <li><span class="text-slate-400 inline-block w-28 sm:w-32 font-sans">紫霞観光線</span> 56～60</li>
                        <li><span class="text-slate-400 inline-block w-28 sm:w-32 font-sans">星句高原線</span> 31～40</li>
                        <li><span class="text-slate-400 inline-block w-28 sm:w-32 font-sans">他社直通路線</span> 09～1</li>
                    </ul>
                </div>
                <div class="bg-slate-800 p-4 sm:p-5 rounded-xl border border-slate-700 space-y-3">
                    <h3 class="text-indigo-400 font-bold border-b border-slate-700 pb-2 text-sm sm:text-base">主要設備・車両基地</h3>
                    <div class="text-xs sm:text-sm text-slate-300 space-y-2">
                        <p><strong class="text-indigo-300">副本線（4番線設置駅）:</strong><br><span class="font-mono text-[11px] sm:text-xs text-slate-400">1, 3, 13, 17, 20, 24, 30, 34, 42, 45, 50, 55, 60</span></p>
                        <p><strong class="text-indigo-300">車両基地設置駅:</strong><br><span class="font-mono text-[11px] sm:text-xs text-slate-400">9, 24, 30, 33, 42</span></p>
                    </div>
                </div>
            </div>
        </div>

        <!-- 2. 時刻表 -->
        <div id="tab-timetable" class="tab-content space-y-4">
            <h2 class="text-lg sm:text-xl font-bold text-indigo-200">全駅時刻表</h2>
            <div class="bg-slate-800 p-3 sm:p-4 rounded-xl border border-slate-700 grid grid-cols-1 sm:grid-cols-4 gap-3 items-end">
                <div>
                    <label class="block text-xs text-slate-400 mb-1">駅選択</label>
                    <select id="tt-station" onchange="renderTimetable()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white outline-none focus:border-indigo-500"></select>
                </div>
                <div>
                    <label class="block text-xs text-slate-400 mb-1">方向（上下切替）</label>
                    <select id="tt-dir" onchange="renderTimetable()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white outline-none focus:border-indigo-500">
                        <option value="down">下り (本線下り・各支線方面)</option>
                        <option value="up">上り (本線上り・中央方面)</option>
                    </select>
                </div>
                <div>
                    <label class="block text-xs text-slate-400 mb-1">運行日</label>
                    <select id="tt-day" onchange="renderTimetable()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white outline-none focus:border-indigo-500">
                        <option value="weekday">平日</option>
                        <option value="holiday">土休日</option>
                    </select>
                </div>
                <button onclick="renderTimetable()" class="bg-indigo-600 hover:bg-indigo-500 text-white px-4 py-2 rounded text-xs sm:text-sm font-bold transition shadow">時刻表を更新</button>
            </div>

            <div id="timetable-container" class="bg-slate-800 rounded-xl border border-slate-700 p-4 sm:p-5 overflow-x-auto text-sm shadow-xl">
                <p class="text-slate-400 text-center py-4 text-xs sm:text-sm">駅を選択してください。</p>
            </div>
        </div>

        <!-- 3. 走行位置 -->
        <div id="tab-operation" class="tab-content space-y-4">
            <div class="flex flex-wrap justify-between items-center gap-3">
                <h2 class="text-lg sm:text-xl font-bold text-indigo-200">列車走行位置</h2>
                <div class="flex items-center gap-2">
                    <select id="op-line-select" onchange="renderOperationTrack()" class="bg-slate-800 border border-slate-700 rounded px-3 py-1.5 text-xs text-white">
                        <option value="main">紫雲本線 (1～30)</option>
                        <option value="branch_ksh">句守支線 (41～55)</option>
                        <option value="branch_skk">紫霞観光線 (56～60)</option>
                        <option value="branch_skh">星句高原線 (31～40)</option>
                        <option value="branch_oth">他社直通線 (09～1)</option>
                    </select>
                </div>
            </div>
            
            <div class="controls bg-slate-800 p-3 sm:p-4 rounded-xl border border-slate-700 flex flex-wrap items-center justify-between gap-3">
                <div class="flex items-center space-x-3">
                    <label class="flex items-center space-x-2 text-xs sm:text-sm text-slate-200 cursor-pointer">
                        <input type="checkbox" id="toggleView" onchange="renderOperationTrack()" class="rounded bg-slate-900 border-slate-700 text-indigo-600 focus:ring-0">
                        <span>四角形（詳細カード）表示</span>
                    </label>
                </div>
                <div class="text-[11px] sm:text-xs text-indigo-300">
                    📍 中央線の上に駅名、<strong>左：上り</strong> ／ <strong>右：下り</strong>
                </div>
            </div>

            <div id="track-container-parent" class="track-container-vertical bg-slate-800 border border-slate-700">
                <div class="vertical-rail"></div>
            </div>

            <div class="status-panel bg-slate-800 p-3 sm:p-4 rounded-xl border border-slate-700 flex justify-between items-center text-xs sm:text-sm">
                <p class="text-slate-300 truncate"><strong>運行状態:</strong> <span id="statusText" class="text-indigo-300 font-bold">停車中・走行中を正確に反映</span></p>
                <button onclick="renderOperationTrack()" class="bg-indigo-600 hover:bg-indigo-500 text-white px-3 py-1.5 rounded text-xs transition font-bold shrink-0">更新</button>
            </div>
        </div>

        <!-- 4. 列車情報 -->
        <div id="tab-traininfo" class="tab-content space-y-4">
            <div class="flex flex-wrap justify-between items-center gap-3">
                <h2 class="text-lg sm:text-xl font-bold text-indigo-200">列車情報</h2>
                <div class="flex items-center gap-2">
                    <button id="btn-mode-normal" onclick="setInfoViewMode('normal')" class="px-3 py-1.5 rounded text-xs font-bold bg-indigo-600 text-white">一覧</button>
                    <button id="btn-mode-opnum" onclick="setInfoViewMode('opnum')" class="px-3 py-1.5 rounded text-xs font-bold bg-slate-800 text-slate-400 hover:text-white">運用別</button>
                </div>
            </div>

            <div id="info-op-bar" class="bg-slate-800 p-3 rounded-xl border border-slate-700 flex flex-wrap items-center gap-3 text-xs">
                <span class="text-slate-400 font-bold">操作:</span>
                <label class="flex items-center space-x-1 cursor-pointer"><input type="radio" name="info-action-mode" value="normal" checked class="text-indigo-600"> <span>通常</span></label>
                <label class="flex items-center space-x-1 cursor-pointer"><input type="radio" name="info-action-mode" value="edit" class="text-indigo-600"> <span>編集</span></label>
                <label class="flex items-center space-x-1 cursor-pointer"><input type="radio" name="info-action-mode" value="clone" class="text-indigo-600"> <span>複製</span></label>
                <label class="flex items-center space-x-1 cursor-pointer"><input type="radio" name="info-action-mode" value="delete" class="text-indigo-600"> <span>削除</span></label>
            </div>

            <div id="train-info-content" class="bg-slate-800 rounded-xl border border-slate-700 overflow-x-auto shadow-xl"></div>
        </div>

        <!-- 5. 列車追加 -->
        <div id="tab-addtrain" class="tab-content space-y-4">
            <h2 id="addtrain-form-title" class="text-lg sm:text-xl font-bold text-indigo-200">列車追加</h2>
            <div class="bg-slate-800 p-4 sm:p-6 rounded-xl border border-slate-700 w-full space-y-4 sm:space-y-5">
                <input type="hidden" id="edit-target-id" value="">

                <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">列車番号</label>
                        <input type="text" id="add-train-num" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white" value="6001M">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">運用番号</label>
                        <input type="text" id="add-op-num" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white" value="y01">
                    </div>
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">路線</label>
                        <select id="add-line" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white">
                            <option value="main">紫雲本線</option>
                            <option value="branch_ksh">句守支線</option>
                            <option value="branch_skk">紫霞観光線</option>
                            <option value="branch_skh">星句高原線</option>
                            <option value="branch_oth">他社直通線</option>
                        </select>
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">種別</label>
                        <select id="add-train-type" onchange="updateStationSchedulePreview()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white">
                            <option value="普通" selected>普通</option>
                            <option value="準急">準急</option>
                            <option value="快速">快速</option>
                            <option value="通勤快速">通勤快速</option>
                            <option value="急行">急行</option>
                            <option value="通勤急行">通勤急行</option>
                            <option value="特急">特急</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">編成番号</label>
                        <input type="text" id="add-consist-num" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white" value="S2-01 (10両)">
                    </div>
                </div>

                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">始点駅</label>
                        <select id="add-start-station" onchange="updateStationSchedulePreview()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white"></select>
                    </div>
                    <div>
                        <label class="block text-xs text-slate-400 mb-1">終点駅（行き先）</label>
                        <select id="add-end-station" onchange="updateStationSchedulePreview()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white"></select>
                    </div>
                </div>

                <div class="space-y-2">
                    <span class="text-xs text-indigo-300 font-bold block">停車駅および時刻設定</span>
                    <div id="station-schedule-preview" class="bg-slate-900 border border-slate-700 rounded-lg p-2 sm:p-3 max-h-60 overflow-y-auto text-xs"></div>
                </div>

                <div class="flex gap-3">
                    <button onclick="saveTrainData()" class="flex-1 bg-indigo-600 hover:bg-indigo-500 text-white font-bold py-2.5 rounded-lg transition shadow text-xs sm:text-sm">列車を保存</button>
                    <button id="btn-cancel-edit" onclick="resetAddTrainForm()" class="hidden bg-slate-700 hover:bg-slate-600 text-slate-200 font-bold px-4 py-2.5 rounded-lg transition text-xs sm:text-sm">キャンセル</button>
                </div>
            </div>
        </div>

        <!-- 6. 編成表 -->
        <div id="tab-consist" class="tab-content space-y-4">
            <h2 class="text-lg sm:text-xl font-bold text-indigo-200">編成表</h2>
            <div id="consist-matrix-container" class="space-y-4"></div>
        </div>

        <!-- 7. ダイヤ表 -->
        <div id="tab-dia" class="tab-content space-y-4">
            <h2 class="text-lg sm:text-xl font-bold text-indigo-200">ダイヤ表</h2>
            <div class="bg-slate-800 p-3 sm:p-4 rounded-xl border border-slate-700 overflow-x-auto shadow-xl">
                <div id="matrix-timetable-container" class="min-w-[600px] sm:min-w-[800px]"></div>
            </div>
        </div>

        <!-- 8. 設定 -->
        <div id="tab-settings" class="tab-content space-y-4">
            <h2 class="text-lg sm:text-xl font-bold text-indigo-200">設定</h2>
            <div class="bg-slate-800 p-5 sm:p-6 rounded-xl border border-slate-700 w-full space-y-5">
                <div class="space-y-3">
                    <h3 class="text-sm font-bold text-indigo-300 border-b border-slate-700 pb-1">運行日固定設定</h3>
                    <select id="setting-fixed-day" onchange="saveSettings()" class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-xs sm:text-sm text-white">
                        <option value="auto">自動（本日の曜日に準拠）</option>
                        <option value="weekday">平日ダイヤ固定</option>
                        <option value="holiday">土休日ダイヤ固定</option>
                    </select>
                </div>
                <div class="space-y-3">
                    <h3 class="text-sm font-bold text-indigo-300 border-b border-slate-700 pb-1">テーマモード</h3>
                    <div class="flex items-center justify-between">
                        <span class="text-xs sm:text-sm text-slate-300">ダークモード / ライトモード</span>
                        <button onclick="toggleTheme()" id="theme-toggle-btn" class="bg-indigo-600 hover:bg-indigo-500 text-white px-3 py-1.5 rounded text-xs font-bold transition">切替</button>
                    </div>
                </div>
                <div class="space-y-3 pt-3 border-t border-slate-700">
                    <h3 class="text-sm font-bold text-indigo-300">クラウド同期 (Firebase)</h3>
                    <input type="text" id="fb-url" readonly class="w-full bg-slate-900 border border-slate-700 rounded px-3 py-2 text-[11px] text-slate-400 cursor-not-allowed font-mono">
                </div>
            </div>
        </div>

    </main>

    <!-- 列車詳細ポップアップ Modal -->
    <div id="train-detail-modal" class="fixed inset-0 modal-bg z-50 flex items-center justify-center p-3 sm:p-4 hidden">
        <div class="bg-slate-800 border border-indigo-500/50 rounded-2xl max-w-md w-full p-5 sm:p-6 space-y-4 shadow-2xl relative">
            <button onclick="closeModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white font-bold text-xl">✕</button>
            <div id="modal-content" class="space-y-3 text-xs sm:text-sm"></div>
            <button onclick="closeModal()" class="w-full bg-indigo-600 hover:bg-indigo-500 text-white py-2 rounded-lg font-bold text-xs transition">閉じる</button>
        </div>
    </div>

    <script>
        const allStationsMaster = [
            {id: "09", name: "水鳥湿原"}, {id: "08", name: "青蓮寺"}, {id: "07", name: "紫水"}, {id: "06", name: "瑠璃川"}, {id: "05", name: "翡翠野"}, {id: "04", name: "琥珀谷"}, {id: "03", name: "瑪瑙台"}, {id: "02", name: "天翔"},
            {id: "1", name: "紫句守中央"}, {id: "2", name: "霞詠ヶ丘"}, {id: "3", name: "紫雲野"}, {id: "4", name: "詩羽町"}, {id: "5", name: "星句台"}, {id: "6", name: "紫陽花前"}, {id: "7", name: "句守書院"}, {id: "8", name: "銀墨坂"}, {id: "9", name: "紫峰高原"}, {id: "10", name: "風詠の森"},
            {id: "11", name: "宵月町"}, {id: "12", name: "句読通り"}, {id: "13", name: "紫霞野"}, {id: "14", name: "鏡句池"}, {id: "15", name: "雨詠坂"}, {id: "16", name: "紫都新町"}, {id: "17", name: "句守空港"}, {id: "18", name: "星詠港"}, {id: "19", name: "紫光浜"}, {id: "20", name: "潮句の杜"},
            {id: "21", name: "紫苑台"}, {id: "22", name: "句守温泉"}, {id: "23", name: "霧詠峠"}, {id: "24", name: "鷲羽句守"}, {id: "25", name: "紫野学園前"}, {id: "26", name: "書詠通り"}, {id: "27", name: "月句台"}, {id: "28", name: "紫句守美術館"}, {id: "29", name: "詩風町"}, {id: "30", name: "紫句守展示場"},
            {id: "31", name: "星句高原入口"}, {id: "32", name: "星霧の丘"}, {id: "33", name: "天詠台"}, {id: "34", name: "星句牧場前"}, {id: "35", name: "星句温泉郷"}, {id: "36", name: "星句森林"}, {id: "37", name: "星句湖畔"}, {id: "38", name: "句守詩碑前"}, {id: "39", name: "星句高原村"}, {id: "40", name: "星句高原"},
            {id: "41", name: "紫句守港"}, {id: "42", name: "紫句守湾岸"}, {id: "43", name: "句守湾岸"}, {id: "44", name: "紫句守タワー"}, {id: "45", name: "句守の杜"}, {id: "46", name: "紫句守工業団地"}, {id: "47", name: "紫句守工業"}, {id: "48", name: "鉄輪詩町"}, {id: "49", name: "紫句守市場"}, {id: "50", name: "詩詠の里"},
            {id: "51", name: "紫句守農園"}, {id: "52", name: "紫句守劇場前"}, {id: "53", name: "句守未来都市"}, {id: "54", name: "紫句守研究所"}, {id: "55", name: "詩句の丘"},
            {id: "56", name: "紫句守展望台"}, {id: "57", name: "句守星見台"}, {id: "58", name: "紫句守森林公園"}, {id: "59", name: "紫句守詩碑前"}, {id: "60", name: "紫句守詩碑"}
        ];

        const typeColorMap = {
            "普通": "bg-brand-local text-white border-blue-400",
            "準急": "bg-brand-semi text-white border-emerald-400",
            "快速": "bg-brand-rapid text-white border-amber-400",
            "通勤快速": "bg-brand-comrapid text-white border-amber-600",
            "急行": "bg-brand-express text-white border-red-400",
            "通勤急行": "bg-brand-comexp text-white border-red-700",
            "特急": "bg-brand-ltdexp text-white border-purple-400"
        };

        const typeTextColorMap = {
            "普通": "text-blue-400", "準急": "text-emerald-400", "快速": "text-amber-400",
            "通勤快速": "text-amber-500", "急行": "text-red-400", "通勤急行": "text-red-600", "特急": "text-purple-400"
        };

        const fullConsistsMaster = [
            { series: "S1系", items: [{ code: "01", text: "S1-01 (10両)" }, { code: "06", text: "S1-06 (8両)" }] },
            { series: "S2系", items: [{ code: "01", text: "S2-01 (10両)" }, { code: "16", text: "S2-16 (8両)" }] },
            { series: "S3系", items: [{ code: "01", text: "S3-01 (10両)" }, { code: "21", text: "S3-21 (4両)" }] },
            { series: "S100系 (特急)", items: [{ code: "01", text: "S100-01 (10両)" }, { code: "16", text: "S100-16 (4両)" }] }
        ];

        const defaultTrainsData = [
            {
                id: 1001,
                trainNum: "0001M",
                opNum: "m01",
                line: "main",
                type: "特急",
                startSt: "1",
                endSt: "30",
                consistNum: "S100-01 (10両)",
                runDay: "weekday",
                schedule: [
                    { stId: "1", arr: "08:00", dep: "08:02" },
                    { stId: "3", arr: "08:08", dep: "08:09" },
                    { stId: "13", arr: "08:20", dep: "08:21" },
                    { stId: "16", arr: "08:28", dep: "08:29" },
                    { stId: "17", arr: "08:33", dep: "08:34" },
                    { stId: "23", arr: "08:45", dep: "08:46" },
                    { stId: "28", arr: "08:55", dep: "08:56" },
                    { stId: "30", arr: "09:02", dep: "09:02" }
                ]
            }
        ];

        let appData = { trains: defaultTrainsData, settings: { fixedDay: "auto" }, consistAssignments: {} };
        let infoViewMode = 'normal';
        let dbRef = null;
        const DEFAULT_FB_URL = "https://original-tetsudo-430ac-default-rtdb.firebaseio.com";

        function initFirebase() {
            try {
                if(firebase.apps.length === 0) {
                    firebase.initializeApp({ databaseURL: DEFAULT_FB_URL });
                }
                dbRef = firebase.database().ref('shinomori_railway_app');
                dbRef.on('value', (snapshot) => {
                    const val = snapshot.val();
                    if(val) {
                        appData = val;
                        if(!appData.trains) appData.trains = defaultTrainsData;
                        if(!appData.settings) appData.settings = { fixedDay: "auto" };
                        if(!appData.consistAssignments) appData.consistAssignments = {};
                        updateUI();
                    } else {
                        pushData();
                    }
                });
                document.getElementById('sync-status-dot').className = "w-2 h-2 rounded-full bg-emerald-500 animate-pulse";
                document.getElementById('sync-status-text').innerText = "同期中";
                document.getElementById('fb-url').value = DEFAULT_FB_URL;
            } catch(e) {
                document.getElementById('sync-status-dot').className = "w-2 h-2 rounded-full bg-red-500";
                document.getElementById('sync-status-text').innerText = "オフライン";
            }
        }

        function pushData() { if(dbRef) dbRef.set(appData); }
        function updateUI() {
            renderTimetable();
            renderTrainInfo();
            renderConsistMatrix();
            renderMatrixTimetable();
            renderOperationTrack();
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.querySelectorAll('.tab-btn').forEach(btn => {
                btn.classList.remove('bg-indigo-600', 'text-white', 'shadow');
                btn.classList.add('text-slate-400', 'hover:text-white', 'hover:bg-slate-800');
            });
            document.getElementById('tab-' + tabId).classList.add('active');
            const targetBtn = document.querySelector(`[data-tab="${tabId}"]`);
            if (targetBtn) {
                targetBtn.classList.add('bg-indigo-600', 'text-white', 'shadow');
                targetBtn.classList.remove('text-slate-400', 'hover:text-white', 'hover:bg-slate-800');
            }
            const menu = document.getElementById('nav-menu');
            if(window.innerWidth < 768 && !menu.classList.contains('hidden')) {
                toggleMenu();
            }

            if (tabId === 'dia') renderMatrixTimetable();
            if (tabId === 'timetable') initTimetableDropdowns();
            if (tabId === 'traininfo') renderTrainInfo();
            if (tabId === 'consist') renderConsistMatrix();
            if (tabId === 'operation') renderOperationTrack();
            if (tabId === 'addtrain') {
                populateStationDropdowns('add-start-station');
                populateStationDropdowns('add-end-station');
                updateStationSchedulePreview();
            }
        }

        function toggleMenu() {
            const menu = document.getElementById('nav-menu');
            if (menu.classList.contains('hidden')) {
                menu.classList.remove('hidden');
                menu.classList.add('flex', 'flex-col', 'absolute', 'top-14', 'left-0', 'right-0', 'bg-slate-950/98', 'border-b', 'border-indigo-800', 'p-3', 'shadow-2xl', 'z-50', 'space-y-1');
            } else {
                menu.classList.add('hidden');
                menu.classList.remove('flex', 'flex-col', 'absolute', 'top-14', 'left-0', 'right-0', 'bg-slate-950/98', 'border-b', 'border-indigo-800', 'p-3', 'shadow-2xl', 'z-50', 'space-y-1');
            }
        }

        function initTimetableDropdowns() {
            const stSelect = document.getElementById('tt-station');
            if(!stSelect) return;
            stSelect.innerHTML = allStationsMaster.map(st => `<option value="${st.id}">${st.id}. ${st.name}</option>`).join('');
            renderTimetable();
        }

        function renderTimetable() {
            const stId = document.getElementById('tt-station').value || "1";
            const container = document.getElementById('timetable-container');
            const stObj = allStationsMaster.find(s => s.id === stId);
            if(!container || !stObj) return;

            const hourMap = {};
            for(let h = 4; h <= 23; h++) hourMap[h] = [];

            (appData.trains || []).forEach(t => {
                const item = t.schedule?.find(s => s.stId === stId && s.dep && s.dep !== '通過' && s.dep !== '-');
                if(item) {
                    const [h, m] = item.dep.split(':').map(Number);
                    if(hourMap[h]) {
                        const destObj = allStationsMaster.find(s => s.id === t.endSt);
                        hourMap[h].push({ min: m, minStr: String(m).padStart(2, '0'), type: t.type, destChar: destObj ? destObj.name.charAt(0) : "行", trainId: t.id });
                    }
                }
            });

            let html = `<div class="mb-3 pb-2 border-b border-slate-700 flex justify-between items-center"><span class="text-sm sm:text-base font-bold text-indigo-300">【${stObj.id}. ${stObj.name} 駅】 発車時刻表</span></div><div class="space-y-2">`;
            for(let h = 4; h <= 23; h++) {
                const list = hourMap[h].sort((a,b) => a.min - b.min);
                html += `<div class="flex border-b border-slate-700/60 pb-2 items-baseline"><div class="w-10 sm:w-12 font-mono text-xl sm:text-2xl font-bold text-indigo-400">${String(h).padStart(2, '0')}</div><div class="flex-1 flex flex-wrap items-center">`;
                if(list.length === 0) html += `<span class="text-slate-600 text-xs font-mono">-</span>`;
                else {
                    list.forEach(it => {
                        html += `<div onclick="showTrainDetail(${it.trainId})" class="tt-minute-item ${typeTextColorMap[it.type] || "text-slate-200"}"><span class="tt-dest-char text-slate-400">${it.destChar}</span><span class="tt-minute-num">${it.minStr}</span></div>`;
                    });
                }
                html += `</div></div>`;
            }
            container.innerHTML = html + `</div>`;
        }

        /* 停車中・走行中を正確に判定するロジック */
        function getTrainStatus(t) {
            if(!t.schedule || t.schedule.length < 2) return "運行前";
            const schedSec = t.schedule.map(s => ({
                stId: s.stId,
                arr: parseTimeToSec(s.arr),
                dep: parseTimeToSec(s.dep)
            })).filter(s => s.arr !== null || s.dep !== null);

            if(schedSec.length < 2) return "運行前";
            const now = new Date();
            const nowSec = now.getHours() * 3600 + now.getMinutes() * 60 + now.getSeconds() + (now.getMilliseconds() / 1000);
            
            const firstDep = schedSec[0].dep !== null ? schedSec[0].dep : schedSec[0].arr;
            const lastArr = schedSec[schedSec.length - 1].arr !== null ? schedSec[schedSec.length - 1].arr : schedSec[schedSec.length - 1].dep;

            if(nowSec < firstDep) return "運行前";
            if(nowSec > lastArr + 30) return "運行終了";

            for(let i = 0; i < schedSec.length; i++) {
                const cur = schedSec[i];
                const arrTime = cur.arr !== null ? cur.arr : cur.dep;
                const depTime = cur.dep !== null ? cur.dep : cur.arr;
                // 到着時刻から出発時刻までの間は「停車中」
                if(arrTime !== null && depTime !== null && nowSec >= arrTime && nowSec <= depTime) {
                    return "停車中";
                }
            }
            return "走行中";
        }

        function renderOperationTrack() {
            const containerParent = document.getElementById('track-container-parent');
            if(!containerParent) return;

            const lineKey = document.getElementById('op-line-select').value;
            const isDetail = document.getElementById('toggleView').checked;

            let stations = [];
            if(lineKey === 'main') stations = allStationsMaster.slice(8, 38);
            else if(lineKey === 'branch_ksh') stations = allStationsMaster.slice(40, 55);
            else if(lineKey === 'branch_skk') stations = allStationsMaster.slice(55, 60);
            else if(lineKey === 'branch_skh') stations = allStationsMaster.slice(30, 40);
            else if(lineKey === 'branch_oth') stations = allStationsMaster.slice(0, 9);

            const totalStations = stations.length;
            const containerHeight = Math.max(1200, totalStations * 50 + 80);
            containerParent.style.height = `${containerHeight}px`;

            let html = `<div class="vertical-rail" style="height: ${containerHeight - 80}px;"></div>`;
            const spacing = (containerHeight - 80) / Math.max(1, totalStations - 1);

            stations.forEach((st, idx) => {
                const topPos = 40 + (idx * spacing);
                html += `
                    <div class="v-station-node" style="top: ${topPos}px;">
                        <div class="v-station-dot"></div>
                        <div class="v-station-label">${st.id}. ${st.name}</div>
                    </div>
                `;
            });

            const now = new Date();
            const nowSec = now.getHours() * 3600 + now.getMinutes() * 60 + now.getSeconds() + (now.getMilliseconds() / 1000);

            (appData.trains || []).forEach(t => {
                if(!t.schedule || t.schedule.length < 2) return;

                const schedSec = t.schedule.map(s => ({
                    stId: s.stId,
                    arr: parseTimeToSec(s.arr),
                    dep: parseTimeToSec(s.dep)
                })).filter(s => s.arr !== null || s.dep !== null);

                if(schedSec.length < 2) return;
                const firstDep = schedSec[0].dep !== null ? schedSec[0].dep : schedSec[0].arr;
                const lastArr = schedSec[schedSec.length - 1].arr !== null ? schedSec[schedSec.length - 1].arr : schedSec[schedSec.length - 1].dep;

                if(nowSec < firstDep || nowSec > lastArr + 30) return;

                let currentPosRatio = -1;
                let currentStationIdx = -1;
                let status = getTrainStatus(t);

                for(let i = 0; i < schedSec.length - 1; i++) {
                    const cur = schedSec[i];
                    const next = schedSec[i+1];
                    const curDep = cur.dep !== null ? cur.dep : cur.arr;
                    const nextArr = next.arr !== null ? next.arr : next.dep;
                    const curArr = cur.arr !== null ? cur.arr : cur.dep;

                    if(curArr !== null && curDep !== null && nowSec >= curArr && nowSec <= curDep) {
                        currentStationIdx = stations.findIndex(s => s.id === cur.stId);
                        break;
                    } else if(curDep !== null && nextArr !== null && nowSec > curDep && nowSec < nextArr) {
                        const idxA = stations.findIndex(s => s.id === cur.stId);
                        const idxB = stations.findIndex(s => s.id === next.stId);
                        if(idxA !== -1 && idxB !== -1) {
                            const prog = (nowSec - curDep) / (nextArr - curDep);
                            currentPosRatio = idxA + (idxB - idxA) * prog;
                        }
                        break;
                    }
                }

                let topPos = -1;
                if(currentStationIdx !== -1) topPos = 40 + (currentStationIdx * spacing);
                else if(currentPosRatio !== -1) topPos = 40 + (currentPosRatio * spacing);

                if(topPos >= 20 && topPos <= containerHeight - 20) {
                    const isUp = parseInt(t.startSt) > parseInt(t.endSt);
                    const arrow = isUp ? "▲" : "▼";
                    const badgeClass = typeColorMap[t.type] || "bg-indigo-600 text-white";
                    const destName = allStationsMaster.find(s => s.id === t.endSt)?.name || t.endSt;

                    if(isDetail) {
                        html += `
                            <div onclick="showTrainDetail(${t.id})" class="v-train ${isUp ? 'up-train' : 'down-train'}" style="top: ${topPos}px;">
                                ${isUp ? `<div class="train-num-label text-indigo-300">${t.trainNum} ${arrow}</div>` : ''}
                                <div class="train-card-v ${badgeClass}">
                                    <div class="font-bold text-[11px]">${t.type} ${destName}行</div>
                                    <div class="text-[9px] opacity-90">${t.opNum} | ${status}</div>
                                </div>
                                ${!isUp ? `<div class="train-num-label text-indigo-300">${arrow} ${t.trainNum}</div>` : ''}
                            </div>
                        `;
                    } else {
                        html += `
                            <div onclick="showTrainDetail(${t.id})" class="v-train ${isUp ? 'up-train' : 'down-train'}" style="top: ${topPos}px;" title="${t.trainNum}">
                                <div class="train-icon-v ${badgeClass}">${arrow}</div>
                            </div>
                        `;
                    }
                }
            });

            containerParent.innerHTML = html;
        }

        function parseTimeToSec(timeStr) {
            if(!timeStr || timeStr === '-' || timeStr === '通過') return null;
            const [h, m] = timeStr.split(':').map(Number);
            return h * 3600 + m * 60;
        }

        function setInfoViewMode(mode) {
            infoViewMode = mode;
            document.getElementById('btn-mode-normal').className = mode === 'normal' ? "px-3 py-1.5 rounded text-xs font-bold bg-indigo-600 text-white" : "px-3 py-1.5 rounded text-xs font-bold bg-slate-800 text-slate-400";
            document.getElementById('btn-mode-opnum').className = mode === 'opnum' ? "px-3 py-1.5 rounded text-xs font-bold bg-indigo-600 text-white" : "px-3 py-1.5 rounded text-xs font-bold bg-slate-800 text-slate-400";
            document.getElementById('info-op-bar').style.display = mode === 'normal' ? 'flex' : 'none';
            renderTrainInfo();
        }

        function renderTrainInfo() {
            const container = document.getElementById('train-info-content');
            if(!container) return;
            const trains = appData.trains || [];

            if(infoViewMode === 'opnum') {
                const opGroup = {};
                trains.forEach(t => {
                    const op = t.opNum || "未設定";
                    if(!opGroup[op]) opGroup[op] = [];
                    opGroup[op].push(t);
                });
                let html = `<div class="p-3 sm:p-4 space-y-3">`;
                Object.keys(opGroup).sort().forEach(op => {
                    html += `<div class="bg-slate-900 border border-slate-700 p-3 sm:p-4 rounded-xl flex flex-col sm:flex-row items-start sm:items-center gap-3"><div class="text-xl font-mono font-bold text-indigo-300 w-20 text-center bg-indigo-950 p-2 rounded-lg border border-indigo-800">${op}</div><div class="flex-1 flex flex-wrap gap-2">`;
                    opGroup[op].forEach(t => {
                        html += `<div onclick="showTrainDetail(${t.id})" class="bg-slate-800 border border-slate-700 p-2 rounded-lg text-xs cursor-pointer hover:border-indigo-500"><div class="font-mono text-indigo-400 font-bold">${t.trainNum}</div><div>${t.type}</div></div>`;
                    });
                    html += `</div></div>`;
                });
                container.innerHTML = html + `</div>`;
            } else {
                let html = `<div class="overflow-x-auto"><table class="w-full text-left text-xs whitespace-nowrap"><thead class="bg-slate-900 text-indigo-200 border-b border-slate-700"><tr><th class="p-3">列車番号</th><th class="p-3">運用</th><th class="p-3">種別</th><th class="p-3">行き先</th><th class="p-3">編成</th><th class="p-3">状態</th></tr></thead><tbody class="divide-y divide-slate-700 text-slate-300">`;
                trains.forEach(t => {
                    const dest = allStationsMaster.find(s => s.id === t.endSt)?.name || t.endSt;
                    const st = getTrainStatus(t);
                    let stBadge = "bg-slate-700 text-slate-300";
                    if(st === "走行中") stBadge = "bg-emerald-950 text-emerald-300 border border-emerald-700";
                    if(st === "停車中") stBadge = "bg-sky-950 text-sky-300 border border-sky-700";
                    if(st === "運行準備中") stBadge = "bg-amber-950 text-amber-300 border border-amber-700";

                    html += `<tr class="hover:bg-slate-750 cursor-pointer" onclick="handleTrainRowClick(${t.id})"><td class="p-3 font-bold text-indigo-300 font-mono">${t.trainNum}</td><td class="p-3 font-mono text-indigo-400">${t.opNum}</td><td class="p-3"><span class="px-2 py-0.5 rounded text-[10px] font-bold ${typeColorMap[t.type] || ''}">${t.type}</span></td><td class="p-3 font-bold text-slate-200">${dest}行</td><td class="p-3 font-mono text-slate-400">${t.consistNum || '-'}</td><td class="p-3"><span class="px-2 py-0.5 rounded text-[10px] font-bold ${stBadge}">${st}</span></td></tr>`;
                });
                container.innerHTML = html + `</tbody></table></div>`;
            }
        }

        function handleTrainRowClick(trainId) {
            const mode = document.querySelector('input[name="info-action-mode"]:checked')?.value || 'normal';
            if(mode === 'edit') editTrain(trainId);
            else if(mode === 'clone') cloneTrain(trainId);
            else if(mode === 'delete') deleteTrain(trainId);
            else showTrainDetail(trainId);
        }

        function cloneTrain(trainId) {
            const t = appData.trains.find(item => item.id === trainId);
            if(t) {
                const newTrain = JSON.parse(JSON.stringify(t));
                newTrain.id = Date.now();
                newTrain.trainNum += "_copy";
                appData.trains.push(newTrain);
                pushData();
                renderTrainInfo();
            }
        }

        function deleteTrain(trainId) {
            if(confirm("削除しますか？")) {
                appData.trains = appData.trains.filter(item => item.id !== trainId);
                pushData();
                renderTrainInfo();
            }
        }

        function editTrain(trainId) {
            const t = appData.trains.find(item => item.id === trainId);
            if(!t) return;
            document.getElementById('edit-target-id').value = t.id;
            document.getElementById('add-train-num').value = t.trainNum;
            document.getElementById('add-op-num').value = t.opNum;
            document.getElementById('add-line').value = t.line || 'main';
            document.getElementById('add-train-type').value = t.type;
            document.getElementById('add-consist-num').value = t.consistNum || '';
            populateStationDropdowns('add-start-station');
            populateStationDropdowns('add-end-station');
            document.getElementById('add-start-station').value = t.startSt;
            document.getElementById('add-end-station').value = t.endSt;
            document.getElementById('addtrain-form-title').innerText = "列車編集";
            document.getElementById('btn-cancel-edit').classList.remove('hidden');
            switchTab('addtrain');
            updateStationSchedulePreview(t.schedule);
        }

        function resetAddTrainForm() {
            document.getElementById('edit-target-id').value = '';
            document.getElementById('addtrain-form-title').innerText = "列車追加";
            document.getElementById('btn-cancel-edit').classList.add('hidden');
            updateStationSchedulePreview();
        }

        function populateStationDropdowns(elementId) {
            const el = document.getElementById(elementId);
            if(el) el.innerHTML = allStationsMaster.map(st => `<option value="${st.id}">${st.id}. ${st.name}</option>`).join('');
        }

        function updateStationSchedulePreview(existingSched = null) {
            const startId = document.getElementById('add-start-station').value || "1";
            const endId = document.getElementById('add-end-station').value || "13";
            const preview = document.getElementById('station-schedule-preview');
            if(!preview) return;

            let sIdx = allStationsMaster.findIndex(s => s.id === startId);
            let eIdx = allStationsMaster.findIndex(s => s.id === endId);
            if(sIdx === -1 || eIdx === -1) return;
            const step = sIdx > eIdx ? -1 : 1;

            let html = `<table class="w-full text-left border-collapse"><tr class="text-indigo-300 border-b border-slate-800 text-[11px]"><th class="p-2">駅名</th><th class="p-2">着</th><th class="p-2">発</th></tr>`;
            let curr = sIdx, count = 0;
            while(true) {
                const st = allStationsMaster[curr];
                const ex = existingSched?.find(s => s.stId === st.id);
                html += `<tr><td class="p-2 font-bold text-slate-200">${st.id}. ${st.name}</td><td class="p-2"><input type="text" data-stid="${st.id}" class="sched-arr bg-slate-950 border border-slate-700 rounded px-1.5 py-1 w-16 sm:w-20 text-xs font-mono text-white" value="${ex ? ex.arr : (curr === sIdx ? '-' : '08:10')}"></td><td class="p-2"><input type="text" data-stid="${st.id}" class="sched-dep bg-slate-950 border border-slate-700 rounded px-1.5 py-1 w-16 sm:w-20 text-xs font-mono text-white" value="${ex ? ex.dep : (curr === eIdx ? '-' : '08:11')}"></td></tr>`;
                if(curr === eIdx) break;
                curr += step;
                count++;
            }
            preview.innerHTML = html + `</table>`;
        }

        function saveTrainData() {
            const editId = document.getElementById('edit-target-id').value;
            const trainNum = document.getElementById('add-train-num').value;
            const opNum = document.getElementById('add-op-num').value;
            const line = document.getElementById('add-line').value;
            const type = document.getElementById('add-train-type').value;
            const consistNum = document.getElementById('add-consist-num').value;
            const startSt = document.getElementById('add-start-station').value;
            const endSt = document.getElementById('add-end-station').value;

            const schedule = [];
            document.querySelectorAll('.sched-arr').forEach(el => {
                const stId = el.getAttribute('data-stid');
                const arr = el.value;
                const dep = document.querySelector(`.sched-dep[data-stid="${stId}"]`)?.value || '-';
                schedule.push({ stId, arr, dep });
            });

            if(editId) {
                const idx = appData.trains.findIndex(t => t.id == editId);
                if(idx !== -1) appData.trains[idx] = { id: Number(editId), trainNum, opNum, line, type, consistNum, startSt, endSt, schedule };
            } else {
                appData.trains.push({ id: Date.now(), trainNum, opNum, line, type, consistNum, startSt, endSt, schedule });
            }

            pushData();
            resetAddTrainForm();
            switchTab('traininfo');
        }

        function renderConsistMatrix() {
            const container = document.getElementById('consist-matrix-container');
            if(!container) return;
            const opList = Array.from(new Set((appData.trains || []).map(t => t.opNum).filter(Boolean))).sort();

            container.innerHTML = fullConsistsMaster.map(group => `
                <div class="bg-slate-800 p-4 rounded-xl border border-slate-700 space-y-3">
                    <h3 class="font-bold text-indigo-300 text-sm border-b border-slate-700 pb-2">${group.series}</h3>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-2 text-xs">
                        ${group.items.map(item => {
                            const currentAssigned = appData.consistAssignments[item.text] || "";
                            let optionsHtml = `<option value="">-- 未選択 --</option>`;
                            opList.forEach(op => {
                                const selected = (currentAssigned === op) ? "selected" : "";
                                optionsHtml += `<option value="${op}" ${selected}>${op}</option>`;
                            });
                            return `
                                <div class="bg-slate-900 p-2.5 rounded border border-slate-700 flex justify-between items-center gap-2">
                                    <span class="font-mono text-slate-200 font-bold">${item.text}</span>
                                    <select onchange="appData.consistAssignments['${item.text}']=this.value;pushData();" class="bg-slate-800 border border-slate-700 rounded px-2 py-1 text-xs text-indigo-300 font-mono outline-none focus:border-indigo-500">
                                        ${optionsHtml}
                                    </select>
                                </div>
                            `;
                        }).join('')}
                    </div>
                </div>
            `).join('');
        }

        function renderMatrixTimetable() {
            const container = document.getElementById('matrix-timetable-container');
            if(!container) return;
            const trains = appData.trains || [];
            if(trains.length === 0) { container.innerHTML = `<p class="text-slate-500 text-center py-8">データなし</p>`; return; }

            let html = `<table class="w-full border-collapse text-xs text-center"><thead><tr class="bg-slate-900 text-indigo-200 border-b border-slate-700"><th class="py-2 px-3 border-r border-slate-700 text-left sticky left-0 bg-slate-900 z-10 w-32 sm:w-36">駅名</th>`;
            trains.forEach(t => { html += `<th class="py-2 px-2 border-r border-slate-700 min-w-[70px]"><div class="font-bold text-indigo-300 font-mono text-xs">${t.trainNum}</div><div class="text-[9px] px-1 rounded ${typeColorMap[t.type] || ''}">${t.type}</div></th>`; });
            html += `</tr></thead><tbody class="divide-y divide-slate-800 text-slate-300">`;

            allStationsMaster.forEach(st => {
                html += `<tr><td class="py-1.5 px-3 border-r border-slate-700 text-left font-medium sticky left-0 bg-slate-800 z-10 w-32 sm:w-36 text-xs">${st.id}. ${st.name}</td>`;
                trains.forEach(t => {
                    const sched = t.schedule?.find(s => s.stId === st.id);
                    html += `<td class="py-1.5 px-2 border-r border-slate-800 font-mono text-[11px]">${sched ? (sched.dep !== '-' ? sched.dep : sched.arr) : '｜'}</td>`;
                });
                html += `</tr>`;
            });
            container.innerHTML = html + `</tbody></table>`;
        }

        function saveSettings() { appData.settings.fixedDay = document.getElementById('setting-fixed-day').value; pushData(); }
        function toggleTheme() { document.body.classList.toggle('dark'); }
        function showTrainDetail(trainId) {
            const t = appData.trains.find(item => item.id === trainId);
            if(!t) return;
            const modal = document.getElementById('train-detail-modal');
            const startName = allStationsMaster.find(s => s.id === t.startSt)?.name || t.startSt;
            const endName = allStationsMaster.find(s => s.id === t.endSt)?.name || t.endSt;
            document.getElementById('modal-content').innerHTML = `
                <div class="border-b border-slate-700 pb-2"><div class="text-xs text-indigo-400 font-mono">列車: ${t.trainNum} / 運用: ${t.opNum}</div><div class="text-base sm:text-lg font-bold text-white mt-1">${t.type} ${endName}行</div></div>
                <div class="text-xs text-slate-300 space-y-1"><p><strong>区間:</strong> ${startName} ➔ ${endName}</p><p><strong>編成:</strong> ${t.consistNum || '-'}</p><p><strong>状態:</strong> ${getTrainStatus(t)}</p></div>
                <div class="max-h-48 overflow-y-auto bg-slate-900 p-2.5 rounded-lg border border-slate-700 space-y-1">
                    ${t.schedule?.map(s => `<div class="flex justify-between text-xs"><span class="text-slate-300">${allStationsMaster.find(st=>st.id===s.stId)?.name || s.stId}</span><span class="font-mono text-indigo-300">着 ${s.arr} / 発 ${s.dep}</span></div>`).join('') || ''}
                </div>`;
            modal.classList.remove('hidden');
        }
        function closeModal() { document.getElementById('train-detail-modal').classList.add('hidden'); }
        function updateClock() {
            const now = new Date();
            const days = ['日','月','火','水','木','金','土'];
            const el = document.getElementById('live-datetime');
            if(el) el.innerText = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')}(${days[now.getDay()]}) ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}:${String(now.getSeconds()).padStart(2,'0')}`;
        }

        window.onload = function() {
            initFirebase();
            setInterval(updateClock, 1000);
            updateClock();
            setInterval(renderOperationTrack, 100);
        };
    </script>
</body>
</html>
