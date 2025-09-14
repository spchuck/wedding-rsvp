<index html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>婚宴出席回覆（RSVP）</title>
  <!-- Tailwind：快速排版用 -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Chart.js：後台圓餅圖用 -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    .card{box-shadow:0 10px 25px rgba(0,0,0,.06)}
  </style>
</head>
<body class="bg-pink-50 min-h-screen">
  <div class="max-w-4xl mx-auto p-6">
    <!-- 頁首標題區 -->
    <header class="text-center mb-6">
      <h1 class="text-3xl font-bold text-pink-700">💌 婚宴出席回覆 RSVP</h1>
      <p class="text-gray-600">請填寫您的出席意願與人數，感謝！</p>
    </header>

    <!-- 前台表單區（所有人可見） -->
    <section id="section-form" class="card bg-white rounded-2xl p-6">
      <form id="rsvpForm" class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <!-- 姓名（必填） -->
        <div class="md:col-span-1">
          <label class="block text-gray-700 font-medium">姓名 <span class="text-pink-600">*</span></label>
          <input name="name" type="text" required class="w-full border rounded-lg p-2 mt-1" placeholder="王小明" />
        </div>
        <!-- 是否出席（必填） -->
        <div class="md:col-span-1">
          <label class="block text-gray-700 font-medium">是否出席 <span class="text-pink-600">*</span></label>
          <select name="attendance" required class="w-full border rounded-lg p-2 mt-1">
            <option value="">請選擇</option>
            <option value="yes">會出席</option>
            <option value="no">不克出席</option>
          </select>
        </div>
        <!-- 攜伴人數（預設 0） -->
        <div class="md:col-span-1">
          <label class="block text-gray-700 font-medium">攜伴人數</label>
          <input name="guests" type="number" min="0" value="0" class="w-full border rounded-lg p-2 mt-1" />
        </div>
        <!-- 餐點偏好（一般/素食/兒童） -->
        <div class="md:col-span-1">
          <label class="block text-gray-700 font-medium">餐點偏好</label>
          <select name="meal" class="w-full border rounded-lg p-2 mt-1">
            <option value="none">一般</option>
            <option value="veg">素食</option>
            <option value="child">兒童餐</option>
          </select>
        </div>
        <!-- 食物過敏（選填） -->
        <div class="md:col-span-2">
          <label class="block text-gray-700 font-medium">食物過敏（選填）</label>
          <input name="allergies" type="text" class="w-full border rounded-lg p-2 mt-1" placeholder="例：花生、海鮮" />
        </div>
        <!-- 備註（選填） -->
        <div class="md:col-span-2">
          <label class="block text-gray-700 font-medium">備註（選填）</label>
          <textarea name="note" rows="3" class="w-full border rounded-lg p-2 mt-1" placeholder="想對新人說的話…"></textarea>
        </div>
        <!-- 送出按鈕與狀態顯示 -->
        <div class="md:col-span-2 flex items-center gap-3">
          <button type="submit" class="flex-1 bg-pink-600 text-white rounded-lg py-2 hover:bg-pink-700">送出回覆</button>
          <span id="formStatus" class="text-sm text-gray-500"></span>
        </div>
      </form>
    </section>

    <!-- 後台儀表板（僅帶 admin_key 才顯示） -->
    <section id="section-admin" class="hidden mt-8 grid grid-cols-1 gap-6">
      <!-- 數字卡 + 圖表 -->
      <div class="card bg-white rounded-2xl p-6">
        <h2 class="text-xl font-bold mb-2">📊 即時統計（僅管理者）</h2>
        <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-4">
          <div class="bg-pink-50 rounded-xl p-4 text-center">
            <div class="text-sm text-gray-500">出席戶數</div>
            <div id="statYes" class="text-2xl font-bold">0</div>
          </div>
          <div class="bg-gray-50 rounded-xl p-4 text-center">
            <div class="text-sm text-gray-500">不克出席戶數</div>
            <div id="statNo" class="text-2xl font-bold">0</div>
          </div>
          <div class="bg-pink-50 rounded-xl p-4 text-center">
            <div class="text-sm text-gray-500">預估到場總人數</div>
            <div id="statHeads" class="text-2xl font-bold">0</div>
          </div>
          <div class="bg-gray-50 rounded-xl p-4 text-center">
            <div class="text-sm text-gray-500">素食份數</div>
            <div id="statVeg" class="text-2xl font-bold">0</div>
          </div>
        </div>
        <canvas id="statsChart" height="120"></canvas>
        <div class="text-right mt-3">
          <button id="btnExport" class="px-3 py-2 bg-gray-800 text-white rounded-lg">匯出 CSV</button>
        </div>
      </div>

      <!-- 明細表（僅管理者） -->
      <div class="card bg-white rounded-2xl p-6">
        <h3 class="text-lg font-bold mb-3">明細列表</h3>
        <div class="overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead>
              <tr class="text-left text-gray-600">
                <th class="p-2">時間</th>
                <th class="p-2">姓名</th>
                <th class="p-2">出席</th>
                <th class="p-2">攜伴</th>
                <th class="p-2">餐點</th>
                <th class="p-2">過敏</th>
                <th class="p-2">備註</th>
              </tr>
            </thead>
            <tbody id="tbody"></tbody>
          </table>
        </div>
      </div>
    </section>
  </div>

  <script>
    // ====== 可調整設定（必改） ======
    const CONFIG = {
      apiBase: "https://script.google.com/macros/s/【AKfycbxzyoijk5WxFAK-mk_KUNtrpbOtgUM0_yWwUfeCqOPLToqJx4GpuITUgEhdAKmdYC6Auw】/exec", // ← 你的 GAS Web App URL（/exec）
      adminKey: "123456" // ← 與後端 ADMIN_KEY 一致
    };

    // 前台：提交表單
    const form = document.getElementById('rsvpForm');
    const formStatus = document.getElementById('formStatus');
    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      formStatus.textContent = '送出中…';

      // 取表單內容
      const fd = new FormData(form);
      const payload = {
        name: (fd.get('name')||'').trim(),
        attendance: fd.get('attendance'),
        guests: Number(fd.get('guests')||0),
        meal: fd.get('meal')||'none',
        allergies: (fd.get('allergies')||'').trim(),
        note: (fd.get('note')||'').trim()
      };

      // 簡單檢核
      if (!payload.name || !payload.attendance) {
        formStatus.textContent = '請完整填寫必填欄位。';
        return;
      }

      try {
        // 呼叫後端（POST JSON）
        const res = await fetch(CONFIG.apiBase, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ action: 'submit', data: payload })
        });
        const json = await res.json();
        if (json.ok) {
          form.reset();
          formStatus.textContent = '已收到，謝謝您的回覆！';
          setTimeout(() => formStatus.textContent = '', 2500);
        } else {
          formStatus.textContent = '送出失敗，請稍後再試。';
        }
      } catch (err) {
        console.error(err);
        formStatus.textContent = '連線發生問題。';
      }
    });

    // 決定是否顯示後台：?admin_key=...
    const params = new URLSearchParams(location.search);
    const adminKey = params.get('admin_key');
    const isAdmin = adminKey && adminKey === CONFIG.adminKey;
    if (isAdmin) {
      document.getElementById('section-admin').classList.remove('hidden');
      initAdmin();
    }

    // 後台初始化
    let chart;
    async function initAdmin(){
      await refreshStats();
      setInterval(refreshStats, 20*1000); // 每 20 秒更新一次
      document.getElementById('btnExport').addEventListener('click', exportCSV);
    }

    // 後台拉統計 + 明細
    async function refreshStats(){
      try{
        const url = `${CONFIG.apiBase}?action=stats&admin_key=${encodeURIComponent(adminKey)}`;
        const res = await fetch(url);
        const json = await res.json();
        if(!json.ok) return;
        const { summary, rows } = json;

        // 更新數字卡
        byId('statYes').textContent = summary.yesCount;
        byId('statNo').textContent = summary.noCount;
        byId('statHeads').textContent = summary.expectedHeads;
        byId('statVeg').textContent = summary.vegCount;

        // 更新圖表
        const data = { labels: ['會出席','不克出席'], datasets: [{ data: [summary.yesCount, summary.noCount] }] };
        const ctx = byId('statsChart').getContext('2d');
        if(chart) chart.destroy();
        chart = new Chart(ctx, { type: 'doughnut', data });

        // 更新明細表
        const tbody = byId('tbody');
        tbody.innerHTML = rows.map(r => `
          <tr class="border-t">
            <td class="p-2">${escapeHTML(r.timestamp)}</td>
            <td class="p-2">${escapeHTML(r.name)}</td>
            <td class="p-2">${r.attendance==='yes'?'出席':'不克'}</td>
            <td class="p-2">${Number(r.guests)||0}</td>
            <td class="p-2">${r.meal==='veg'?'素食':(r.meal==='child'?'兒童':'一般')}</td>
            <td class="p-2">${escapeHTML(r.allergies||'')}</td>
            <td class="p-2">${escapeHTML(r.note||'')}</td>
          </tr>
        `).join('');
      }catch(err){
        console.error(err);
      }
    }

    // 匯出 CSV（由後端直接產出）
    function exportCSV(){
      const url = `${CONFIG.apiBase}?action=export&admin_key=${encodeURIComponent(adminKey)}`;
      window.open(url, '_blank');
    }

    // 小工具
    function byId(id){ return document.getElementById(id); }
    function escapeHTML(s){ return (s||'').replace(/[&<>\"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','\"':'&quot;'}[c])); }
  </script>
</body>
</html>
