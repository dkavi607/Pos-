<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ProPOS | All-in-One System</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --primary: #2563eb;
            --secondary: #64748b;
            --success: #22c55e;
            --danger: #ef4444;
            --dark: #1e293b;
            --light: #f8fafc;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { background-color: #f1f5f9; display: flex; height: 100vh; overflow: hidden; }

        /* Sidebar */
        nav { width: 260px; background: var(--dark); color: white; display: flex; flex-direction: column; padding: 20px; }
        nav h2 { margin-bottom: 30px; text-align: center; color: #38bdf8; }
        nav button { 
            background: transparent; border: none; color: #cbd5e1; padding: 12px; text-align: left; 
            font-size: 16px; cursor: pointer; border-radius: 8px; margin-bottom: 5px; transition: 0.3s;
        }
        nav button:hover, nav button.active { background: var(--primary); color: white; }
        nav button i { margin-right: 10px; width: 20px; }

        /* Main Content */
        main { flex: 1; padding: 30px; overflow-y: auto; }
        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        /* Dashboard Cards */
        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 30px; }
        .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
        .card h3 { color: var(--secondary); font-size: 14px; text-transform: uppercase; }
        .card p { font-size: 24px; font-weight: bold; color: var(--dark); }

        /* Forms & Tables */
        .form-group { margin-bottom: 15px; }
        input, select { width: 100%; padding: 10px; margin-top: 5px; border: 1px solid #ddd; border-radius: 6px; }
        button.btn-main { background: var(--primary); color: white; border: none; padding: 10px 20px; border-radius: 6px; cursor: pointer; }
        
        table { width: 100%; border-collapse: collapse; background: white; border-radius: 12px; overflow: hidden; margin-top: 20px; }
        th, td { padding: 15px; text-align: left; border-bottom: 1px solid #eee; }
        th { background: #f8fafc; color: var(--secondary); }

        /* POS UI */
        .pos-container { display: grid; grid-template-columns: 2fr 1fr; gap: 20px; }
        .cart-area { background: white; padding: 20px; border-radius: 12px; height: fit-content; }
        .product-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 10px; }
        .prod-item { background: white; padding: 15px; border: 1px solid #ddd; border-radius: 8px; text-align: center; cursor: pointer; transition: 0.2s; }
        .prod-item:hover { border-color: var(--primary); transform: translateY(-2px); }

        /* Print Bill Style */
        @media print {
            body * { visibility: hidden; }
            #bill-receipt, #bill-receipt * { visibility: visible; }
            #bill-receipt { position: absolute; left: 0; top: 0; width: 100%; padding: 20px; }
        }
    </style>
</head>
<body>

    <nav>
        <h2>ProPOS v1.0</h2>
        <button onclick="showSection('dashboard')" class="active"><i class="fas fa-chart-line"></i> Dashboard</button>
        <button onclick="showSection('inventory')"><i class="fas fa-boxes"></i> Inventory</button>
        <button onclick="showSection('customers')"><i class="fas fa-users"></i> Customers</button>
        <button onclick="showSection('pos')"><i class="fas fa-shopping-cart"></i> POS / Sales</button>
        <button onclick="showSection('history')"><i class="fas fa-history"></i> Sales History</button>
        <button onclick="showSection('wage')"><i class="fas fa-hand-holding-usd"></i> Wage Advance</button>
    </nav>

    <main>
        <!-- Dashboard -->
        <div id="dashboard" class="section active">
            <h1>Dashboard</h1>
            <div class="stats-grid">
                <div class="card"><h3>Total Sales</h3><p id="stat-sales">LKR 0</p></div>
                <div class="card"><h3>Products</h3><p id="stat-products">0</p></div>
                <div class="card"><h3>Customers</h3><p id="stat-customers">0</p></div>
                <div class="card"><h3>Total Advances</h3><p id="stat-advances">LKR 0</p></div>
            </div>
        </div>

        <!-- Inventory -->
        <div id="inventory" class="section">
            <h1>Product Management</h1>
            <div class="card" style="margin-top: 20px;">
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="p-name" placeholder="Product Name">
                    <input type="number" id="p-price" placeholder="Price">
                    <input type="number" id="p-stock" placeholder="Stock Qty">
                    <button class="btn-main" onclick="addProduct()">Add Product</button>
                </div>
            </div>
            <table id="product-table">
                <thead><tr><th>ID</th><th>Name</th><th>Price</th><th>Stock</th><th>Action</th></tr></thead>
                <tbody></tbody>
            </table>
        </div>

        <!-- Customers -->
        <div id="customers" class="section">
            <h1>Customer Management</h1>
            <div class="card" style="margin-top: 20px;">
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="c-name" placeholder="Customer Name">
                    <input type="text" id="c-phone" placeholder="Phone Number">
                    <button class="btn-main" onclick="addCustomer()">Add Customer</button>
                </div>
            </div>
            <table id="customer-table">
                <thead><tr><th>ID</th><th>Name</th><th>Phone</th><th>Action</th></tr></thead>
                <tbody></tbody>
            </table>
        </div>

        <!-- POS -->
        <div id="pos" class="section">
            <h1>Point of Sale</h1>
            <div class="pos-container">
                <div>
                    <h3>Select Products</h3>
                    <div class="product-grid" id="pos-product-grid"></div>
                </div>
                <div class="cart-area">
                    <h3>Current Bill</h3>
                    <select id="pos-customer-select" style="margin-bottom: 10px;"><option value="Guest">Guest Customer</option></select>
                    <div id="cart-items"></div>
                    <hr>
                    <h2 style="margin: 15px 0;">Total: <span id="cart-total">0.00</span></h2>
                    <button class="btn-main" style="width: 100%; background: var(--success);" onclick="checkout()">Complete Sale & Print</button>
                </div>
            </div>
        </div>

        <!-- Sales History -->
        <div id="history" class="section">
            <h1>Sales History</h1>
            <table id="history-table">
                <thead><tr><th>Date</th><th>Customer</th><th>Total</th><th>Action</th></tr></thead>
                <tbody></tbody>
            </table>
        </div>

        <!-- Wage Advance -->
        <div id="wage" class="section">
            <h1>Wage Advance Management</h1>
            <div class="card" style="margin-top: 20px;">
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="w-name" placeholder="Staff Name">
                    <input type="number" id="w-amount" placeholder="Advance Amount">
                    <button class="btn-main" onclick="addAdvance()">Record Advance</button>
                </div>
            </div>
            <table id="advance-table">
                <thead><tr><th>Date</th><th>Staff Name</th><th>Amount</th></tr></thead>
                <tbody></tbody>
            </table>
        </div>
    </main>

    <!-- Hidden Bill Template -->
    <div id="bill-receipt" style="display: none; background: white; color: black; font-family: monospace;">
        <center>
            <h2>STORE NAME</h2>
            <p>123 Business Road, City</p>
            <p>Tel: 0112-345678</p>
            <hr>
        </center>
        <p>Date: <span id="bill-date"></span></p>
        <p>Customer: <span id="bill-cust"></span></p>
        <hr>
        <div id="bill-items"></div>
        <hr>
        <h3>Total: <span id="bill-total"></span></h3>
        <center><p>Thank you for your business!</p></center>
    </div>

    <script>
        // Database Simulation
        let products = JSON.parse(localStorage.getItem('products')) || [];
        let customers = JSON.parse(localStorage.getItem('customers')) || [];
        let sales = JSON.parse(localStorage.getItem('sales')) || [];
        let advances = JSON.parse(localStorage.getItem('advances')) || [];
        let cart = [];

        // Navigation
        function showSection(id) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            event.currentTarget.classList.add('active');
            renderAll();
        }

        // --- Product Logic ---
        function addProduct() {
            const name = document.getElementById('p-name').value;
            const price = document.getElementById('p-price').value;
            const stock = document.getElementById('p-stock').value;
            if(!name || !price) return alert("Fill details");
            products.push({ id: Date.now(), name, price: parseFloat(price), stock: parseInt(stock) });
            saveAndRefresh();
        }

        function deleteProduct(id) {
            products = products.filter(p => p.id !== id);
            saveAndRefresh();
        }

        // --- Customer Logic ---
        function addCustomer() {
            const name = document.getElementById('c-name').value;
            const phone = document.getElementById('c-phone').value;
            if(!name) return alert("Fill name");
            customers.push({ id: Date.now(), name, phone });
            saveAndRefresh();
        }

        // --- Wage Advance Logic ---
        function addAdvance() {
            const name = document.getElementById('w-name').value;
            const amount = document.getElementById('w-amount').value;
            if(!name || !amount) return alert("Fill details");
            advances.push({ date: new Date().toLocaleDateString(), name, amount: parseFloat(amount) });
            saveAndRefresh();
        }

        // --- POS Logic ---
        function addToCart(prodId) {
            const prod = products.find(p => p.id === prodId);
            if(prod.stock <= 0) return alert("Out of Stock!");
            
            const existing = cart.find(item => item.id === prodId);
            if(existing) {
                existing.qty++;
            } else {
                cart.push({ ...prod, qty: 1 });
            }
            renderCart();
        }

        function renderCart() {
            const container = document.getElementById('cart-items');
            container.innerHTML = '';
            let total = 0;
            cart.forEach(item => {
                total += (item.price * item.qty);
                container.innerHTML += `<div style="display:flex; justify-content:space-between; margin-bottom:5px;">
                    <span>${item.name} x${item.qty}</span>
                    <span>${(item.price * item.qty).toFixed(2)}</span>
                </div>`;
            });
            document.getElementById('cart-total').innerText = total.toFixed(2);
        }

        function checkout() {
            if(cart.length === 0) return alert("Cart is empty");
            const total = cart.reduce((sum, item) => sum + (item.price * item.qty), 0);
            const customer = document.getElementById('pos-customer-select').value;
            
            const sale = {
                id: Date.now(),
                date: new Date().toLocaleString(),
                customer,
                items: [...cart],
                total
            };

            // Update Stock
            cart.forEach(item => {
                const p = products.find(prod => prod.id === item.id);
                if(p) p.stock -= item.qty;
            });

            sales.push(sale);
            printBill(sale);
            cart = [];
            saveAndRefresh();
        }

        function printBill(sale) {
            document.getElementById('bill-date').innerText = sale.date;
            document.getElementById('bill-cust').innerText = sale.customer;
            document.getElementById('bill-total').innerText = sale.total.toFixed(2);
            
            let itemHtml = '';
            sale.items.forEach(i => {
                itemHtml += `<p>${i.name} x${i.qty} = ${(i.price * i.qty).toFixed(2)}</p>`;
            });
            document.getElementById('bill-items').innerHTML = itemHtml;
            
            window.print();
        }

        // --- Rendering Helpers ---
        function saveAndRefresh() {
            localStorage.setItem('products', JSON.stringify(products));
            localStorage.setItem('customers', JSON.stringify(customers));
            localStorage.setItem('sales', JSON.stringify(sales));
            localStorage.setItem('advances', JSON.stringify(advances));
            renderAll();
        }

        function renderAll() {
            // Dashboard
            document.getElementById('stat-sales').innerText = "LKR " + sales.reduce((a, b) => a + b.total, 0).toFixed(2);
            document.getElementById('stat-products').innerText = products.length;
            document.getElementById('stat-customers').innerText = customers.length;
            document.getElementById('stat-advances').innerText = "LKR " + advances.reduce((a, b) => a + b.amount, 0).toFixed(2);

            // Product Table
            const pTable = document.querySelector('#product-table tbody');
            pTable.innerHTML = products.map(p => `<tr>
                <td>${p.id}</td><td>${p.name}</td><td>${p.price}</td><td>${p.stock}</td>
                <td><button onclick="deleteProduct(${p.id})" style="color:red; border:none; cursor:pointer;"><i class="fas fa-trash"></i></button></td>
            </tr>`).join('');

            // Customer Table & POS Select
            const cTable = document.querySelector('#customer-table tbody');
            cTable.innerHTML = customers.map(c => `<tr><td>${c.id}</td><td>${c.name}</td><td>${c.phone}</td><td>Delete</td></tr>`).join('');
            
            const cSelect = document.getElementById('pos-customer-select');
            cSelect.innerHTML = '<option value="Guest">Guest Customer</option>' + customers.map(c => `<option value="${c.name}">${c.name}</option>`).join('');

            // POS Grid
            const posGrid = document.getElementById('pos-product-grid');
            posGrid.innerHTML = products.map(p => `<div class="prod-item" onclick="addToCart(${p.id})">
                <strong>${p.name}</strong><br><small>LKR ${p.price}</small><br><small>Stock: ${p.stock}</small>
            </div>`).join('');

            // History Table
            const hTable = document.querySelector('#history-table tbody');
            hTable.innerHTML = sales.map(s => `<tr>
                <td>${s.date}</td><td>${s.customer}</td><td>${s.total.toFixed(2)}</td>
                <td><button onclick='alert("Bill Data: "+ JSON.stringify(this.sale))' class="btn-main" style="padding:5px 10px">View</button></td>
            </tr>`).join('');

            // Advance Table
            const aTable = document.querySelector('#advance-table tbody');
            aTable.innerHTML = advances.map(a => `<tr><td>${a.date}</td><td>${a.name}</td><td>${a.amount}</td></tr>`).join('');
        }

        // Initial Load
        renderAll();
    </script>
</body>
</html>
