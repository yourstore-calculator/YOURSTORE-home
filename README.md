# YOURSTORE-home
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <title>حاسبة الأبواب والنوافذ</title>
  <style>
    body { font-family: 'Tahoma', sans-serif; direction: rtl; background: #f4f4f4; padding: 20px; }
    h1 { color: #333; }
    label, select, input, button { display: block; margin: 10px 0; }
    #results { background: #fff; padding: 15px; border: 1px solid #ccc; margin-top: 20px; }
    .hidden { display: none; }
  </style>
</head>
<body>

<h1>حاسبة الأبواب والنوافذ</h1>

<form id="calculator">
  <label>النوع:
    <select id="productType">
      <option value="window">نافذة</option>
      <option value="door">باب</option>
      <option value="movingDoor">باب متحرك</option>
    </select>
  </label>

  <label>النوع الفرعي:
    <select id="subType"></select>
  </label>

  <div id="dimensions">
    <label>الطول (متر): <input type="number" step="0.01" id="length"></label>
    <label>العرض (متر): <input type="number" step="0.01" id="width"></label>
  </div>

  <div id="extraOptions" class="hidden">
    <label>الإضافات:
      <select id="addon">
        <option value="none">بدون إضافات</option>
        <option value="curtain">ستارة</option>
        <option value="mesh">شبك</option>
      </select>
    </label>
  </div>

  <div id="curtainArea" class="hidden">
    <label>طول الستارة (م): <input type="number" step="0.01" id="curtainLength"></label>
    <label>عرض الستارة (م): <input type="number" step="0.01" id="curtainWidth"></label>
  </div>

  <button type="submit">احسب</button>
</form>

<div id="results"></div>
<button onclick="downloadResults()">💾 حفظ النتائج</button>
<button onclick="clearResults()">🗑️ محو النتائج</button>

<script>
const prices = {
  window: {
    "دبل جلاس دبل فريم - ثابت": { price: 34, cbm: 0.13, addons: true },
    "دبل جلاس دبل فريم - حركة": { price: 73, cbm: 0.13, addons: true },
    "دبل جلاس دبل فريم - حركتين": { price: 92, cbm: 0.13, addons: true },
    "دبل جلاس سنجل فريم - ثابت": { price: 26, cbm: 0.07, addons: true },
    "دبل جلاس سنجل فريم - حركة": { price: 46, cbm: 0.07, addons: true },
    "دبل جلاس سنجل فريم - حركتين": { price: 58, cbm: 0.07, addons: true },
    "سنجل - ثابت": { price: 20, cbm: 0.07, addons: true },
    "سنجل - حركة": { price: 43, cbm: 0.07, addons: true },
    "سنجل - حركتين": { price: 47, cbm: 0.07, addons: true },
    "سكاي لايت - بدون ماكينة": { price: 56, cbm: 0.13, addons: false },
    "سكاي لايت - مع ماكينة": { price: 145, cbm: 0.13, addons: false },
    "كارتن وول - ثقيل": { price: 56, cbm: 0.15, addons: false },
    "كارتن وول - خفيف": { price: 45, cbm: 0.15, addons: false }
  },
  door: {
    "WPC - فارغ": { price: 45, cbm: 0.11, fixedSize: true },
    "WPC - مع خشب": { price: 50, cbm: 0.11, fixedSize: true },
    "WPC - عازل صوت": { price: 60, cbm: 0.11, fixedSize: true },
    "WPC - فريم المنيوم": { price: 67, cbm: 0.11, fixedSize: true },
    "ألمنيوم - فارغ": { price: 65, cbm: 0.11, fixedSize: true },
    "ألمنيوم - مع خشب": { price: 75, cbm: 0.11, fixedSize: true },
    "ألمنيوم - فل": { price: 85, cbm: 0.11, fixedSize: true },
    "ألمنيوم - مخفي": { price: 110, cbm: 0.11, fixedSize: true },
    "باب دورات مياه - جديد": { price: 55, cbm: 0.11, fixedSize: true },
    "باب دورات مياه - قديم": { price: 45, cbm: 0.11, fixedSize: true },
    "باب دورات مياه - مخفي زجاجي": { price: 65, cbm: 0.11, fixedSize: true }
  },
  movingDoor: {
    "سحب - داخلي زجاج": { price: 38, cbm: 0.15, addons: true },
    "سحب - داخلي متين": { price: 41, cbm: 0.15, addons: true },
    "سحب - خارجي مفتوح": { price: 55, cbm: 0.15, addons: true },
    "سحب - خارجي جزئين": { price: 58, cbm: 0.15, addons: true },
    "WPC سلايد": { price: 61, cbm: 0.15, addons: true },
    "فولدنج - داخلي": { price: 39, cbm: 0.15, addons: true },
    "فولدنج - خارجي": { price: 56, cbm: 0.15, addons: true }
  }
};

let savedResults = [];

document.getElementById('productType').addEventListener('change', updateSubTypes);
document.getElementById('subType').addEventListener('change', checkAddons);
document.getElementById('addon').addEventListener('change', () => {
  document.getElementById('curtainArea').classList.toggle('hidden', document.getElementById('addon').value !== 'curtain');
});

function updateSubTypes() {
  const type = document.getElementById('productType').value;
  const subType = document.getElementById('subType');
  subType.innerHTML = "";
  for (let key in prices[type]) {
    const opt = document.createElement('option');
    opt.value = key;
    opt.textContent = key;
    subType.appendChild(opt);
  }
  checkAddons();
}

function checkAddons() {
  const type = document.getElementById('productType').value;
  const subType = document.getElementById('subType').value;
  const item = prices[type][subType];

  if (item.fixedSize) {
    document.getElementById('length').value = 2.2;
    document.getElementById('width').value = 1;
  }

  document.getElementById('extraOptions').classList.toggle('hidden', !item.addons);
}

document.getElementById('calculator').addEventListener('submit', function(e) {
  e.preventDefault();
  const type = document.getElementById('productType').value;
  const subType = document.getElementById('subType').value;
  const length = parseFloat(document.getElementById('length').value);
  const width = parseFloat(document.getElementById('width').value);
  const addon = document.getElementById('addon').value;
  const item = prices[type][subType];
  const area = length * width;
  const base = area * item.price;

  let shipping = area * item.cbm * 48;
  let total = base + shipping;

  if (addon === 'curtain') {
    const cl = parseFloat(document.getElementById('curtainLength').value);
    const cw = parseFloat(document.getElementById('curtainWidth').value);
    total += cl * cw * 26;
  } else if (addon === 'mesh') {
    const meshArea = area * 0.5;
    let meshPrice = 0;
    if (subType.includes("باب")) meshPrice = 39;
    else if (subType.includes("فولدنج")) meshPrice = 18;
    else meshPrice = 14;
    total += meshArea * meshPrice;
  }

  const result = `✅ ${subType}: ${total.toFixed(2)} ريال (المساحة: ${area.toFixed(2)}م²)`;
  savedResults.push(result);
  displayResults();
});

function displayResults() {
  document.getElementById('results').innerHTML = savedResults.join("<br>");
}

function clearResults() {
  savedResults = [];
  displayResults();
}

function downloadResults() {
  const blob = new Blob([savedResults.join("\n")], { type: "text/plain" });
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = "نتائج-الحاسبة.txt";
  a.click();
}

updateSubTypes(); // Initialize
</script>

</body>
</html>
