<!DOCTYPE html>
<html lang="ja" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>星空未来鉄道 (HFR) 総合指令・運行管理システム v5.0</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            cyber: {
              bg: '#0a0d18',
              card: '#121829',
              border: '#1f293d',
              accent: '#3b82f6',
              neon: '#60a5fa',
              gold: '#f59e0b',
              pink: '#ec4899',
              purple: '#8b5cf6',
              green: '#10b981'
            }
          },
          fontFamily: {
            mono: ['Consolas', 'Monaco', 'Courier New', 'monospace'],
            sans: ['Inter', 'Hiragino Kaku Gothic ProN', 'Meiryo', 'sans-serif']
          }
        }
      }
    }
  </script>
  <!-- FontAwesome & Lucide Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    /* Custom Scrollbar */
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #0a0d18; }
    ::-webkit-scrollbar-thumb { background: #1f293d; border-radius: 3px; }
    ::-webkit-scrollbar-thumb:hover { background: #3b82f6; }
    
    .glow-effect { box-shadow: 0 0 15px rgba(59, 130, 246, 0.25); }
    .glow-neon { box-shadow: 0 0 12px rgba(96, 165, 250, 0.4); }
    .glow-gold { box-shadow: 0 0 12px rgba(245, 158, 11, 0.4); }
    
    /* Railway diagram line */
    .vertical-track-line {
      background: linear-gradient(180deg, #3b82f6 0%, #8b5cf6 50%, #ec4899 100%);
    }

    /* Pulse animation for moving/stopped trains */
    @keyframes pulse-light {
      0%, 100% { opacity: 1; transform: scale(1); }
      50% { opacity: 0.7; transform: scale(1.05); }
    }
    .animate-pulse-train {
      animation: pulse-light 2s infinite ease-in-out;
    }
  </style>
</head>
<body class="bg-cyber-bg text-slate-200 font-sans min-h-screen flex flex-col antialiased selection:bg-blue-600 selection:text-white">

  <!-- TOP HEADER / NAV BAR -->
  <header class="bg-cyber-card/90 backdrop-blur border-b border-cyber-border sticky top-0 z-50 px-4 py-2.5 flex flex-wrap items-center justify-between gap-3">
    <div class="flex items-center gap-3">
      <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-blue-600 via-indigo-600 to-purple-600 flex items-center justify-center shadow-lg shadow-blue-500/20 ring-1 ring-white/20">
        <i class="fa-solid font-bold fa-train-subway text-xl text-white"></i>
      </div>
      <div>
        <div class="flex items-center gap-2">
          <h1 class="font-extrabold text-lg tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-indigo-300 to-purple-400">
            星空未来鉄道 <span class="text-xs px-1.5 py-0.5 rounded bg-blue-500/20 text-blue-300 border border-blue-500/30">HFR Sys v5.0</span>
          </h1>
        </div>
        <p class="text-xs text-slate-400">Hoshizora Future Railway Operations & Command System</p>
      </div>
    </div>

    <!-- Clock and Global Controls -->
    <div class="flex items-center gap-4 text-xs">
      <div class="bg-black/40 border border-slate-800 px-3 py-1.5 rounded-lg flex items-center gap-3 font-mono">
        <span class="flex items-center gap-1.5 text-slate-400">
          <i class="fa-regular fa-clock text-blue-400"></i>
          <span id="header-day-type" class="text-amber-400 font-sans font-semibold">平日ダイヤ</span>
        </span>
        <span id="system-clock" class="text-base font-bold text-blue-300 tracking-wider">08:00:00</span>
        <span id="speed-badge" class="px-1.5 py-0.5 rounded bg-blue-900/60 text-blue-300 text-[10px] border border-blue-700">1x</span>
      </div>

      <div class="hidden md:flex items-center gap-2 text-slate-400">
        <span class="flex items-center gap-1"><i class="fa-solid fa-satellite text-emerald-400 animate-pulse"></i> 衛星回線: 正常</span>
        <span class="text-slate-600">|</span>
        <button onclick="switchTab('settings')" class="hover:text-blue-400 transition-colors">
          <i class="fa-solid fa-gear"></i> 設定
        </button>
      </div>
    </div>
  </header>

  <!-- NAVIGATION TABS -->
  <nav class="bg-slate-950/80 border-b border-cyber-border px-4 overflow-x-auto">
    <div class="flex space-x-1 min-w-max py-2" id="nav-tabs">
      <button onclick="switchTab('company')" id="tab-btn-company" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-building text-blue-400"></i> 会社について
      </button>
      <button onclick="switchTab('timetable')" id="tab-btn-timetable" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-regular fa-clock text-amber-400"></i> 時刻表
      </button>
      <button onclick="switchTab('live-map')" id="tab-btn-live-map" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-route text-emerald-400"></i> 走行位置 <span class="w-2 h-2 rounded-full bg-emerald-400 animate-ping"></span>
      </button>
      <button onclick="switchTab('train-info')" id="tab-btn-train-info" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-list-check text-purple-400"></i> 列車情報
      </button>
      <button onclick="switchTab('add-train')" id="tab-btn-add-train" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-circle-plus text-pink-400"></i> 列車追加
      </button>
      <button onclick="switchTab('fleet')" id="tab-btn-fleet" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-train text-indigo-400"></i> 編成表
      </button>
      <button onclick="switchTab('diagram')" id="tab-btn-diagram" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-table text-cyan-400"></i> ダイヤ表
      </button>
      <button onclick="switchTab('settings')" id="tab-btn-settings" class="tab-btn px-3.5 py-1.5 rounded-lg text-xs font-medium transition-all flex items-center gap-2 text-slate-400 hover:text-white hover:bg-slate-800">
        <i class="fa-solid fa-sliders text-slate-400"></i> 設定
      </button>
    </div>
  </nav>

  <!-- MAIN CONTENT AREA -->
  <main class="flex-1 p-4 md:p-6 max-w-7xl w-full mx-auto">

    <!-- 1. 会社について TAB -->
    <section id="tab-company" class="tab-content hidden space-y-6">
      <div class="bg-cyber-card border border-cyber-border rounded-2xl p-6 relative overflow-hidden shadow-xl">
        <div class="absolute -right-10 -bottom-10 text-slate-800/30 text-9xl font-black pointer-events-none">HFR</div>
        <div class="flex items-center gap-4 mb-4">
          <div class="p-3 bg-blue-500/10 border border-blue-500/30 rounded-xl text-blue-400">
            <i class="fa-solid fa-rocket text-3xl"></i>
          </div>
          <div>
            <h2 class="text-2xl font-bold text-white">星空未来鉄道株式会社 (HFR)</h2>
            <p class="text-xs text-blue-400 font-mono">Hoshizora Future Railway Co., Ltd.</p>
          </div>
        </div>
        <p class="text-sm text-slate-300 leading-relaxed max-w-3xl">
          星空未来鉄道（HFR）は、都心エリアから新都心・宇宙港（スペースポート）を結ぶ近未来型都市超高速交通システムです。全線リニア・リニア誘導磁気架線と自動運行管理システム（AACS-v5）を備え、定時到達率99.999%と超快適な超高音速輸送力機能を実現しています。
        </p>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <!-- Main Line -->
        <div class="bg-cyber-card border border-blue-500/30 rounded-xl p-5 hover:border-blue-500 transition-all">
          <div class="flex items-center justify-between mb-3">
            <span class="px-2.5 py-1 rounded-full text-xs font-bold bg-blue-500/20 text-blue-400 border border-blue-500/40">HFR-01</span>
            <i class="fa-solid fa-bolt text-blue-400"></i>
          </div>
          <h3 class="text-lg font-bold text-white mb-1">星空本線 (Hoshizora Main Line)</h3>
          <p class="text-xs text-slate-400 mb-3">中央未来 ⇄ 宇宙港ターミナル (全10駅)</p>
          <div class="text-xs text-slate-300 space-y-1 bg-slate-900/60 p-2.5 rounded-lg">
            <div>最高速度: 240km/h</div>
            <div>主要種別: 宇宙特急, 快速, 普通</div>
          </div>
        </div>

        <!-- Luna Circle Line -->
        <div class="bg-cyber-card border border-purple-500/30 rounded-xl p-5 hover:border-purple-500 transition-all">
          <div class="flex items-center justify-between mb-3">
            <span class="px-2.5 py-1 rounded-full text-xs font-bold bg-purple-500/20 text-purple-400 border border-purple-500/40">HFR-02</span>
            <i class="fa-solid fa-arrows-spin text-purple-400"></i>
          </div>
          <h3 class="text-lg font-bold text-white mb-1">ルナサークル線 (Luna Circle Line)</h3>
          <p class="text-xs text-slate-400 mb-3">月見ヶ丘 ⇄ ルナタワー (環状線・全8駅)</p>
          <div class="text-xs text-slate-300 space-y-1 bg-slate-900/60 p-2.5 rounded-lg">
            <div>最高速度: 160km/h</div>
            <div>主要種別: 環状急行, 各駅停車</div>
          </div>
        </div>

        <!-- Galaxy Skyline Line -->
        <div class="bg-cyber-card border border-pink-500/30 rounded-xl p-5 hover:border-pink-500 transition-all">
          <div class="flex items-center justify-between mb-3">
            <span class="px-2.5 py-1 rounded-full text-xs font-bold bg-pink-500/20 text-pink-400 border border-pink-500/40">HFR-03</span>
            <i class="fa-solid fa-plane-departure text-pink-400"></i>
          </div>
          <h3 class="text-lg font-bold text-white mb-1">銀河スカイライン (Galaxy Skyline)</h3>
          <p class="text-xs text-slate-400 mb-3">銀河ドーム ⇄ 軌道エレベータ前 (高架線・全6駅)</p>
          <div class="text-xs text-slate-300 space-y-1 bg-slate-900/60 p-2.5 rounded-lg">
            <div>最高速度: 320km/h</div>
            <div>主要種別: ハイパーライナー, 直行</div>
          </div>
        </div>
      </div>
    </section>

    <!-- 2. 時刻表 TAB -->
    <section id="tab-timetable" class="tab-content hidden space-y-4">
      <!-- Filter Bar -->
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 flex flex-wrap items-center justify-between gap-4">
        <div class="flex flex-wrap items-center gap-3">
          <!-- Station Selection -->
          <div>
            <label class="block text-[10px] text-slate-400 mb-1">対象駅</label>
            <select id="tt-station-select" onchange="renderTimetable()" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
              <!-- Dynamically Populated -->
            </select>
          </div>

          <!-- Line Selection -->
          <div>
            <label class="block text-[10px] text-slate-400 mb-1">路線</label>
            <select id="tt-line-select" onchange="renderTimetable()" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
              <option value="ALL">全路線</option>
              <option value="星空本線">星空本線</option>
              <option value="ルナサークル線">ルナサークル線</option>
              <option value="銀河スカイライン">銀河スカイライン</option>
            </select>
          </div>

          <!-- Day Type Select -->
          <div>
            <label class="block text-[10px] text-slate-400 mb-1">ダイヤ種別</label>
            <div class="inline-flex rounded-lg bg-slate-900 p-0.5 border border-slate-800">
              <button onclick="setTimetableDay('平日')" id="tt-day-weekday" class="px-2.5 py-1 text-xs rounded-md bg-blue-600 text-white font-medium">平日</button>
              <button onclick="setTimetableDay('土休日')" id="tt-day-weekend" class="px-2.5 py-1 text-xs rounded-md text-slate-400 hover:text-white">土休日</button>
              <button onclick="setTimetableDay('特別')" id="tt-day-event" class="px-2.5 py-1 text-xs rounded-md text-slate-400 hover:text-white">イベント</button>
            </div>
          </div>

          <!-- Direction Toggle -->
          <div>
            <label class="block text-[10px] text-slate-400 mb-1">方向</label>
            <div class="inline-flex rounded-lg bg-slate-900 p-0.5 border border-slate-800">
              <button onclick="setTimetableDir('下り')" id="tt-dir-down" class="px-3 py-1 text-xs rounded-md bg-indigo-600 text-white font-medium">下り (郊外/宇宙港方面)</button>
              <button onclick="setTimetableDir('上り')" id="tt-dir-up" class="px-3 py-1 text-xs rounded-md text-slate-400 hover:text-white">上り (都心方面)</button>
            </div>
          </div>
        </div>

        <div class="text-xs text-slate-400">
          <i class="fa-solid fa-circle-info text-blue-400 mr-1"></i> 発車時刻をクリックすると全停止駅時刻を確認できます
        </div>
      </div>

      <!-- Station Paper Style Timetable Grid -->
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 overflow-x-auto shadow-xl">
        <div id="timetable-paper-grid" class="space-y-3 min-w-[700px]">
          <!-- Rendered via JS -->
        </div>
      </div>
    </section>

    <!-- 3. 走行位置 (LIVE LOCATION) TAB - えれサイト風 縦型表示 -->
    <section id="tab-live-map" class="tab-content hidden space-y-4">
      <!-- Location Header Bar -->
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 flex flex-wrap items-center justify-between gap-4">
        <div class="flex items-center gap-3">
          <div class="flex items-center gap-2">
            <span class="w-3 h-3 rounded-full bg-emerald-500 animate-ping"></span>
            <h2 class="text-lg font-bold text-white">リアルタイム列車走行位置</h2>
          </div>
          <span class="text-xs text-slate-400 bg-slate-900 px-2.5 py-1 rounded-md border border-slate-800">
            更新間隔: リアルタイム連動
          </span>
        </div>

        <!-- Controls: Line Switcher & Display Mode Toggle -->
        <div class="flex items-center gap-4">
          <!-- Line Selector -->
          <div class="flex items-center gap-2">
            <span class="text-xs text-slate-400">表示路線:</span>
            <select id="live-line-select" onchange="renderLiveMap()" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
              <option value="星空本線">星空本線 (中央未来〜宇宙港)</option>
              <option value="ルナサークル線">ルナサークル線 (月見ヶ丘〜ルナタワー)</option>
              <option value="銀河スカイライン">銀河スカイライン (銀河ドーム〜軌道エレベータ)</option>
            </select>
          </div>

          <!-- Display Mode Toggle: Icon vs Card Details -->
          <div class="flex items-center bg-slate-900 rounded-lg p-1 border border-slate-800 text-xs">
            <button id="live-mode-icon-btn" onclick="setLiveDisplayMode('icon')" class="px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all">
              <i class="fa-solid fa-train text-xs mr-1"></i> 列車アイコン
            </button>
            <button id="live-mode-card-btn" onclick="setLiveDisplayMode('card')" class="px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all">
              <i class="fa-solid fa-address-card text-xs mr-1"></i> 列車情報
            </button>
          </div>
        </div>
      </div>

      <!-- ERESITE-STYLE VERTICAL MONITOR -->
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-6 shadow-2xl relative overflow-hidden">
        
        <!-- Center Axis Legend / Title Header -->
        <div class="grid grid-cols-11 items-center mb-6 text-xs font-bold border-b border-slate-800 pb-3">
          <div class="col-span-5 text-right pr-6 text-blue-400 flex items-center justify-end gap-2">
            <i class="fa-solid fa-arrow-up"></i>
            <span>【上り】 都心・起点方面</span>
          </div>
          <div class="col-span-1 text-center text-slate-400 font-mono tracking-wider">
            駅 / 駅間
          </div>
          <div class="col-span-5 text-left pl-6 text-purple-400 flex items-center justify-start gap-2">
            <span>【下り】 郊外・終点方面</span>
            <i class="fa-solid fa-arrow-down"></i>
          </div>
        </div>

        <!-- Dynamic Vertical Shaft Diagram Container -->
        <div id="vertical-eresite-container" class="relative py-2 max-w-4xl mx-auto">
          <!-- Rendered via JS -->
        </div>

      </div>
    </section>

    <!-- 4. 列車情報 (TRAIN INFO / MANAGEMENT) TAB -->
    <section id="tab-train-info" class="tab-content hidden space-y-4">
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 flex flex-wrap items-center justify-between gap-4">
        <div class="flex items-center gap-3">
          <h2 class="text-base font-bold text-white">ダイヤ登録済み列車一覧</h2>
          <span id="train-count-badge" class="px-2 py-0.5 rounded-full text-xs bg-blue-500/20 text-blue-400 border border-blue-500/30">0本</span>
        </div>

        <!-- Mode Toggle: Train No Table vs Operation ID Timeline -->
        <div class="flex items-center bg-slate-900 rounded-lg p-1 border border-slate-800 text-xs">
          <button id="info-mode-num-btn" onclick="setTrainInfoMode('number')" class="px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all">
            <i class="fa-solid fa-list-ol mr-1"></i> 列車番号順 (テーブル)
          </button>
          <button id="info-mode-op-btn" onclick="setTrainInfoMode('operation')" class="px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all">
            <i class="fa-solid fa-timeline mr-1"></i> 運用番号順 (タイムライン)
          </button>
        </div>
      </div>

      <!-- Mode 1 Container: Table View -->
      <div id="train-info-table-container" class="bg-cyber-card border border-cyber-border rounded-xl overflow-x-auto shadow-xl">
        <table class="w-full text-left border-collapse text-xs">
          <thead>
            <tr class="bg-slate-900/80 text-slate-400 border-b border-slate-800 font-mono">
              <th class="p-3">列車番号</th>
              <th class="p-3">運用番号</th>
              <th class="p-3">種別</th>
              <th class="p-3">路線</th>
              <th class="p-3">行先 (始発 ➔ 終着)</th>
              <th class="p-3">形式 / 両数</th>
              <th class="p-3">状態</th>
              <th class="p-3 text-right">操作 (通常/編集/複製/削除)</th>
            </tr>
          </thead>
          <tbody id="train-info-tbody" class="divide-y divide-slate-800/60 font-mono">
            <!-- Rendered via JS -->
          </tbody>
        </table>
      </div>

      <!-- Mode 2 Container: Operation Timeline View -->
      <div id="train-info-op-container" class="hidden space-y-4">
        <!-- Rendered via JS -->
      </div>
    </section>

    <!-- 5. 列車追加 (ADD TRAIN) TAB -->
    <section id="tab-add-train" class="tab-content hidden space-y-6">
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-6 shadow-xl max-w-4xl mx-auto">
        <h2 class="text-lg font-bold text-white mb-4 flex items-center gap-2 border-b border-slate-800 pb-3">
          <i class="fa-solid fa-circle-plus text-pink-400"></i> 新規列車 ダイヤ・編成設定
        </h2>

        <form id="add-train-form" onsubmit="handleAddTrainSubmit(event)" class="space-y-6">
          <!-- Basic Info Grid -->
          <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-4">
            <div>
              <label class="block text-xs text-slate-400 mb-1">列車番号 <span class="text-pink-400">*</span></label>
              <input type="text" id="form-train-no" required placeholder="例: 1021M" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500 font-mono">
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">運用番号 <span class="text-pink-400">*</span></label>
              <input type="text" id="form-op-no" required placeholder="例: A01" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500 font-mono">
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">所属路線</label>
              <select id="form-line" onchange="updateFormStationOptions()" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
                <option value="星空本線">星空本線</option>
                <option value="ルナサークル線">ルナサークル線</option>
                <option value="銀河スカイライン">銀河スカイライン</option>
              </select>
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">列車種別</label>
              <select id="form-type" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
                <option value="普通">普通 (Local)</option>
                <option value="快速">快速 (Rapid)</option>
                <option value="宇宙特急">宇宙特急 (Limited Express)</option>
                <option value="ハイパーライナー">ハイパーライナー (Hyperliner)</option>
              </select>
            </div>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
            <div>
              <label class="block text-xs text-slate-400 mb-1">始発駅</label>
              <select id="form-origin" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
                <!-- JS populated -->
              </select>
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">終着駅</label>
              <select id="form-terminus" class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200 focus:outline-none focus:border-blue-500">
                <!-- JS populated -->
              </select>
            </div>
            <div>
              <label class="block text-xs text-slate-400 mb-1">車両形式 / 両数</label>
              <div class="grid grid-cols-2 gap-2">
                <select id="form-series" class="bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200">
                  <option value="H1000系">H1000系</option>
                  <option value="L2000系">L2000系</option>
                  <option value="G7000系">G7000系</option>
                </select>
                <select id="form-cars" class="bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-slate-200">
                  <option value="4両">4両</option>
                  <option value="6両" selected>6両</option>
                  <option value="8両">8両</option>
                  <option value="10両">10両</option>
                </select>
              </div>
            </div>
          </div>

          <!-- ADVANCED CHECKBOX OPTIONS -->
          <div class="border-t border-slate-800 pt-4 space-y-4">
            <h3 class="text-xs font-bold text-slate-300 uppercase tracking-wider">特殊運用・編成設定</h3>

            <!-- 1. 種別変更 -->
            <div class="bg-slate-900/70 border border-slate-800 rounded-xl p-4 space-y-3">
              <label class="flex items-center gap-2 cursor-pointer">
                <input type="checkbox" id="chk-type-change" onchange="toggleFormSection('sec-type-change', this.checked)" class="rounded bg-slate-800 border-slate-700 text-blue-600 focus:ring-0">
                <span class="text-xs font-bold text-blue-400">途中で「種別変更」を行う</span>
              </label>
              <div id="sec-type-change" class="hidden grid grid-cols-1 sm:grid-cols-3 gap-3 pl-6 pt-2 border-l-2 border-blue-500/40">
                <div>
                  <label class="block text-[10px] text-slate-400 mb-1">変更実施駅</label>
                  <select id="form-tc-station" class="w-full bg-slate-950 border border-slate-700 rounded p-2 text-xs text-slate-200">
                    <!-- JS populated -->
                  </select>
                </div>
                <div>
                  <label class="block text-[10px] text-slate-400 mb-1">変更後の新種別</label>
                  <select id="form-tc-new-type" class="w-full bg-slate-950 border border-slate-700 rounded p-2 text-xs text-slate-200">
                    <option value="普通">普通</option>
                    <option value="快速">快速</option>
                    <option value="宇宙特急">宇宙特急</option>
                  </select>
                </div>
                <div>
                  <label class="block text-[10px] text-slate-400 mb-1">変更後 列車番号</label>
                  <input type="text" id="form-tc-new-no" placeholder="例: 2021M" class="w-full bg-slate-950 border border-slate-700 rounded p-2 text-xs text-slate-200 font-mono">
                </div>
              </div>
            </div>

            <!-- 2. 連結編成 -->
            <div class="bg-slate-900/70 border border-slate-800 rounded-xl p-4 space-y-3">
              <label class="flex items-center gap-2 cursor-pointer">
                <input type="checkbox" id="chk-coupling" onchange="toggleFormSection('sec-coupling', this.checked)" class="rounded bg-slate-800 border-slate-700 text-purple-600 focus:ring-0">
                <span class="text-xs font-bold text-purple-400">複数編成の「連結編成」として登録（前部・後部編成）</span>
              </label>
              <div id="sec-coupling" class="hidden grid grid-cols-1 sm:grid-cols-2 gap-4 pl-6 pt-2 border-l-2 border-purple-500/40">
                <div class="bg-slate-950 p-3 rounded-lg border border-slate-800">
                  <span class="text-[10px] font-bold text-purple-300 block mb-2">前部編成 (1〜4号車)</span>
                  <input type="text" id="form-c-front" placeholder="前部形式 例: H1000形 4両" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-xs text-slate-200 font-mono">
                </div>
                <div class="bg-slate-950 p-3 rounded-lg border border-slate-800">
                  <span class="text-[10px] font-bold text-pink-300 block mb-2">後部編成 (5〜8号車)</span>
                  <input type="text" id="form-c-rear" placeholder="後部形式 例: L2000形 4両" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-xs text-slate-200 font-mono">
                </div>
              </div>
            </div>

            <!-- 3. 連結と解結作業 -->
            <div class="bg-slate-900/70 border border-slate-800 rounded-xl p-4 space-y-3">
              <label class="flex items-center gap-2 cursor-pointer">
                <input type="checkbox" id="chk-work" onchange="toggleFormSection('sec-work', this.checked)" class="rounded bg-slate-800 border-slate-700 text-emerald-600 focus:ring-0">
                <span class="text-xs font-bold text-emerald-400">途中で「連結作業 / 解結作業」を行う</span>
              </label>
              <div id="sec-work" class="hidden space-y-3 pl-6 pt-2 border-l-2 border-emerald-500/40">
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                  <div>
                    <label class="block text-[10px] text-slate-400 mb-1">作業実施駅</label>
                    <select id="form-w-station" class="w-full bg-slate-950 border border-slate-700 rounded p-2 text-xs text-slate-200">
                      <!-- JS populated -->
                    </select>
                  </div>
                  <div>
                    <label class="block text-[10px] text-slate-400 mb-1">作業種別</label>
                    <select id="form-w-type" class="w-full bg-slate-950 border border-slate-700 rounded p-2 text-xs text-slate-200">
                      <option value="解結">解結作業 (切り離し)</option>
                      <option value="連結">連結作業 (増結)</option>
                    </select>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="flex justify-end gap-3 pt-2">
            <button type="reset" class="px-4 py-2 rounded-lg bg-slate-800 text-slate-300 text-xs hover:bg-slate-700 transition-colors">リセット</button>
            <button type="submit" class="px-6 py-2 rounded-lg bg-gradient-to-r from-blue-600 to-indigo-600 text-white text-xs font-bold hover:from-blue-500 hover:to-indigo-500 transition-all shadow-lg shadow-blue-500/20">
              <i class="fa-solid fa-plus mr-1"></i> 列車をダイヤに追加・登録
            </button>
          </div>
        </form>
      </div>
    </section>

    <!-- 6. 編成表 (FLEET / FORMATIONS) TAB -->
    <section id="tab-fleet" class="tab-content hidden space-y-6">
      <div class="flex items-center justify-between">
        <h2 class="text-lg font-bold text-white flex items-center gap-2">
          <i class="fa-solid fa-train text-indigo-400"></i> HFR 車両運用・編成一覧
        </h2>
        <span class="text-xs text-slate-400">全3系列・次世代超電導リニア車両</span>
      </div>

      <div class="grid grid-cols-1 gap-6" id="fleet-cards-container">
        <!-- Rendered via JS -->
      </div>
    </section>

    <!-- 7. ダイヤ表 (DIAGRAM / PAPER TIMETABLE) TAB -->
    <section id="tab-diagram" class="tab-content hidden space-y-4">
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 flex flex-wrap items-center justify-between gap-4">
        <div class="flex items-center gap-3">
          <h2 class="text-base font-bold text-white flex items-center gap-2">
            <i class="fa-solid fa-table text-cyan-400"></i> 本格紙時刻表風 ダイヤ縦型マトリックス表
          </h2>
        </div>
        <div class="flex items-center gap-3 text-xs">
          <select id="diagram-line-select" onchange="renderPaperDiagram()" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-slate-200">
            <option value="星空本線">星空本線</option>
            <option value="ルナサークル線">ルナサークル線</option>
            <option value="銀河スカイライン">銀河スカイライン</option>
          </select>
          <select id="diagram-dir-select" onchange="renderPaperDiagram()" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-slate-200">
            <option value="下り">下り (郊外方面)</option>
            <option value="上り">上り (都心方面)</option>
          </select>
        </div>
      </div>

      <!-- Authentic Paper Timetable Grid Container -->
      <div class="bg-slate-950 border border-slate-800 rounded-xl p-4 overflow-x-auto shadow-2xl">
        <div id="paper-diagram-matrix" class="min-w-max text-xs">
          <!-- Rendered via JS -->
        </div>
      </div>
    </section>

    <!-- 8. 設定 (SETTINGS) TAB -->
    <section id="tab-settings" class="tab-content hidden space-y-6">
      <div class="bg-cyber-card border border-cyber-border rounded-xl p-6 shadow-xl max-w-3xl mx-auto space-y-6">
        <h2 class="text-lg font-bold text-white mb-2 flex items-center gap-2 border-b border-slate-800 pb-3">
          <i class="fa-solid fa-sliders text-slate-400"></i> システム設定 & 運行環境調整
        </h2>

        <!-- Setting 1: Global Day Lock -->
        <div class="space-y-2">
          <label class="block text-sm font-bold text-slate-200">1. サイト全運行日ダイヤの固定</label>
          <p class="text-xs text-slate-400">システム全体に適用する運行日ダイヤを統一固定します。</p>
          <div class="grid grid-cols-2 sm:grid-cols-4 gap-2 pt-1">
            <button onclick="setGlobalDayLock('AUTO')" id="lock-btn-AUTO" class="setting-day-btn px-3 py-2 rounded-lg bg-blue-600 text-white text-xs font-medium border border-blue-500">自動 (曜日連動)</button>
            <button onclick="setGlobalDayLock('平日')" id="lock-btn-平日" class="setting-day-btn px-3 py-2 rounded-lg bg-slate-900 text-slate-400 text-xs hover:bg-slate-800 border border-slate-800">平日ダイヤ固定</button>
            <button onclick="setGlobalDayLock('土休日')" id="lock-btn-土休日" class="setting-day-btn px-3 py-2 rounded-lg bg-slate-900 text-slate-400 text-xs hover:bg-slate-800 border border-slate-800">土休日ダイヤ固定</button>
            <button onclick="setGlobalDayLock('特別')" id="lock-btn-特別" class="setting-day-btn px-3 py-2 rounded-lg bg-slate-900 text-slate-400 text-xs hover:bg-slate-800 border border-slate-800">特別ダイヤ固定</button>
          </div>
        </div>

        <!-- Setting 2: Notification & Audio -->
        <div class="space-y-2 border-t border-slate-800 pt-4">
          <div class="flex items-center justify-between">
            <div>
              <label class="block text-sm font-bold text-slate-200">2. 運行アナウンス & アラート通知</label>
              <p class="text-xs text-slate-400">発車時・遅延発生時の効果音および通知ポップアップ</p>
            </div>
            <label class="relative inline-flex items-center cursor-pointer">
              <input type="checkbox" id="setting-notify-toggle" checked class="sr-only peer">
              <div class="w-11 h-6 bg-slate-800 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-slate-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
            </label>
          </div>
        </div>

        <!-- Setting 3: System Clock Speed Multiplier -->
        <div class="space-y-2 border-t border-slate-800 pt-4">
          <label class="block text-sm font-bold text-slate-200">3. システム模擬時間の倍速設定</label>
          <p class="text-xs text-slate-400">走行位置アニメーションや時刻表進行のシミュレーション速度</p>
          <div class="flex items-center gap-3 pt-1">
            <button onclick="setClockSpeed(1)" id="speed-btn-1" class="speed-btn px-3 py-1.5 rounded-lg bg-blue-600 text-white text-xs font-mono font-bold">1x (等倍)</button>
            <button onclick="setClockSpeed(5)" id="speed-btn-5" class="speed-btn px-3 py-1.5 rounded-lg bg-slate-900 text-slate-400 text-xs font-mono hover:bg-slate-800">5x (高速)</button>
            <button onclick="setClockSpeed(10)" id="speed-btn-10" class="speed-btn px-3 py-1.5 rounded-lg bg-slate-900 text-slate-400 text-xs font-mono hover:bg-slate-800">10x (超高速)</button>
            <button onclick="setClockSpeed(60)" id="speed-btn-60" class="speed-btn px-3 py-1.5 rounded-lg bg-slate-900 text-slate-400 text-xs font-mono hover:bg-slate-800">60x (1秒=1分)</button>
          </div>
        </div>

        <!-- Setting 4: Manual Custom Clock Set -->
        <div class="space-y-2 border-t border-slate-800 pt-4">
          <label class="block text-sm font-bold text-slate-200">4. 仮想システム時刻の手動変更</label>
          <div class="flex items-center gap-3">
            <input type="time" id="manual-time-input" value="08:00" class="bg-slate-900 border border-slate-700 rounded-lg px-3 py-1.5 text-xs text-white font-mono">
            <button onclick="applyManualTime()" class="px-4 py-1.5 rounded-lg bg-indigo-600 text-white text-xs font-bold hover:bg-indigo-500">時刻変更適用</button>
            <button onclick="resetToRealTime()" class="px-4 py-1.5 rounded-lg bg-slate-800 text-slate-300 text-xs hover:bg-slate-700">現在時刻に同期</button>
          </div>
        </div>

      </div>
    </section>

  </main>

  <!-- EDIT TRAIN MODAL -->
  <div id="edit-train-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-cyber-card border border-blue-500/40 rounded-2xl p-6 max-w-xl w-full space-y-4 shadow-2xl relative">
      <button onclick="closeEditModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>
      <h3 class="text-base font-bold text-white flex items-center gap-2">
        <i class="fa-solid fa-pen-to-square text-blue-400"></i> 列車データ 編集
      </h3>
      <div class="space-y-3 text-xs">
        <input type="hidden" id="edit-train-id">
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="block text-slate-400 mb-1">列車番号</label>
            <input type="text" id="edit-train-no" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white font-mono">
          </div>
          <div>
            <label class="block text-slate-400 mb-1">運用番号</label>
            <input type="text" id="edit-op-no" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white font-mono">
          </div>
        </div>
        <div class="grid grid-cols-2 gap-3">
          <div>
            <label class="block text-slate-400 mb-1">種別</label>
            <select id="edit-type" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
              <option value="普通">普通</option>
              <option value="快速">快速</option>
              <option value="宇宙特急">宇宙特急</option>
              <option value="ハイパーライナー">ハイパーライナー</option>
            </select>
          </div>
          <div>
            <label class="block text-slate-400 mb-1">運行状態</label>
            <select id="edit-status" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
              <option value="定時">定時運行</option>
              <option value="遅延">遅延 (+5分)</option>
              <option value="車両点検">車両点検</option>
            </select>
          </div>
        </div>
      </div>
      <div class="flex justify-end gap-2 pt-2">
        <button onclick="closeEditModal()" class="px-4 py-2 rounded bg-slate-800 text-slate-300 text-xs">キャンセル</button>
        <button onclick="saveTrainEdit()" class="px-5 py-2 rounded bg-blue-600 text-white text-xs font-bold hover:bg-blue-500">保存更新</button>
      </div>
    </div>
  </div>

  <!-- TRAIN DETAIL HOLOGRAPHIC POPUP MODAL -->
  <div id="detail-modal" class="fixed inset-0 bg-black/70 backdrop-blur-sm z-50 hidden flex items-center justify-center p-4">
    <div class="bg-cyber-card border border-indigo-500/50 rounded-2xl p-6 max-w-lg w-full space-y-4 shadow-2xl relative">
      <button onclick="closeDetailModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>
      
      <div id="detail-modal-content">
        <!-- Rendered via JS -->
      </div>
    </div>
  </div>

  <!-- JAVASCRIPT LOGIC STORE -->
  <script>
    // --- STATIONS DATA ---
    const STATIONS_DATA = {
      "星空本線": [
        { name: "中央未来", code: "H01", expressStop: true, rapidStop: true },
        { name: "新光台", code: "H02", expressStop: false, rapidStop: true },
        { name: "星空ドーム", code: "H03", expressStop: true, rapidStop: true },
        { name: "緑ヶ丘リニア", code: "H04", expressStop: false, rapidStop: false },
        { name: "サイバーポート", code: "H05", expressStop: true, rapidStop: true },
        { name: "天の川中央", code: "H06", expressStop: false, rapidStop: true },
        { name: "未来学園前", code: "H07", expressStop: false, rapidStop: false },
        { name: "フューチャーヒルズ", code: "H08", expressStop: true, rapidStop: true },
        { name: "光の森", code: "H09", expressStop: false, rapidStop: false },
        { name: "宇宙港ターミナル", code: "H10", expressStop: true, rapidStop: true }
      ],
      "ルナサークル線": [
        { name: "月見ヶ丘", code: "L01", expressStop: true, rapidStop: true },
        { name: "月面フロント", code: "L02", expressStop: true, rapidStop: true },
        { name: "クレーターテラス", code: "L03", expressStop: false, rapidStop: true },
        { name: "静寂の海", code: "L04", expressStop: true, rapidStop: true },
        { name: "銀河モール", code: "L05", expressStop: false, rapidStop: false },
        { name: "ルナパーク", code: "L06", expressStop: true, rapidStop: true },
        { name: "コスモセンター", code: "L07", expressStop: false, rapidStop: true },
        { name: "ルナタワー", code: "L08", expressStop: true, rapidStop: true }
      ],
      "銀河スカイライン": [
        { name: "銀河ドーム", code: "G01", expressStop: true, rapidStop: true },
        { name: "エアロシティ", code: "G02", expressStop: true, rapidStop: true },
        { name: "スカイハブ", code: "G03", expressStop: true, rapidStop: true },
        { name: "成層圏ステーション", code: "G04", expressStop: true, rapidStop: true },
        { name: "無重力リゾート", code: "G05", expressStop: false, rapidStop: true },
        { name: "軌道エレベータ前", code: "G06", expressStop: true, rapidStop: true }
      ]
    };

    // --- INITIAL TRAINS DATA STORE ---
    let trainsData = [
      {
        id: "tr-101",
        trainNo: "1011M",
        operationNo: "A01",
        line: "星空本線",
        type: "宇宙特急",
        direction: "下り",
        origin: "中央未来",
        terminus: "宇宙港ターミナル",
        series: "H1000系",
        cars: "8両",
        status: "定時",
        dayType: "平日",
        startTime: "08:00",
        endTime: "08:28",
        stops: [
          { station: "中央未来", dep: "08:00", arr: "08:00" },
          { station: "星空ドーム", dep: "08:07", arr: "08:06" },
          { station: "サイバーポート", dep: "08:14", arr: "08:13" },
          { station: "フューチャーヒルズ", dep: "08:21", arr: "08:20" },
          { station: "宇宙港ターミナル", dep: "08:28", arr: "08:28" }
        ],
        typeChange: null,
        coupling: { front: "H1000形 4両", rear: "L2000形 4両" },
        work: null
      },
      {
        id: "tr-102",
        trainNo: "1012M",
        operationNo: "A01",
        line: "星空本線",
        type: "宇宙特急",
        direction: "上り",
        origin: "宇宙港ターミナル",
        terminus: "中央未来",
        series: "H1000系",
        cars: "8両",
        status: "定時",
        dayType: "平日",
        startTime: "08:10",
        endTime: "08:38",
        stops: [
          { station: "宇宙港ターミナル", dep: "08:10", arr: "08:10" },
          { station: "フューチャーヒルズ", dep: "08:17", arr: "08:16" },
          { station: "サイバーポート", dep: "08:24", arr: "08:23" },
          { station: "星空ドーム", dep: "08:31", arr: "08:30" },
          { station: "中央未来", dep: "08:38", arr: "08:38" }
        ],
        typeChange: null,
        coupling: null,
        work: null
      },
      {
        id: "tr-201",
        trainNo: "2015K",
        operationNo: "B03",
        line: "星空本線",
        type: "快速",
        direction: "下り",
        origin: "中央未来",
        terminus: "宇宙港ターミナル",
        series: "L2000系",
        cars: "6両",
        status: "定時",
        dayType: "平日",
        startTime: "08:05",
        endTime: "08:42",
        stops: [
          { station: "中央未来", dep: "08:05", arr: "08:05" },
          { station: "新光台", dep: "08:09", arr: "08:08" },
          { station: "星空ドーム", dep: "08:14", arr: "08:13" },
          { station: "サイバーポート", dep: "08:21", arr: "08:20" },
          { station: "天の川中央", dep: "08:27", arr: "08:26" },
          { station: "フューチャーヒルズ", dep: "08:34", arr: "08:33" },
          { station: "宇宙港ターミナル", dep: "08:42", arr: "08:41" }
        ],
        typeChange: null,
        coupling: null,
        work: null
      },
      {
        id: "tr-301",
        trainNo: "501C",
        operationNo: "C12",
        line: "星空本線",
        type: "普通",
        direction: "下り",
        origin: "中央未来",
        terminus: "宇宙港ターミナル",
        series: "H1000系",
        cars: "6両",
        status: "定時",
        dayType: "平日",
        startTime: "07:55",
        endTime: "08:45",
        stops: [
          { station: "中央未来", dep: "07:55", arr: "07:55" },
          { station: "新光台", dep: "08:00", arr: "07:59" },
          { station: "星空ドーム", dep: "08:05", arr: "08:04" },
          { station: "緑ヶ丘リニア", dep: "08:10", arr: "08:09" },
          { station: "サイバーポート", dep: "08:16", arr: "08:15" },
          { station: "天の川中央", dep: "08:22", arr: "08:21" },
          { station: "未来学園前", dep: "08:27", arr: "08:26" },
          { station: "フューチャーヒルズ", dep: "08:33", arr: "08:32" },
          { station: "光の森", dep: "08:39", arr: "08:38" },
          { station: "宇宙港ターミナル", dep: "08:45", arr: "08:44" }
        ],
        typeChange: null,
        coupling: null,
        work: null
      },
      {
        id: "tr-401",
        trainNo: "701S",
        operationNo: "G05",
        line: "銀河スカイライン",
        type: "ハイパーライナー",
        direction: "下り",
        origin: "銀河ドーム",
        terminus: "軌道エレベータ前",
        series: "G7000系",
        cars: "10両",
        status: "定時",
        dayType: "平日",
        startTime: "08:02",
        endTime: "08:22",
        stops: [
          { station: "銀河ドーム", dep: "08:02", arr: "08:02" },
          { station: "エアロシティ", dep: "08:06", arr: "08:05" },
          { station: "スカイハブ", dep: "08:11", arr: "08:10" },
          { station: "成層圏ステーション", dep: "08:16", arr: "08:15" },
          { station: "軌道エレベータ前", dep: "08:22", arr: "08:21" }
        ],
        typeChange: null,
        coupling: null,
        work: null
      }
    ];

    // FLEET FORMATION DATA
    const FLEET_DATA = [
      {
        series: "H1000系 次世代超電導リニア",
        badgeColor: "bg-blue-500/20 text-blue-400 border-blue-500/40",
        desc: "HFRの基幹を担う標準型リニア車両。高加速性能と静粛性を兼ね備える。",
        totalSets: "14編成 (H101〜H114)",
        carsCount: "6両 / 8両",
        carsList: [
          { carNo: 1, type: "制御電動車", icon: "fa-rocket", spec: "展望サロン・自動運転ユニット" },
          { carNo: 2, type: "中間客車", icon: "fa-wifi", spec: "全席高速Wi-Fi・USB-C給電" },
          { carNo: 3, type: "カフェテラス車", icon: "fa-mug-hot", spec: "無人宇宙バーラウンジ" },
          { carNo: 4, type: "中間客車", icon: "fa-chair", spec: "ユニバーサルシート" },
          { carNo: 5, type: "サイレント車", icon: "fa-volume-xmark", spec: "防音遮音カプセル" },
          { carNo: 6, type: "制御電動車", icon: "fa-shield-halved", spec: "補助非常用バッテリ搭載" }
        ]
      },
      {
        series: "L2000系 環状スマートライナー",
        badgeColor: "bg-purple-500/20 text-purple-400 border-purple-500/40",
        desc: "ルナサークル線および通勤快速向け。多扉・広々としたAI空調車内。",
        totalSets: "10編成 (L201〜L210)",
        carsCount: "4両 / 6両",
        carsList: [
          { carNo: 1, type: "制御車", icon: "fa-bolt", spec: "高出力磁気モーター" },
          { carNo: 2, type: "中間車", icon: "fa-users", spec: "ワイドドア・AI混雑計測" },
          { carNo: 3, type: "中間車", icon: "fa-suitcase", spec: "大型荷物・車椅子スペース" },
          { carNo: 4, type: "制御車", icon: "fa-microchip", spec: "自動制御端末" }
        ]
      },
      {
        series: "G7000系 超高音速シャトル (ハイパーライナー)",
        badgeColor: "bg-pink-500/20 text-pink-400 border-pink-500/40",
        desc: "最高速度320km/hを誇る最速旗艦型車両。軌道エレベータ直通特急用。",
        totalSets: "6編成 (G701〜G706)",
        carsCount: "10両",
        carsList: [
          { carNo: "1-2", type: "ファーストクラス", icon: "fa-crown", spec: "無重力リクライニングシート" },
          { carNo: "3-8", type: "ビジネスクラス", icon: "fa-briefcase", spec: "ホログラム作業デスク" },
          { carNo: "9-10", type: "プレミアムラウンジ", icon: "fa-star", spec: "パノラマ宇宙ビュー" }
        ]
      }
    ];

    // --- SYSTEM STATE ---
    let currentTab = "live-map";
    let globalDayLock = "AUTO"; // "AUTO", "平日", "土休日", "特別"
    let systemTimeSeconds = 8 * 3600 + 12 * 60; // 08:12:00 default
    let clockSpeed = 1;
    let liveDisplayMode = "icon"; // "icon" or "card"
    let trainInfoMode = "number"; // "number" or "operation"

    let timetableDay = "平日";
    let timetableDir = "下り";

    // --- INITIALIZATION ---
    window.addEventListener("DOMContentLoaded", () => {
      populateStationDropdowns();
      switchTab("live-map");
      renderAllTabs();
      
      // Start Clock Loop
      setInterval(tickClock, 1000);
    });

    // --- CLOCK CONTROLLER ---
    function tickClock() {
      systemTimeSeconds = (systemTimeSeconds + clockSpeed) % (24 * 3600);
      const h = Math.floor(systemTimeSeconds / 3600);
      const m = Math.floor((systemTimeSeconds % 3600) / 60);
      const s = systemTimeSeconds % 60;
      const timeStr = `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
      
      const clockEl = document.getElementById("system-clock");
      if (clockEl) clockEl.innerText = timeStr;

      // Update Header Day Lock Display
      const headerDayEl = document.getElementById("header-day-type");
      if (headerDayEl) {
        headerDayEl.innerText = (globalDayLock === "AUTO" ? "平日ダイヤ (自動)" : globalDayLock + "ダイヤ");
      }

      // Re-render live location on tick
      if (currentTab === "live-map") {
        renderLiveMap();
      }
    }

    // --- TAB SWITCHER ---
    function switchTab(tabId) {
      currentTab = tabId;
      document.querySelectorAll(".tab-content").forEach(el => el.classList.add("hidden"));
      document.querySelectorAll(".tab-btn").forEach(btn => {
        btn.classList.remove("bg-blue-600/20", "text-blue-400", "border", "border-blue-500/30");
        btn.classList.add("text-slate-400");
      });

      const targetTab = document.getElementById("tab-" + tabId);
      const targetBtn = document.getElementById("tab-btn-" + tabId);
      if (targetTab) targetTab.classList.remove("hidden");
      if (targetBtn) {
        targetBtn.classList.add("bg-blue-600/20", "text-blue-400", "border", "border-blue-500/30");
        targetBtn.classList.remove("text-slate-400");
      }

      // Tab specific re-render
      if (tabId === "timetable") renderTimetable();
      if (tabId === "live-map") renderLiveMap();
      if (tabId === "train-info") renderTrainInfo();
      if (tabId === "fleet") renderFleet();
      if (tabId === "diagram") renderPaperDiagram();
    }

    function renderAllTabs() {
      renderTimetable();
      renderLiveMap();
      renderTrainInfo();
      renderFleet();
      renderPaperDiagram();
    }

    // --- DROPDOWN POPULATION ---
    function populateStationDropdowns() {
      const ttStationSel = document.getElementById("tt-station-select");
      if (ttStationSel) {
        ttStationSel.innerHTML = "";
        Object.keys(STATIONS_DATA).forEach(line => {
          STATIONS_DATA[line].forEach(st => {
            const opt = document.createElement("option");
            opt.value = st.name;
            opt.innerText = `${st.name} (${st.code})`;
            ttStationSel.appendChild(opt);
          });
        });
      }
      updateFormStationOptions();
    }

    function updateFormStationOptions() {
      const line = document.getElementById("form-line") ? document.getElementById("form-line").value : "星空本線";
      const stations = STATIONS_DATA[line] || [];

      const originSel = document.getElementById("form-origin");
      const termSel = document.getElementById("form-terminus");
      const tcStationSel = document.getElementById("form-tc-station");
      const wStationSel = document.getElementById("form-w-station");

      const optionsHtml = stations.map(s => `<option value="${s.name}">${s.name}</option>`).join("");
      
      if (originSel) originSel.innerHTML = optionsHtml;
      if (termSel) {
        termSel.innerHTML = optionsHtml;
        if (stations.length > 1) termSel.selectedIndex = stations.length - 1;
      }
      if (tcStationSel) tcStationSel.innerHTML = optionsHtml;
      if (wStationSel) wStationSel.innerHTML = optionsHtml;
    }

    // --- 3. 走行位置 (LIVE LOCATION) TAB (えれサイト風 縦型表示) ---
    function setLiveDisplayMode(mode) {
      liveDisplayMode = mode;
      const iconBtn = document.getElementById("live-mode-icon-btn");
      const cardBtn = document.getElementById("live-mode-card-btn");

      if (mode === 'icon') {
        iconBtn.className = "px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all";
        cardBtn.className = "px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all";
      } else {
        cardBtn.className = "px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all";
        iconBtn.className = "px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all";
      }
      renderLiveMap();
    }

    function renderLiveMap() {
      const container = document.getElementById("vertical-eresite-container");
      if (!container) return;

      const selectedLine = document.getElementById("live-line-select") ? document.getElementById("live-line-select").value : "星空本線";
      const stations = STATIONS_DATA[selectedLine] || [];

      // Current system time string HH:MM
      const curH = Math.floor(systemTimeSeconds / 3600);
      const curM = Math.floor((systemTimeSeconds % 3600) / 60);
      const currentMinutes = curH * 60 + curM;

      // Filter active trains on this line
      const activeTrains = trainsData.filter(t => t.line === selectedLine);

      let html = "";

      stations.forEach((st, idx) => {
        // Find trains at this station
        const upAtStation = activeTrains.filter(t => t.direction === "上り" && isTrainAtStation(t, st.name, currentMinutes));
        const downAtStation = activeTrains.filter(t => t.direction === "下り" && isTrainAtStation(t, st.name, currentMinutes));

        // STATION ROW
        html += `
          <div class="grid grid-cols-11 items-center my-3 relative z-10 group">
            <!-- Left Side: Inbound (上り) Station Stop Slot -->
            <div class="col-span-5 flex justify-end items-center pr-4 space-x-2">
              ${upAtStation.map(t => renderTrainBadge(t, 'up')).join('')}
            </div>

            <!-- Center: Station Node badge & Central Divide Axis -->
            <div class="col-span-1 flex flex-col items-center justify-center">
              <div class="w-8 h-8 rounded-full bg-slate-900 border-2 border-blue-500 text-blue-300 font-mono font-bold text-xs flex items-center justify-center shadow-lg shadow-blue-500/20 group-hover:border-purple-400 group-hover:scale-110 transition-all">
                ${st.code}
              </div>
              <span class="text-xs font-bold text-white mt-1 group-hover:text-blue-300">${st.name}</span>
            </div>

            <!-- Right Side: Outbound (下り) Station Stop Slot -->
            <div class="col-span-5 flex justify-start items-center pl-4 space-x-2">
              ${downAtStation.map(t => renderTrainBadge(t, 'down')).join('')}
            </div>
          </div>
        `;

        // IN-BETWEEN SECTION (E.g. Between st[idx] and st[idx+1])
        if (idx < stations.length - 1) {
          const stNext = stations[idx + 1];

          // In-transit trains between st.name and stNext.name
          // For DOWN: traveling from st to stNext
          const downBetween = activeTrains.filter(t => t.direction === "下り" && isTrainBetween(t, st.name, stNext.name, currentMinutes));
          // For UP: traveling from stNext to st
          const upBetween = activeTrains.filter(t => t.direction === "上り" && isTrainBetween(t, stNext.name, st.name, currentMinutes));

          html += `
            <div class="grid grid-cols-11 items-center my-4 relative">
              <!-- Central Line Shaft Background -->
              <div class="absolute inset-y-0 left-1/2 -translate-x-1/2 w-1 bg-gradient-to-b from-blue-500 via-indigo-500 to-purple-600 opacity-60 rounded"></div>

              <!-- Left: Inbound Between Trains -->
              <div class="col-span-5 flex justify-end items-center pr-6 space-x-2 z-10">
                ${upBetween.map(t => renderTrainBadge(t, 'up', true)).join('')}
              </div>

              <!-- Center Track Slot Marker -->
              <div class="col-span-1 flex justify-center items-center z-10">
                <div class="w-2 h-2 rounded-full bg-slate-700 border border-slate-500"></div>
              </div>

              <!-- Right: Outbound Between Trains -->
              <div class="col-span-5 flex justify-start items-center pl-6 space-x-2 z-10">
                ${downBetween.map(t => renderTrainBadge(t, 'down', true)).join('')}
              </div>
            </div>
          `;
        }
      });

      container.innerHTML = html;
    }

    // Helper: Badge Renderer for Eresite mode
    function renderTrainBadge(train, dir, isBetween = false) {
      const typeBg = getTypeColor(train.type);
      
      if (liveDisplayMode === "icon") {
        // COMPACT TRAIN ICON MODE
        return `
          <div onclick="openDetailModal('${train.id}')" class="cursor-pointer flex items-center gap-1.5 px-2.5 py-1 rounded-full ${typeBg} text-white font-mono font-bold text-xs shadow-md hover:scale-105 transition-all animate-pulse-train border border-white/20">
            <i class="fa-solid fa-train-subway text-[10px]"></i>
            <span>${train.trainNo}</span>
            <span class="text-[9px] opacity-80 font-sans">(${train.type})</span>
          </div>
        `;
      } else {
        // DETAILED TRAIN CARD MODE
        return `
          <div onclick="openDetailModal('${train.id}')" class="cursor-pointer bg-slate-900/90 border border-slate-700 hover:border-blue-400 p-2 rounded-xl text-xs space-y-1 shadow-lg max-w-[170px] hover:scale-102 transition-all">
            <div class="flex items-center justify-between gap-1">
              <span class="px-1.5 py-0.5 rounded text-[10px] font-bold ${typeBg} text-white">${train.type}</span>
              <span class="font-mono font-bold text-blue-300 text-xs">${train.trainNo}</span>
            </div>
            <div class="text-[10px] text-slate-300 font-mono flex justify-between">
              <span>運用: ${train.operationNo}</span>
              <span class="text-emerald-400">${train.status}</span>
            </div>
            <div class="text-[10px] text-slate-400 truncate">
              行き先: ${train.terminus}
            </div>
            ${isBetween ? `<div class="text-[9px] text-amber-400 animate-pulse">走行中 ➔</div>` : `<div class="text-[9px] text-emerald-400">停車中 停車駅</div>`}
          </div>
        `;
      }
    }

    // Helper: Check train location
    function timeToMinutes(timeStr) {
      if (!timeStr) return 0;
      const [h, m] = timeStr.split(":").map(Number);
      return h * 60 + m;
    }

    function isTrainAtStation(train, stationName, curMin) {
      if (!train.stops) return false;
      const st = train.stops.find(s => s.station === stationName);
      if (!st) return false;
      const arr = timeToMinutes(st.arr);
      const dep = timeToMinutes(st.dep);
      return curMin >= arr && curMin <= dep + 1;
    }

    function isTrainBetween(train, stFrom, stTo, curMin) {
      if (!train.stops) return false;
      const idxFrom = train.stops.findIndex(s => s.station === stFrom);
      const idxTo = train.stops.findIndex(s => s.station === stTo);
      if (idxFrom === -1 || idxTo === -1) return false;

      const depFrom = timeToMinutes(train.stops[idxFrom].dep);
      const arrTo = timeToMinutes(train.stops[idxTo].arr);
      return curMin > depFrom && curMin < arrTo;
    }

    function getTypeColor(type) {
      switch (type) {
        case "宇宙特急": return "bg-pink-600";
        case "ハイパーライナー": return "bg-purple-600";
        case "快速": return "bg-blue-600";
        default: return "bg-slate-700";
      }
    }

    // --- 2. 時刻表 TAB RENDER ---
    function setTimetableDay(day) {
      timetableDay = day;
      ["weekday","weekend","event"].forEach(d => {
        const btn = document.getElementById("tt-day-" + d);
        if (btn) btn.className = "px-2.5 py-1 text-xs rounded-md text-slate-400 hover:text-white";
      });
      const key = day === '平日' ? 'weekday' : day === '土休日' ? 'weekend' : 'event';
      const activeBtn = document.getElementById("tt-day-" + key);
      if (activeBtn) activeBtn.className = "px-2.5 py-1 text-xs rounded-md bg-blue-600 text-white font-medium";
      renderTimetable();
    }

    function setTimetableDir(dir) {
      timetableDir = dir;
      const upBtn = document.getElementById("tt-dir-up");
      const downBtn = document.getElementById("tt-dir-down");
      if (dir === '下り') {
        downBtn.className = "px-3 py-1 text-xs rounded-md bg-indigo-600 text-white font-medium";
        upBtn.className = "px-3 py-1 text-xs rounded-md text-slate-400 hover:text-white";
      } else {
        upBtn.className = "px-3 py-1 text-xs rounded-md bg-indigo-600 text-white font-medium";
        downBtn.className = "px-3 py-1 text-xs rounded-md text-slate-400 hover:text-white";
      }
      renderTimetable();
    }

    function renderTimetable() {
      const container = document.getElementById("timetable-paper-grid");
      if (!container) return;

      const station = document.getElementById("tt-station-select") ? document.getElementById("tt-station-select").value : "中央未来";

      // Hours 5 to 24
      let html = "";
      for (let hour = 5; hour <= 24; hour++) {
        const hourStr = String(hour).padStart(2, '0');
        
        // Find matching departure items
        const departures = [];
        trainsData.forEach(t => {
          if (t.direction !== timetableDir) return;
          if (!t.stops) return;
          const stInfo = t.stops.find(s => s.station === station);
          if (stInfo && stInfo.dep) {
            const [h, m] = stInfo.dep.split(":");
            if (parseInt(h) === hour) {
              departures.push({ train: t, min: m, depTime: stInfo.dep });
            }
          }
        });

        departures.sort((a,b) => parseInt(a.min) - parseInt(b.min));

        html += `
          <div class="flex items-center border-b border-slate-800/80 py-2 hover:bg-slate-900/40 transition-colors">
            <div class="w-16 font-mono font-bold text-lg text-blue-400 border-r border-slate-800 pr-3 text-right">
              ${hourStr}
            </div>
            <div class="flex-1 flex flex-wrap gap-2 pl-4 items-center">
              ${departures.length === 0 ? `<span class="text-slate-600 text-xs italic">運行なし</span>` : ''}
              ${departures.map(d => `
                <div onclick="openDetailModal('${d.train.id}')" class="cursor-pointer flex items-center gap-1 bg-slate-900 border border-slate-700 hover:border-blue-400 px-2 py-1 rounded text-xs font-mono transition-all hover:scale-105">
                  <span class="font-bold text-white">${d.min}</span>
                  <span class="text-[10px] px-1 rounded ${getTypeColor(d.train.type)} text-white">${d.train.type.substring(0,2)}</span>
                  <span class="text-[10px] text-slate-400">${d.train.terminus}行</span>
                </div>
              `).join('')}
            </div>
          </div>
        `;
      }
      container.innerHTML = html;
    }

    // --- 4. 列車情報 TAB RENDER ---
    function setTrainInfoMode(mode) {
      trainInfoMode = mode;
      const numBtn = document.getElementById("info-mode-num-btn");
      const opBtn = document.getElementById("info-mode-op-btn");

      if (mode === 'number') {
        numBtn.className = "px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all";
        opBtn.className = "px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all";
        document.getElementById("train-info-table-container").classList.remove("hidden");
        document.getElementById("train-info-op-container").classList.add("hidden");
      } else {
        opBtn.className = "px-3 py-1 rounded-md bg-blue-600 text-white font-medium transition-all";
        numBtn.className = "px-3 py-1 rounded-md text-slate-400 hover:text-white transition-all";
        document.getElementById("train-info-table-container").classList.add("hidden");
        document.getElementById("train-info-op-container").classList.remove("hidden");
      }
      renderTrainInfo();
    }

    function renderTrainInfo() {
      const badge = document.getElementById("train-count-badge");
      if (badge) badge.innerText = trainsData.length + "本";

      // Render Table View
      const tbody = document.getElementById("train-info-tbody");
      if (tbody) {
        tbody.innerHTML = trainsData.map(t => `
          <tr class="hover:bg-slate-900/60 transition-colors">
            <td class="p-3 font-bold text-blue-300">${t.trainNo}</td>
            <td class="p-3 text-slate-300">${t.operationNo}</td>
            <td class="p-3"><span class="px-2 py-0.5 rounded text-[10px] ${getTypeColor(t.type)} text-white font-sans">${t.type}</span></td>
            <td class="p-3 text-slate-400">${t.line}</td>
            <td class="p-3 text-slate-200">${t.origin} ➔ ${t.terminus}</td>
            <td class="p-3 text-slate-400">${t.series} (${t.cars})</td>
            <td class="p-3"><span class="text-emerald-400">${t.status}</span></td>
            <td class="p-3 text-right space-x-1">
              <button onclick="openDetailModal('${t.id}')" class="px-2 py-1 rounded bg-slate-800 text-blue-400 hover:bg-slate-700">通常</button>
              <button onclick="openEditModal('${t.id}')" class="px-2 py-1 rounded bg-slate-800 text-amber-400 hover:bg-slate-700">編集</button>
              <button onclick="duplicateTrain('${t.id}')" class="px-2 py-1 rounded bg-slate-800 text-purple-400 hover:bg-slate-700">複製</button>
              <button onclick="deleteTrain('${t.id}')" class="px-2 py-1 rounded bg-slate-800 text-pink-400 hover:bg-slate-700">削除</button>
            </td>
          </tr>
        `).join('');
      }

      // Render Operation Timeline View
      const opContainer = document.getElementById("train-info-op-container");
      if (opContainer) {
        // Group trains by operationNo
        const ops = {};
        trainsData.forEach(t => {
          if (!ops[t.operationNo]) ops[t.operationNo] = [];
          ops[t.operationNo].push(t);
        });

        opContainer.innerHTML = Object.keys(ops).map(opId => `
          <div class="bg-cyber-card border border-cyber-border rounded-xl p-4 flex flex-col md:flex-row items-start md:items-center gap-4">
            <div class="w-24 h-16 rounded-xl bg-gradient-to-br from-indigo-600 to-purple-700 flex flex-col items-center justify-center font-mono text-white shadow-lg shrink-0">
              <span class="text-[10px] text-indigo-200">運用番号</span>
              <span class="text-xl font-black">${opId}</span>
            </div>

            <div class="flex-1 flex gap-3 overflow-x-auto py-2 w-full">
              ${ops[opId].map(t => `
                <div onclick="openDetailModal('${t.id}')" class="cursor-pointer bg-slate-900 border border-slate-700 hover:border-blue-500 p-3 rounded-lg shrink-0 min-w-[180px] space-y-1">
                  <div class="flex items-center justify-between">
                    <span class="font-mono font-bold text-blue-300 text-sm">${t.trainNo}</span>
                    <span class="text-[10px] px-1.5 py-0.5 rounded ${getTypeColor(t.type)} text-white">${t.type}</span>
                  </div>
                  <div class="text-xs text-slate-300 truncate">${t.origin} ➔ ${t.terminus}</div>
                  <div class="text-[10px] font-mono text-slate-400">${t.startTime} 出発</div>
                </div>
              `).join('<div class="self-center text-slate-600 font-bold">➔</div>')}
            </div>
          </div>
        `).join('');
      }
    }

    // CRUD ACTIONS
    function deleteTrain(id) {
      if (confirm("この列車データを削除しますか？")) {
        trainsData = trainsData.filter(t => t.id !== id);
        renderAllTabs();
      }
    }

    function duplicateTrain(id) {
      const train = trainsData.find(t => t.id === id);
      if (train) {
        const copy = JSON.parse(JSON.stringify(train));
        copy.id = "tr-" + Date.now();
        copy.trainNo = copy.trainNo + "D";
        trainsData.push(copy);
        renderAllTabs();
      }
    }

    // EDIT MODAL
    function openEditModal(id) {
      const train = trainsData.find(t => t.id === id);
      if (!train) return;
      document.getElementById("edit-train-id").value = train.id;
      document.getElementById("edit-train-no").value = train.trainNo;
      document.getElementById("edit-op-no").value = train.operationNo;
      document.getElementById("edit-type").value = train.type;
      document.getElementById("edit-status").value = train.status;
      document.getElementById("edit-train-modal").classList.remove("hidden");
    }

    function closeEditModal() {
      document.getElementById("edit-train-modal").classList.add("hidden");
    }

    function saveTrainEdit() {
      const id = document.getElementById("edit-train-id").value;
      const train = trainsData.find(t => t.id === id);
      if (train) {
        train.trainNo = document.getElementById("edit-train-no").value;
        train.operationNo = document.getElementById("edit-op-no").value;
        train.type = document.getElementById("edit-type").value;
        train.status = document.getElementById("edit-status").value;
        closeEditModal();
        renderAllTabs();
      }
    }

    // DETAIL MODAL
    function openDetailModal(id) {
      const train = trainsData.find(t => t.id === id);
      if (!train) return;

      const content = document.getElementById("detail-modal-content");
      content.innerHTML = `
        <div class="space-y-4">
          <div class="flex items-center justify-between border-b border-slate-800 pb-3">
            <div>
              <span class="px-2 py-0.5 rounded text-xs ${getTypeColor(train.type)} text-white font-bold">${train.type}</span>
              <h3 class="text-xl font-extrabold font-mono text-white mt-1">${train.trainNo} <span class="text-xs text-slate-400 font-sans">(${train.operationNo}運用)</span></h3>
            </div>
            <span class="px-3 py-1 rounded-full bg-emerald-500/20 text-emerald-400 text-xs border border-emerald-500/30 font-bold">${train.status}</span>
          </div>

          <div class="grid grid-cols-2 gap-2 text-xs bg-slate-900/80 p-3 rounded-xl border border-slate-800 font-mono">
            <div><span class="text-slate-400">路線:</span> ${train.line}</div>
            <div><span class="text-slate-400">方向:</span> ${train.direction}</div>
            <div><span class="text-slate-400">区間:</span> ${train.origin} ➔ ${train.terminus}</div>
            <div><span class="text-slate-400">編成:</span> ${train.series} (${train.cars})</div>
          </div>

          ${train.coupling ? `
            <div class="bg-purple-950/40 border border-purple-500/30 p-2.5 rounded-lg text-xs space-y-1">
              <span class="font-bold text-purple-300">併結編成:</span>
              <div class="text-[11px] text-slate-300">前部: ${train.coupling.front} | 後部: ${train.coupling.rear}</div>
            </div>
          ` : ''}

          <div>
            <h4 class="text-xs font-bold text-slate-300 mb-2">停車駅・通過予定時刻</h4>
            <div class="max-h-48 overflow-y-auto space-y-1 text-xs font-mono pr-1">
              ${(train.stops || []).map(s => `
                <div class="flex items-center justify-between bg-slate-900/60 p-2 rounded border border-slate-800">
                  <span class="text-white font-bold">${s.station}</span>
                  <span class="text-blue-300">着 ${s.arr} / 発 ${s.dep}</span>
                </div>
              `).join('')}
            </div>
          </div>
        </div>
      `;

      document.getElementById("detail-modal").classList.remove("hidden");
    }

    function closeDetailModal() {
      document.getElementById("detail-modal").classList.add("hidden");
    }

    // --- 5. 列車追加 FORM logic ---
    function toggleFormSection(secId, show) {
      const el = document.getElementById(secId);
      if (el) {
        if (show) el.classList.remove("hidden");
        else el.classList.add("hidden");
      }
    }

    function handleAddTrainSubmit(e) {
      e.preventDefault();
      const line = document.getElementById("form-line").value;
      const origin = document.getElementById("form-origin").value;
      const terminus = document.getElementById("form-terminus").value;
      const stations = STATIONS_DATA[line] || [];

      // Build simple stop array
      const stops = stations.map((st, i) => {
        const depMin = 8 * 60 + i * 4;
        const arrMin = depMin - 1;
        const hArr = Math.floor(arrMin / 60); const mArr = arrMin % 60;
        const hDep = Math.floor(depMin / 60); const mDep = depMin % 60;
        return {
          station: st.name,
          arr: `${String(hArr).padStart(2,'0')}:${String(mArr).padStart(2,'0')}`,
          dep: `${String(hDep).padStart(2,'0')}:${String(mDep).padStart(2,'0')}`
        };
      });

      const newTrain = {
        id: "tr-" + Date.now(),
        trainNo: document.getElementById("form-train-no").value,
        operationNo: document.getElementById("form-op-no").value,
        line: line,
        type: document.getElementById("form-type").value,
        direction: "下り",
        origin: origin,
        terminus: terminus,
        series: document.getElementById("form-series").value,
        cars: document.getElementById("form-cars").value,
        status: "定時",
        dayType: "平日",
        startTime: stops[0].dep,
        endTime: stops[stops.length - 1].arr,
        stops: stops,
        typeChange: document.getElementById("chk-type-change").checked ? {
          station: document.getElementById("form-tc-station").value,
          newType: document.getElementById("form-tc-new-type").value,
          newNo: document.getElementById("form-tc-new-no").value
        } : null,
        coupling: document.getElementById("chk-coupling").checked ? {
          front: document.getElementById("form-c-front").value,
          rear: document.getElementById("form-c-rear").value
        } : null,
        work: document.getElementById("chk-work").checked ? {
          station: document.getElementById("form-w-station").value,
          type: document.getElementById("form-w-type").value
        } : null
      };

      trainsData.push(newTrain);
      alert("新規列車をダイヤに登録しました！");
      document.getElementById("add-train-form").reset();
      switchTab("train-info");
    }

    // --- 6. 編成表 TAB RENDER ---
    function renderFleet() {
      const container = document.getElementById("fleet-cards-container");
      if (!container) return;

      container.innerHTML = FLEET_DATA.map(f => `
        <div class="bg-cyber-card border border-cyber-border rounded-xl p-6 space-y-4 shadow-xl">
          <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-3">
            <div>
              <span class="px-2.5 py-0.5 rounded text-xs font-bold ${f.badgeColor}">${f.carsCount}</span>
              <h3 class="text-lg font-bold text-white mt-1">${f.series}</h3>
            </div>
            <span class="text-xs text-slate-400 font-mono">${f.totalSets} 所属</span>
          </div>
          <p class="text-xs text-slate-300">${f.desc}</p>
          
          <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3 pt-2">
            ${f.carsList.map(c => `
              <div class="bg-slate-900/80 border border-slate-800 p-3 rounded-lg space-y-1">
                <div class="flex items-center justify-between">
                  <span class="text-xs font-bold text-blue-400 font-mono">${c.carNo}号車</span>
                  <i class="fa-solid ${c.icon} text-slate-500 text-xs"></i>
                </div>
                <div class="text-xs font-bold text-white">${c.type}</div>
                <div class="text-[10px] text-slate-400">${c.spec}</div>
              </div>
            `).join('')}
          </div>
        </div>
      `).join('');
    }

    // --- 7. ダイヤ表 TAB RENDER ---
    function renderPaperDiagram() {
      const container = document.getElementById("paper-diagram-matrix");
      if (!container) return;

      const selectedLine = document.getElementById("diagram-line-select") ? document.getElementById("diagram-line-select").value : "星空本線";
      const selectedDir = document.getElementById("diagram-dir-select") ? document.getElementById("diagram-dir-select").value : "下り";
      
      const stations = STATIONS_DATA[selectedLine] || [];
      const lineTrains = trainsData.filter(t => t.line === selectedLine && t.direction === selectedDir);

      let html = `
        <table class="w-full text-center border-collapse border border-slate-800 font-mono">
          <thead>
            <tr class="bg-slate-900 text-slate-300">
              <th class="p-2 border border-slate-800 text-left min-w-[120px]">列車種別</th>
              ${lineTrains.map(t => `<th class="p-2 border border-slate-800"><span class="px-1.5 py-0.5 rounded text-[10px] ${getTypeColor(t.type)} text-white">${t.type}</span></th>`).join('')}
            </tr>
            <tr class="bg-slate-900 text-slate-300">
              <th class="p-2 border border-slate-800 text-left">列車番号</th>
              ${lineTrains.map(t => `<th class="p-2 border border-slate-800 font-bold text-blue-300">${t.trainNo}</th>`).join('')}
            </tr>
            <tr class="bg-slate-900/60 text-slate-400 text-[10px]">
              <th class="p-2 border border-slate-800 text-left">運用番号</th>
              ${lineTrains.map(t => `<th class="p-2 border border-slate-800">${t.operationNo}</th>`).join('')}
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-800">
      `;

      stations.forEach(st => {
        html += `
          <tr class="hover:bg-slate-900/40">
            <td class="p-2 border border-slate-800 text-left font-sans font-bold text-white bg-slate-900/40">${st.name}</td>
            ${lineTrains.map(t => {
              const stop = (t.stops || []).find(s => s.station === st.name);
              if (stop) {
                return `<td class="p-2 border border-slate-800 text-blue-200">${stop.dep}</td>`;
              } else {
                return `<td class="p-2 border border-slate-800 text-slate-600">レ</td>`;
              }
            }).join('')}
          </tr>
        `;
      });

      html += `
          </tbody>
        </table>
      `;

      container.innerHTML = html;
    }

    // --- 8. SETTINGS CONTROLLER ---
    function setGlobalDayLock(day) {
      globalDayLock = day;
      document.querySelectorAll(".setting-day-btn").forEach(btn => {
        btn.className = "setting-day-btn px-3 py-2 rounded-lg bg-slate-900 text-slate-400 text-xs hover:bg-slate-800 border border-slate-800";
      });
      const activeBtn = document.getElementById("lock-btn-" + day);
      if (activeBtn) {
        activeBtn.className = "setting-day-btn px-3 py-2 rounded-lg bg-blue-600 text-white text-xs font-medium border border-blue-500";
      }
    }

    function setClockSpeed(spd) {
      clockSpeed = spd;
      const speedBadge = document.getElementById("speed-badge");
      if (speedBadge) speedBadge.innerText = spd + "x";

      document.querySelectorAll(".speed-btn").forEach(btn => {
        btn.className = "speed-btn px-3 py-1.5 rounded-lg bg-slate-900 text-slate-400 text-xs font-mono hover:bg-slate-800";
      });
      const activeBtn = document.getElementById("speed-btn-" + spd);
      if (activeBtn) {
        activeBtn.className = "speed-btn px-3 py-1.5 rounded-lg bg-blue-600 text-white text-xs font-mono font-bold";
      }
    }

    function applyManualTime() {
      const val = document.getElementById("manual-time-input").value;
      if (val) {
        const [h, m] = val.split(":").map(Number);
        systemTimeSeconds = h * 3600 + m * 60;
        tickClock();
      }
    }

    function resetToRealTime() {
      const now = new Date();
      systemTimeSeconds = now.getHours() * 3600 + now.getMinutes() * 60 + now.getSeconds();
      tickClock();
    }
  </script>
</body>
</html>
