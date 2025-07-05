<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>ระบบใบเสนอราคา - บจก.แหลมทองกิจเกษตร</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', Arial, sans-serif;
      max-width: 800px;
      margin: 40px auto;
      padding: 32px 24px;
      background: #f6fafd;
      color: #222;
      border-radius: 16px;
      box-shadow: 0 4px 24px rgba(0,0,0,0.08);
    }
    h2 {
      color: #1976d2;
      text-align: center;
      margin-bottom: 32px;
      letter-spacing: 1px;
    }
    label {
      font-weight: 500;
      color: #1565c0;
      margin-bottom: 4px;
      display: inline-block;
    }
    input[type="text"], input[type="date"], input[type="number"], textarea {
      width: 100%;
      padding: 8px 12px;
      margin: 6px 0 14px 0;
      border: 1px solid #b3c6e0;
      border-radius: 6px;
      background: #fff;
      font-size: 1em;
      transition: border 0.2s;
      box-sizing: border-box;
    }
    input:focus, textarea:focus {
      border: 1.5px solid #1976d2;
      outline: none;
      background: #e3f2fd;
    }
    textarea {
      min-height: 60px;
      resize: vertical;
    }
    button {
      background: linear-gradient(90deg, #1976d2 60%, #64b5f6 100%);
      color: #fff;
      border: none;
      border-radius: 6px;
      padding: 8px 20px;
      font-size: 1em;
      font-weight: 500;
      margin: 4px 8px 4px 0;
      cursor: pointer;
      box-shadow: 0 2px 8px rgba(25, 118, 210, 0.08);
      transition: background 0.2s, box-shadow 0.2s;
    }
    button:hover {
      background: linear-gradient(90deg, #1565c0 60%, #42a5f5 100%);
      box-shadow: 0 4px 16px rgba(25, 118, 210, 0.13);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 12px;
      background: #fff;
      border-radius: 10px;
      overflow: hidden;
      box-shadow: 0 2px 8px rgba(25, 118, 210, 0.06);
    }
    th, td {
      border: 1px solid #e3eaf5;
      padding: 10px 6px;
      text-align: center;
      font-size: 1em;
    }
    th {
      background: #1976d2;
      color: #fff;
      font-weight: 600;
      letter-spacing: 0.5px;
    }
    tr:nth-child(even) td {
      background: #f0f7fa;
    }
    .total-row {
      background: #e3f2fd !important;
      color: #1976d2;
      font-weight: bold;
      font-size: 1.1em;
    }
    #totals p {
      margin: 6px 0;
      font-size: 1.05em;
    }
    @media (max-width: 600px) {
      body {
        padding: 8px 2px;
        max-width: 100vw;
      }
      table, th, td {
        font-size: 0.95em;
      }
      h2 {
        font-size: 1.2em;
      }
    }
  </style>
</head>
<body>
  <h2>ระบบใบเสนอราคา</h2>
  <form id="quoteForm">
    <!-- ข้อมูลเอกสาร -->
    <div>
      <label>เลขที่เอกสาร:</label>
      <input type="text" id="docNumber" value="OOQ680705474597" readonly><br>
      <label>วันที่ออกเอกสาร:</label>
      <input type="date" id="docDate" value="2025-05-07">
    </div>

    <!-- ข้อมูลลูกค้า -->
    <div style="margin-top: 20px;">
      <label>ชื่อลูกค้า:</label><br>
      <input type="text" id="customerName" placeholder="กรอกชื่อลูกค้า"><br>
      <label>ที่อยู่ลูกค้า:</label><br>
      <textarea id="customerAddress" placeholder="กรอกที่อยู่ลูกค้า"></textarea>
    </div>

    <!-- รายการสินค้า -->
    <div style="margin-top: 20px;">
      <button type="button" onclick="addProductRow()">เพิ่มรายการสินค้า</button>
      <table id="productTable">
        <thead>
          <tr>
            <th>รายการ</th><th>จำนวน</th><th>หน่วย</th><th>ราคา/หน่วย</th><th>ส่วนลด</th><th>จำนวนเงิน</th><th>ลบ</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <!-- ส่วนลดและการคำนวณ -->
    <div style="margin-top: 20px;">
      <label>ส่วนลดการค้า (บาท):</label>
      <input type="number" id="discount" value="0" min="0" oninput="calculateTotals()">
      <hr>
      <div id="totals">
        <p>รวมเป็นเงิน: 0.00 บาท</p>
        <p>หักส่วนลดการค้า: 0.00 บาท</p>
        <p>ยอดคงเหลือ: 0.00 บาท</p>
        <p>ภาษีมูลค่าเพิ่ม: 0.00 บาท</p>
        <p class="total-row">ยอดรวมสุทธิ: 0.00 บาท</p>
      </div>
    </div>

    <!-- ปุ่มบันทึกและรีเซ็ต -->
    <div style="margin-top: 20px;">
      <button type="submit">บันทึกและส่ง PDF ใบเสนอราคา</button>
      <button type="button" onclick="resetForm()">เริ่มใหม่</button>
    </div>
  </form>

  <script>
    // เพิ่มแถวสินค้า
    function addProductRow() {
      const table = document.getElementById('productTable').getElementsByTagName('tbody')[0];
      const row = table.insertRow();
      row.innerHTML = `
        <td><input type="text" class="product-name" placeholder="ชื่อสินค้า"></td>
        <td><input type="number" class="quantity" value="1" min="1" oninput="calculateRow(this)"></td>
        <td><input type="text" class="unit" value="ชิ้น"></td>
        <td><input type="number" class="price" value="0" oninput="calculateRow(this)"></td>
        <td><input type="number" class="row-discount" value="0" oninput="calculateRow(this)"></td>
        <td class="amount">0.00</td>
        <td><button type="button" onclick="removeRow(this)">ลบ</button></td>
      `;
    }

    // ลบแถวสินค้า
    function removeRow(btn) {
      btn.closest('tr').remove();
      calculateTotals();
    }

    // คำนวณยอดรวมของแต่ละแถว
    function calculateRow(input) {
      const row = input.closest('tr');
      const quantity = parseFloat(row.querySelector('.quantity').value) || 0;
      const price = parseFloat(row.querySelector('.price').value) || 0;
      const discount = parseFloat(row.querySelector('.row-discount').value) || 0;
      const amount = (quantity * price) - discount;
      row.querySelector('.amount').textContent = amount.toFixed(2);
      calculateTotals();
    }

    // คำนวณยอดรวมทั้งหมด
    function calculateTotals() {
      let subtotal = 0;
      document.querySelectorAll('#productTable .amount').forEach(cell => {
        subtotal += parseFloat(cell.textContent);
      });
      const discount = parseFloat(document.getElementById('discount').value) || 0;
      const tax = (subtotal - discount) * 0.07;
      const total = subtotal - discount + tax;

      document.getElementById('totals').innerHTML = `
        <p>รวมเป็นเงิน: ${subtotal.toFixed(2)} บาท</p>
        <p>หักส่วนลดการค้า: ${discount.toFixed(2)} บาท</p>
        <p>ยอดคงเหลือ: ${(subtotal - discount).toFixed(2)} บาท</p>
        <p>ภาษีมูลค่าเพิ่ม: ${tax.toFixed(2)} บาท</p>
        <p class="total-row">ยอดรวมสุทธิ: ${total.toFixed(2)} บาท</p>
      `;
    }

    // สร้าง PDF และส่งไป Telegram
    document.getElementById('quoteForm').addEventListener('submit', async function(e) {
      e.preventDefault();
      
      // ดึงข้อมูลจากฟอร์ม
      const docNumber = document.getElementById('docNumber').value;
      const docDate = document.getElementById('docDate').value;
      const customerName = document.getElementById('customerName').value;
      const customerAddress = document.getElementById('customerAddress').value;
      const discount = document.getElementById('discount').value;
      const totals = document.getElementById('totals').innerText;

      // สร้าง PDF
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      let y = 20;

      // หัวเอกสาร
      doc.setFontSize(16);
      doc.text("บจก.แหลมทองกิจเกษตร", 10, y);
      y += 10;
      doc.setFontSize(12);
      doc.text(`เลขที่เอกสาร: ${docNumber}`, 10, y);
      doc.text(`วันที่: ${docDate}`, 150, y);
      y += 10;
      doc.text(`ลูกค้า: ${customerName}`, 10, y);
      y += 10;
      doc.text(`ที่อยู่: ${customerAddress}`, 10, y);
      y += 15;

      // ตารางสินค้า
      const products = [];
      document.querySelectorAll('#productTable tbody tr').forEach(row => {
        products.push([
          row.children[0].children[0].value,
          row.children[1].children[0].value,
          row.children[2].children[0].value,
          row.children[3].children[0].value,
          row.children[4].children[0].value,
          row.children[5].textContent
        ]);
      });
      doc.autoTable({
        head: [['รายการ', 'จำนวน', 'หน่วย', 'ราคา/หน่วย', 'ส่วนลด', 'จำนวนเงิน']],
        body: products,
        startY: y
      });
      y = doc.lastAutoTable.finalY + 10;

      // ยอดรวม
      doc.text(totals, 10, y);

      // ส่ง PDF ไป Telegram
      const pdfBlob = doc.output('blob');
      const formData = new FormData();
      formData.append('chat_id', '8168649234');
      formData.append('document', pdfBlob, `${docNumber}.pdf`);

      await fetch(`https://api.telegram.org/8168894246:AAEKgqX0VLKp0LrqTB1HigIK13l8bhkDIjM/sendDocument`, {
        method: 'POST',
        body: formData
      });

      alert('ส่งใบเสนอราคาสำเร็จ!');
    });

    // รีเซ็ตฟอร์ม
    function resetForm() {
      document.getElementById('quoteForm').reset();
      document.getElementById('productTable').getElementsByTagName('tbody')[0].innerHTML = '';
      document.getElementById('totals').innerHTML = `
        <p>รวมเป็นเงิน: 0.00 บาท</p>
        <p>หักส่วนลดการค้า: 0.00 บาท</p>
        <p>ยอดคงเหลือ: 0.00 บาท</p>
        <p>ภาษีมูลค่าเพิ่ม: 0.00 บาท</p>
        <p class="total-row">ยอดรวมสุทธิ: 0.00 บาท</p>
      `;
    }
  </script>
</body>
</html>
