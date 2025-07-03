<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Jio Recharge Tracker</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
</head>
<body class="bg-gray-100 p-4 font-sans">
  <div class="max-w-xl mx-auto bg-white p-6 rounded-2xl shadow-lg">
    <h2 class="text-2xl font-bold text-center text-blue-600 mb-4">📱 Jio Recharge Tracker</h2>

    <div class="space-y-4">
      <input id="name" type="text" placeholder="Customer Name" class="w-full border p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400">
      <input id="number" type="tel" placeholder="Mobile Number" class="w-full border p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400">

      <select id="plan" class="w-full border p-2 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400">
        <option value="28">1 Month (28 days)</option>
        <option value="56">2 Months (56 days)</option>
        <option value="84">3 Months (84 days)</option>
        <option value="112">4 Months (112 days)</option>
        <option value="50">50 Days</option>
      </select>

      <button onclick="saveCustomer()" class="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 transition">Save</button>
    </div>
  </div>

  <div class="max-w-2xl mx-auto mt-6">
    <div class="flex justify-between items-center mb-3">
      <h3 class="text-xl font-semibold text-gray-700 text-center w-full">📋 Saved Customers</h3>
    </div>

    <div class="text-right mb-4">
      <button onclick="exportPDF()" class="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition">
        📄 Export to PDF
      </button>
    </div>

    <div id="customerList" class="space-y-3 bg-white p-4 rounded-xl shadow"></div>
  </div>

  <script>
    function saveCustomer() {
      const name = document.getElementById('name').value.trim();
      const number = document.getElementById('number').value.trim();
      const planDays = parseInt(document.getElementById('plan').value);

      if (!name || !number) {
        alert("Please enter both name and number.");
        return;
      }

      const today = new Date();
      const dueDate = new Date(today);
      dueDate.setDate(dueDate.getDate() + planDays);

      const customer = {
        id: Date.now(),
        name,
        number,
        plan: planDays + ' Days',
        rechargeDate: today.toISOString().split('T')[0],
        dueDate: dueDate.toISOString().split('T')[0],
        calledAt: null
      };

      let customers = JSON.parse(localStorage.getItem('customers')) || [];
      customers.push(customer);
      localStorage.setItem('customers', JSON.stringify(customers));
      displayCustomers();

      document.getElementById('name').value = '';
      document.getElementById('number').value = '';
    }

    function deleteCustomer(id) {
      let customers = JSON.parse(localStorage.getItem('customers')) || [];
      customers = customers.filter(c => c.id !== id);
      localStorage.setItem('customers', JSON.stringify(customers));
      displayCustomers();
    }

    function sendWhatsApp(number) {
      const message = encodeURIComponent(`Recharge_${number}`);
      const url = `https://wa.me/917001170011?text=${message}`;
      window.location.href = url;
    }

    function markCalled(id) {
      let customers = JSON.parse(localStorage.getItem('customers')) || [];
      const index = customers.findIndex(c => c.id === id);
      if (index !== -1) {
        customers[index].calledAt = new Date().toLocaleTimeString();
        localStorage.setItem('customers', JSON.stringify(customers));
        displayCustomers();
      }
    }

    function displayCustomers() {
      const customers = JSON.parse(localStorage.getItem('customers')) || [];
      const container = document.getElementById('customerList');
      container.innerHTML = '';

      if (customers.length === 0) {
        container.innerHTML = '<p class="text-center text-gray-500">No customers added yet.</p>';
        return;
      }

      const today = new Date();
      today.setHours(0, 0, 0, 0);

      customers.forEach(c => {
        const due = new Date(c.dueDate);
        due.setHours(0, 0, 0, 0);

        const msPerDay = 1000 * 60 * 60 * 24;
        const daysLeft = Math.floor((due - today) / msPerDay);

        let daysText = '';
        let dueClass = '';
        let nameClass = '';

        if (daysLeft < 0) {
          daysText = `<span class="text-red-600 font-semibold">Expired ${Math.abs(daysLeft)} day(s) ago</span>`;
          dueClass = 'text-red-600 font-bold';
          nameClass = 'text-red-600';
        } else if (daysLeft === 0) {
          daysText = `<span class="text-yellow-600 font-semibold">Due Today</span>`;
          dueClass = 'text-yellow-600 font-bold';
          nameClass = 'text-blue-600';
        } else {
          daysText = `<span class="text-green-600 font-semibold">${daysLeft} day(s) left</span>`;
          dueClass = 'text-gray-700';
          nameClass = 'text-blue-600';
        }

        const calledTime = c.calledAt
          ? `<p class="text-sm text-gray-600 mt-1">📞 Called at ${c.calledAt}</p>`
          : '';

        const callButtonClass = c.calledAt
          ? 'bg-gray-400 cursor-not-allowed pointer-events-none'
          : 'bg-green-600 hover:bg-green-700';

        const callButtonAction = c.calledAt
          ? 'javascript:void(0)'
          : `tel:${c.number}" onclick="markCalled(${c.id})`;

        container.innerHTML += `
          <div class="bg-white p-4 rounded-xl shadow border border-gray-200">
            <div class="flex flex-col sm:flex-row sm:justify-between items-start sm:items-center">
              <div>
                <p class="font-bold text-lg ${nameClass}">${c.name}</p>
                <p class="text-sm text-gray-600">📞 ${c.number}</p>
                <p class="text-sm text-gray-600">Plan: ${c.plan}</p>
              </div>
              <div class="mt-2 sm:mt-0 text-right">
                <p class="text-sm">Recharge: <span class="font-medium">${c.rechargeDate}</span></p>
                <p class="text-sm ${dueClass}">Due: <span class="font-medium">${c.dueDate}</span></p>
                <p class="text-sm mt-1">${daysText}</p>
              </div>
            </div>
            ${calledTime}
            <div class="mt-3 flex flex-wrap gap-2 justify-end">
              <button onclick="sendWhatsApp('${c.number}')" class="text-xs px-3 py-1 bg-indigo-600 text-white rounded hover:bg-indigo-700">📩 Send Link</button>
              <a href="${callButtonAction}" class="text-xs px-3 py-1 text-white rounded ${callButtonClass}">📞 Call</a>
              <button onclick="deleteCustomer(${c.id})" class="text-xs px-3 py-1 bg-red-600 text-white rounded hover:bg-red-700">🗑 Delete</button>
            </div>
          </div>`;
      });
    }

    async function exportPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      const element = document.getElementById("customerList");

      await html2canvas(element).then(canvas => {
        const imgData = canvas.toDataURL('image/png');
        const imgProps = doc.getImageProperties(imgData);
        const pdfWidth = doc.internal.pageSize.getWidth();
        const pdfHeight = (imgProps.height * pdfWidth) / imgProps.width;
        doc.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
        doc.save("JioRechargeCustomers.pdf");
      });
    }

    displayCustomers();
  </script>
</body>
</html>
