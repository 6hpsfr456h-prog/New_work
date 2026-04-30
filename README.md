# New_work
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>筋トレまとめて記録</title>

<style>
body {
  font-family: sans-serif;
  margin: 0;
  padding: 10px;
  background: #f5f5f5;
}

h2 {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

#ymSelect {
  display: flex;
  gap: 5px;
}

select {
  padding: 5px;
  border-radius: 5px;
}

/* チェックボックス */
#controls {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 15px 0;
}

label {
  background: #fff;
  padding: 8px 12px;
  border-radius: 8px;
  border: 1px solid #ccc;
}

/* メモ */
#memo {
  width: 100%;
  height: 60px;
  border-radius: 8px;
  border: 1px solid #ccc;
  padding: 8px;
}

/* ボタン */
#saveBtn {
  margin-top: 10px;
  padding: 10px;
  width: 100%;
  border: none;
  border-radius: 8px;
  background: #2196F3;
  color: white;
  font-size: 16px;
}

/* カレンダー */
#calendar {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 5px;
  margin-top: 10px;
}

.day {
  background: #fff;
  padding: 10px;
  text-align: center;
  border-radius: 8px;
  cursor: pointer;
}

.day.has-record {
  background: #4CAF50;
  color: white;
  font-weight: bold;
}

.day.selected {
  border: 2px solid red;
}

.day:hover {
  background: #ddd;
}

/* 詳細 */
#detail {
  margin-top: 20px;
  background: #fff;
  padding: 15px;
  border-radius: 10px;
}

@media (max-width: 600px) {
  .day {
    padding: 15px 0;
  }
}
</style>
</head>

<body>

<h2>
  筋トレ記録
  <div id="ymSelect">
    <select id="year"></select>
    <select id="month"></select>
  </div>
</h2>

<!-- 上 -->
<div id="controls">
  <label><input type="checkbox" value="胸"> 胸</label>
  <label><input type="checkbox" value="背中"> 背中</label>
  <label><input type="checkbox" value="肩"> 肩</label>
  <label><input type="checkbox" value="腕"> 腕</label>
  <label><input type="checkbox" value="脚"> 脚</label>
</div>

<textarea id="memo" placeholder="メモ（重量・回数などを入力してください。）"></textarea>

<button id="saveBtn">保存</button>

<!-- 中 -->
<div id="calendar"></div>

<!-- 下 -->
<div id="detail">
  <h3>詳細</h3>
  <p id="detailText">日付を選択してください</p>
</div>

<script>
const calendarEl = document.getElementById("calendar");
const detailText = document.getElementById("detailText");
const yearSelect = document.getElementById("year");
const monthSelect = document.getElementById("month");
const memoInput = document.getElementById("memo");

let today = new Date();
let year = today.getFullYear();
let month = today.getMonth();
let selectedDate = null;

let data = JSON.parse(localStorage.getItem("trainingData")) || {};

// 年
for (let y = 2020; y <= 2030; y++) {
  let opt = document.createElement("option");
  opt.value = y;
  opt.text = y;
  if (y === year) opt.selected = true;
  yearSelect.appendChild(opt);
}

// 月
for (let m = 0; m < 12; m++) {
  let opt = document.createElement("option");
  opt.value = m;
  opt.text = m + 1;
  if (m === month) opt.selected = true;
  monthSelect.appendChild(opt);
}

yearSelect.onchange = monthSelect.onchange = () => {
  year = parseInt(yearSelect.value);
  month = parseInt(monthSelect.value);
  renderCalendar();
};

// カレンダー
function renderCalendar() {
  calendarEl.innerHTML = "";

  const firstDay = new Date(year, month, 1).getDay();
  const lastDate = new Date(year, month + 1, 0).getDate();

  for (let i = 0; i < firstDay; i++) {
    calendarEl.innerHTML += `<div></div>`;
  }

  for (let day = 1; day <= lastDate; day++) {
    const dateStr = `${year}-${month + 1}-${day}`;
    const div = document.createElement("div");

    div.className = "day";

    if (data[dateStr]) div.classList.add("has-record");
    if (dateStr === selectedDate) div.classList.add("selected");

    div.textContent = day;
    div.onclick = () => {
      selectedDate = dateStr;
      showDetail(dateStr);
      renderCalendar();
    };

    calendarEl.appendChild(div);
  }
}

// 保存
document.getElementById("saveBtn").onclick = () => {
  if (!selectedDate) {
    alert("日付を選択してください");
    return;
  }

  const checks = document.querySelectorAll("#controls input:checked");
  const selected = Array.from(checks).map(c => c.value);
  const memo = memoInput.value;

  if (selected.length === 0) {
    alert("部位を選択してください");
    return;
  }

  data[selectedDate] = {
    parts: selected,
    memo: memo
  };

  localStorage.setItem("trainingData", JSON.stringify(data));

  // リセット
  document.querySelectorAll("#controls input").forEach(c => c.checked = false);
  memoInput.value = "";

  showDetail(selectedDate);
  renderCalendar();
};

// 詳細
function showDetail(dateStr) {
  if (data[dateStr]) {
    detailText.innerHTML = `
      ${dateStr}<br>
      部位：${data[dateStr].parts.join(", ")}<br>
      メモ：${data[dateStr].memo || "なし"}
    `;
  } else {
    detailText.textContent = `${dateStr}： 記録なし`;
  }
}

renderCalendar();
</script>

</body>
</html>
