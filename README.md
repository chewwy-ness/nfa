<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>INVENTORY NFA CHECKLIST</title>
  <style>
    body { font-family: Arial, sans-serif; background: #f4f4f4; padding: 20px; }
    .container { max-width: 1200px; margin: auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }
    h1 { text-align: center; }
    .toolbar { text-align: center; margin-bottom: 20px; }
    .toolbar input[type="text"] { padding: 10px; width: 50%; font-size: 16px; margin-right: 10px; }
    .toolbar button { padding: 8px 15px; margin: 5px; background: #007BFF; color: white; border: none; border-radius: 5px; cursor: pointer; }
    .toolbar button:hover { background: #0056b3; }
    table { width: 100%; border-collapse: collapse; margin-top: 20px; }
    th, td { border: 1px solid #ccc; padding: 8px; text-align: center; }
    th { background-color: #007BFF; color: white; cursor: pointer; }
    tr:nth-child(even) { background-color: #f9f9f9; }
    input[type="text"], textarea { width: 95%; padding: 5px; }
  </style>
</head>
<body>
  <div class="container">
    <h1>INVENTORY NFA CHECKLIST</h1>
    <div class="toolbar">
      <input type="text" id="searchInput" placeholder="Search items..." onkeyup="filterTable()">
      <button onclick="saveProgress()">Save Progress</button>
      <button onclick="document.getElementById('fileInput').click()">Load Progress</button>
      <input type="file" id="fileInput" style="display:none" onchange="loadProgress(event)">
    </div>
    <table id="inventoryTable">
      <thead>
        <tr>
          <th onclick="sortTable(0)">Name of Item</th>
          <th onclick="sortTable(1)">Description</th>
          <th onclick="sortTable(2)">Stock Number</th>
          <th onclick="sortTable(3)">Unit of Measure</th>
          <th onclick="sortTable(4)">Unit Value</th>
          <th onclick="sortTable(5)">Balance per Card (Qty)</th>
          <th onclick="sortTable(6)">On Hand per Count (Qty)</th>
          <th onclick="sortTable(7)">Shortage (Qty)</th>
          <th onclick="sortTable(8)">Overage (Qty)</th>
          <th onclick="sortTable(9)">Remarks</th>
          <th>Check</th>
        </tr>
      </thead>
      <tbody id="inventoryBody">
        <tr><td>Rice Bag</td><td>25kg Premium Rice</td><td>ITEM001</td><td>Bag</td><td>1200</td><td>100</td><td>98</td><td>2</td><td>0</td><td><input type="text" value="Missing 2 bags"></td><td><input type="checkbox"></td></tr>
        <tr><td>Corn Bag</td><td>50kg Yellow Corn</td><td>ITEM002</td><td>Bag</td><td>900</td><td>200</td><td>200</td><td>0</td><td>0</td><td><input type="text" value="All good"></td><td><input type="checkbox"></td></tr>
        <tr><td>Flour Sack</td><td>50kg Baking Flour</td><td>ITEM003</td><td>Sack</td><td>1000</td><td>150</td><td>149</td><td>1</td><td>0</td><td><input type="text" value="Missing 1 sack"></td><td><input type="checkbox"></td></tr>
        <tr><td>Sugar Sack</td><td>50kg Refined Sugar</td><td>ITEM004</td><td>Sack</td><td>1200</td><td>100</td><td>100</td><td>0</td><td>0</td><td><input type="text" value="All good"></td><td><input type="checkbox"></td></tr>
        <tr><td>Canned Tuna</td><td>155g Canned Tuna</td><td>ITEM005</td><td>Can</td><td>35</td><td>500</td><td>498</td><td>2</td><td>0</td><td><input type="text" value="Missing 2 cans"></td><td><input type="checkbox"></td></tr>
      </tbody>
    </table>
  </div>

  <script>
    // Search filter
    function filterTable() {
      const query = document.getElementById("searchInput").value.toLowerCase();
      const rows = document.querySelectorAll("#inventoryBody tr");
      rows.forEach(row => {
        const text = row.innerText.toLowerCase();
        row.style.display = text.includes(query) ? "" : "none";
      });
    }

    // Sort table
    function sortTable(n) {
      const table = document.getElementById("inventoryTable");
      let rows = Array.from(table.rows).slice(1);
      let switching = true;
      let dir = "asc";
      let switchcount = 0;

      while (switching) {
        switching = false;
        for (let i = 0; i < rows.length - 1; i++) {
          let x = rows[i].getElementsByTagName("TD")[n];
          let y = rows[i + 1].getElementsByTagName("TD")[n];
          let shouldSwitch = false;
          if (dir === "asc" && x.innerHTML.toLowerCase() > y.innerHTML.toLowerCase()) {
            shouldSwitch = true;
          } else if (dir === "desc" && x.innerHTML.toLowerCase() < y.innerHTML.toLowerCase()) {
            shouldSwitch = true;
          }
          if (shouldSwitch) {
            rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
            switching = true;
            switchcount++;
          }
        }
        if (switchcount === 0 && dir === "asc") {
          dir = "desc";
          switching = true;
        }
      }
    }

    // Save progress
    function saveProgress() {
      const rows = [];
      document.querySelectorAll("#inventoryBody tr").forEach(tr => {
        const row = [];
        tr.querySelectorAll("td").forEach((td, i) => {
          if (i === 9) row.push(td.querySelector("input").value);
          else if (i === 10) row.push(td.querySelector("input").checked ? "Checked" : "Unchecked");
          else row.push(td.innerText);
        });
        rows.push(row);
      });
      const blob = new Blob([JSON.stringify(rows, null, 2)], { type: "application/json" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "inventory-progress.json";
      link.click();
    }

    // Load progress
    function loadProgress(event) {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        const data = JSON.parse(e.target.result);
        const rows = document.querySelectorAll("#inventoryBody tr");
        data.forEach((rowData, index) => {
          if (rows[index]) {
            rows[index].querySelectorAll("td").forEach((td, i) => {
              if (i === 9) td.querySelector("input").value = rowData[i];
              if (i === 10) td.querySelector("input").checked = rowData[i] === "Checked";
            });
          }
        });
      };
      reader.readAsText(file);
    }

    // Auto-search via QR code parameter
    window.onload = function() {
      const urlParams = new URLSearchParams(window.location.search);
      const search = urlParams.get('search');
      if (search) {
        document.getElementById('searchInput').value = search;
        filterTable();
      }
    };
  </script>
</body>
</html>

