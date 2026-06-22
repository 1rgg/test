# ????????????????????

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** ?????ß›????????????????????????????????????????????????????????Óï

**Architecture:** ??????????®π???????Tab?ß›????"???¶Ã???"??"??????"??????Óï???????localStorage?????õ•?????ß€?????index.html???????????

**Tech Stack:** HTML5, CSS3, JavaScript (ES6+), localStorage

---

## ?????

| ??? | ???? | ??? |
|------|------|------|
| `index.html` | ??? | ????????????????Tab?ß›???????????? |

---

## Task 1: ????Tab?ß›?UI??

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ??????°§?????Tab????HTML**

?? `<h2 class="title">? ??????????????</h2>` ???`<!-- ???????? -->` ??????

```html
        <!-- Tab???? -->
        <div class="tab-nav">
            <button class="tab-btn active" data-tab="single">???¶Ã???</button>
            <button class="tab-btn" data-tab="portfolio">??????</button>
        </div>
```

- [ ] **Step 2: ????Tab????????**

?????ß÷? `<!-- ???????? -->` ?? `<button class="btn-calc"` ??????????????????

```html
        <!-- ???¶Ã???Tab???? -->
        <div class="tab-content active" id="singleTab">
            <!-- ???ß÷????????????????????? -->
        </div>
```

- [ ] **Step 3: ?????????Tab?????¶À**

????¶Ã???Tab????????????

```html
        <!-- ??????Tab???? -->
        <div class="tab-content" id="portfolioTab">
            <p style="text-align:center;color:#999;padding:50px 0;">???????????????...</p>
        </div>
```

- [ ] **Step 4: ????Tab?ß›?CSS???**

?? `</style>` ??????????

```css
        .tab-nav {
            display: flex;
            gap: 0;
            margin-bottom: 25px;
            border-bottom: 2px solid #eee;
        }

        .tab-btn {
            flex: 1;
            padding: 12px 20px;
            border: none;
            background: none;
            font-size: 16px;
            color: #7f8c8d;
            cursor: pointer;
            border-bottom: 2px solid transparent;
            margin-bottom: -2px;
            transition: all 0.3s;
        }

        .tab-btn:hover {
            color: #27ae60;
        }

        .tab-btn.active {
            color: #27ae60;
            border-bottom-color: #27ae60;
            font-weight: 600;
        }

        .tab-content {
            display: none;
        }

        .tab-content.active {
            display: block;
        }
```

- [ ] **Step 5: ????Tab?ß›?JavaScript???**

?? `<script>` ?????????????

```javascript
        // Tab?ß›?????
        const tabBtns = document.querySelectorAll('.tab-btn');
        const tabContents = document.querySelectorAll('.tab-content');

        tabBtns.forEach(btn => {
            btn.addEventListener('click', function() {
                const tabId = this.dataset.tab;

                // ?ß›??????
                tabBtns.forEach(b => b.classList.remove('active'));
                this.classList.add('active');

                // ?ß›????????
                tabContents.forEach(content => {
                    content.classList.remove('active');
                    if (content.id === tabId + 'Tab') {
                        content.classList.add('active');
                    }
                });
            });
        });
```

- [ ] **Step 6: ???Tab?ß›?????**

????????ß’?index.html?????????Tab?????????ß›???????

---

## Task 2: ???????õ•???

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ????????õ•???JavaScript????**

??Tab?ß›?????????????

```javascript
        // ==================== ????õ•??? ====================
        const STORAGE_KEY = 'goldTradeData';

        // ???????????
        function getData() {
            const data = localStorage.getItem(STORAGE_KEY);
            if (data) {
                return JSON.parse(data);
            }
            return { buyRecords: [], sellRecords: [] };
        }

        // ????????????
        function saveData(data) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
        }

        // ????¶∑?ID
        function generateId() {
            return Date.now().toString(36) + Math.random().toString(36).substr(2);
        }

        // ????????
        function formatTime(date) {
            const d = new Date(date);
            const year = d.getFullYear();
            const month = String(d.getMonth() + 1).padStart(2, '0');
            const day = String(d.getDate()).padStart(2, '0');
            const hour = String(d.getHours()).padStart(2, '0');
            const minute = String(d.getMinutes()).padStart(2, '0');
            return `${year}-${month}-${day} ${hour}:${minute}`;
        }

        // ??????????
        function addBuyRecord(price, weight, feeRate) {
            const data = getData();
            const amount = price * weight;
            const fee = amount * feeRate / 100;

            const record = {
                id: generateId(),
                timestamp: new Date().toISOString(),
                price: parseFloat(price),
                weight: parseFloat(weight),
                amount: amount,
                feeRate: parseFloat(feeRate),
                fee: fee,
                remainingWeight: parseFloat(weight)
            };

            data.buyRecords.push(record);
            saveData(data);
            return record;
        }

        // ?????????
        function deleteBuyRecord(id) {
            const data = getData();
            data.buyRecords = data.buyRecords.filter(r => r.id !== id);
            saveData(data);
        }

        // ?????????????
        function clearAllBuyRecords() {
            const data = getData();
            data.buyRecords = [];
            data.sellRecords = [];
            saveData(data);
        }

        // ???????????????????
        function addSellRecord(price, weight, feeRate) {
            const data = getData();
            const amount = price * weight;
            const fee = amount * feeRate / 100;

            // ??FIFO?????????????????
            let remainingSell = parseFloat(weight);
            for (let record of data.buyRecords) {
                if (remainingSell <= 0) break;
                if (record.remainingWeight > 0) {
                    const deduct = Math.min(remainingSell, record.remainingWeight);
                    record.remainingWeight -= deduct;
                    remainingSell -= deduct;
                }
            }

            const sellRecord = {
                id: generateId(),
                timestamp: new Date().toISOString(),
                price: parseFloat(price),
                weight: parseFloat(weight),
                amount: amount,
                feeRate: parseFloat(feeRate),
                fee: fee
            };

            data.sellRecords.push(sellRecord);
            saveData(data);
            return sellRecord;
        }

        // ?????????
        function getPortfolioStats() {
            const data = getData();
            let totalWeight = 0;
            let totalAmount = 0;
            let totalFee = 0;

            data.buyRecords.forEach(record => {
                if (record.remainingWeight > 0) {
                    totalWeight += record.remainingWeight;
                    // ?????????????????????
                    const ratio = record.remainingWeight / record.weight;
                    totalAmount += record.amount * ratio;
                    totalFee += record.fee * ratio;
                }
            });

            const totalInvestment = totalAmount + totalFee;
            const avgCost = totalWeight > 0 ? totalInvestment / totalWeight : 0;

            return {
                totalWeight: totalWeight,
                totalAmount: totalAmount,
                totalFee: totalFee,
                totalInvestment: totalInvestment,
                avgCost: avgCost
            };
        }

        // ????????????
        function calculateBreakEvenPrice(avgCost, sellFeeRate) {
            if (avgCost <= 0) return 0;
            return avgCost / (1 - sellFeeRate / 100);
        }
```

---

## Task 3: ????????Tab UI - ???????????

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ?????????Tab??HTML??**

??Task 1?ß÷??¶À?????ùI???

```html
        <!-- ??????Tab???? -->
        <div class="tab-content" id="portfolioTab">
            <!-- ???????? -->
            <div class="stats-card" id="statsCard">
                <h3 class="card-title">? ??????</h3>
                <div class="stats-grid">
                    <div class="stat-item">
                        <span class="stat-label">???????</span>
                        <span class="stat-value" id="statTotalWeight">0.00 ??</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-label">???????</span>
                        <span class="stat-value" id="statTotalInvestment">0.00 ?</span>
                    </div>
                    <div class="stat-item">
                        <span class="stat-label">?????????</span>
                        <span class="stat-value" id="statAvgCost">0.00 ?/??</span>
                    </div>
                </div>
                <div class="break-even-box" id="breakEvenBox">
                    ? ???????????<span id="breakEvenPrice">0.00</span> ?/??
                </div>
            </div>

            <!-- ??????????? -->
            <div class="form-card">
                <h3 class="card-title">? ????????</h3>
                <div class="form-row">
                    <div class="form-group">
                        <label>???????/???</label>
                        <input type="number" id="buyPriceInput" placeholder="????????????" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>????????????</label>
                        <input type="number" id="buyWeightInput" placeholder="?????????????" step="0.01">
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>?????????????%??</label>
                        <input type="number" id="buyFeeRateInput" placeholder="???0.1%" value="0.1" step="0.01">
                    </div>
                    <div class="form-group preview-group">
                        <label>?????????</label>
                        <div class="preview-value" id="previewBuyAmount">0.00 ?</div>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group preview-group">
                        <label>?????????</label>
                        <div class="preview-value" id="previewBuyFee">0.00 ?</div>
                    </div>
                    <div class="form-group">
                        <button class="btn-calc" onclick="handleAddBuy()">???????</button>
                    </div>
                </div>
            </div>

            <!-- ???????????? -->
            <div class="form-card">
                <h3 class="card-title">? ????????</h3>
                <div class="sell-type-group">
                    <label class="radio-label">
                        <input type="radio" name="sellType" value="weight" checked> ??????
                    </label>
                    <label class="radio-label">
                        <input type="radio" name="sellType" value="amount"> ?????
                    </label>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>????????/???</label>
                        <input type="number" id="sellPriceInput" placeholder="??????????????" step="0.01">
                    </div>
                    <div class="form-group">
                        <label id="sellQuantityLabel">?????????????</label>
                        <input type="number" id="sellQuantityInput" placeholder="??????????????" step="0.01">
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>?????????????%??</label>
                        <input type="number" id="sellFeeRateInput" placeholder="???0.1%" value="0.1" step="0.01">
                    </div>
                    <div class="form-group preview-group">
                        <label>??????????</label>
                        <div class="preview-value" id="previewSellAmount">0.00 ?</div>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group preview-group">
                        <label>?????????</label>
                        <div class="preview-value" id="previewSellFee">0.00 ?</div>
                    </div>
                    <div class="form-group preview-group">
                        <label>????????</label>
                        <div class="preview-value" id="previewNetProfit">0.00 ?</div>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <button class="btn-calc btn-sell" onclick="handleSell()">???????</button>
                    </div>
                </div>
            </div>

            <!-- ???????ß“? -->
            <div class="record-card">
                <div class="record-header">
                    <h3 class="card-title">? ??????</h3>
                    <button class="btn-clear" onclick="handleClearAll()">??????</button>
                </div>
                <div class="table-container">
                    <table class="record-table" id="buyRecordsTable">
                        <thead>
                            <tr>
                                <th>???</th>
                                <th>????</th>
                                <th>????</th>
                                <th>???</th>
                                <th>??????</th>
                                <th>???</th>
                                <th>????</th>
                            </tr>
                        </thead>
                        <tbody id="buyRecordsBody">
                            <!-- ??????? -->
                        </tbody>
                        <tfoot id="buyRecordsFoot">
                            <!-- ?????????? -->
                        </tfoot>
                    </table>
                </div>
                <div class="empty-tip" id="emptyTip">??????????</div>
            </div>
        </div>
```

- [ ] **Step 2: ????????????CSS???**

?? `</style>` ??????????

```css
        /* ??????Tab??? */
        .stats-card {
            background: linear-gradient(135deg, #27ae60 0%, #2ecc71 100%);
            color: white;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
        }

        .stats-card .card-title {
            margin-bottom: 15px;
            font-size: 18px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }

        .stat-item {
            text-align: center;
        }

        .stat-label {
            display: block;
            font-size: 12px;
            opacity: 0.9;
            margin-bottom: 5px;
        }

        .stat-value {
            display: block;
            font-size: 18px;
            font-weight: bold;
        }

        .break-even-box {
            margin-top: 15px;
            padding: 12px;
            background: rgba(255,255,255,0.2);
            border-radius: 8px;
            text-align: center;
            font-size: 16px;
        }

        .form-card {
            background: white;
            padding: 20px;
            border-radius: 12px;
            margin-bottom: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        }

        .card-title {
            font-size: 16px;
            color: #2c3e50;
            margin-bottom: 15px;
        }

        .form-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 15px;
        }

        .preview-group {
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
        }

        .preview-value {
            padding: 12px 0;
            font-size: 16px;
            font-weight: 600;
            color: #27ae60;
        }

        .sell-type-group {
            display: flex;
            gap: 20px;
            margin-bottom: 15px;
        }

        .radio-label {
            display: flex;
            align-items: center;
            gap: 5px;
            cursor: pointer;
            color: #34495e;
        }

        .btn-sell {
            background-color: #e74c3c;
        }

        .btn-sell:hover {
            background-color: #c0392b;
        }

        .record-card {
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
        }

        .record-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .record-header .card-title {
            margin-bottom: 0;
        }

        .btn-clear {
            padding: 6px 12px;
            background: #e74c3c;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 13px;
            cursor: pointer;
        }

        .btn-clear:hover {
            background: #c0392b;
        }

        .table-container {
            overflow-x: auto;
        }

        .record-table {
            width: 100%;
            border-collapse: collapse;
            font-size: 14px;
        }

        .record-table th,
        .record-table td {
            padding: 12px 8px;
            text-align: center;
            border-bottom: 1px solid #eee;
        }

        .record-table th {
            background: #f8f9fa;
            color: #7f8c8d;
            font-weight: 500;
        }

        .record-table tbody tr:hover {
            background: #f8f9fa;
        }

        .record-table tfoot td {
            font-weight: bold;
            background: #f8f9fa;
        }

        .btn-delete {
            padding: 4px 10px;
            background: #e74c3c;
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 12px;
            cursor: pointer;
        }

        .btn-delete:hover {
            background: #c0392b;
        }

        .empty-tip {
            text-align: center;
            color: #999;
            padding: 30px 0;
            display: none;
        }

        .profit-text {
            color: #e74c3c;
        }

        .loss-text {
            color: #27ae60;
        }

        @media (max-width: 500px) {
            .stats-grid {
                grid-template-columns: 1fr;
            }
            .form-row {
                grid-template-columns: 1fr;
            }
        }
```

---

## Task 4: ???????????????

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ?????????????????**

??????õ•??????????????

```javascript
        // ==================== ?????? ====================
        const buyPriceInput = document.getElementById('buyPriceInput');
        const buyWeightInput = document.getElementById('buyWeightInput');
        const buyFeeRateInput = document.getElementById('buyFeeRateInput');

        function updateBuyPreview() {
            const price = parseFloat(buyPriceInput.value) || 0;
            const weight = parseFloat(buyWeightInput.value) || 0;
            const feeRate = parseFloat(buyFeeRateInput.value) || 0;

            const amount = price * weight;
            const fee = amount * feeRate / 100;

            document.getElementById('previewBuyAmount').textContent = amount.toFixed(2) + ' ?';
            document.getElementById('previewBuyFee').textContent = fee.toFixed(2) + ' ?';
        }

        buyPriceInput.addEventListener('input', updateBuyPreview);
        buyWeightInput.addEventListener('input', updateBuyPreview);
        buyFeeRateInput.addEventListener('input', updateBuyPreview);
```

- [ ] **Step 2: ?????????????????**

```javascript
        function handleAddBuy() {
            const price = parseFloat(buyPriceInput.value) || 0;
            const weight = parseFloat(buyWeightInput.value) || 0;
            const feeRate = parseFloat(buyFeeRateInput.value) || 0;

            if (price <= 0 || weight <= 0) {
                alert('????????ßπ??????????????');
                return;
            }

            addBuyRecord(price, weight, feeRate);

            // ??????
            buyPriceInput.value = '';
            buyWeightInput.value = '';
            updateBuyPreview();

            // ??????
            refreshPortfolio();

            alert('????????');
        }
```

- [ ] **Step 3: ????????????????**

```javascript
        function refreshPortfolio() {
            const data = getData();
            const stats = getPortfolioStats();
            const sellFeeRate = parseFloat(document.getElementById('sellFeeRateInput').value) || 0.1;

            // ?????????
            document.getElementById('statTotalWeight').textContent = stats.totalWeight.toFixed(4) + ' ??';
            document.getElementById('statTotalInvestment').textContent = stats.totalInvestment.toFixed(2) + ' ?';
            document.getElementById('statAvgCost').textContent = stats.avgCost.toFixed(2) + ' ?/??';

            // ???°¿?????
            const breakEvenPrice = calculateBreakEvenPrice(stats.avgCost, sellFeeRate);
            document.getElementById('breakEvenPrice').textContent = breakEvenPrice.toFixed(2);

            // ??????????????
            const tbody = document.getElementById('buyRecordsBody');
            const tfoot = document.getElementById('buyRecordsFoot');
            const emptyTip = document.getElementById('emptyTip');

            if (data.buyRecords.length === 0) {
                tbody.innerHTML = '';
                tfoot.innerHTML = '';
                emptyTip.style.display = 'block';
                document.querySelector('.table-container').style.display = 'none';
            } else {
                emptyTip.style.display = 'none';
                document.querySelector('.table-container').style.display = 'block';

                let totalWeight = 0;
                let totalAmount = 0;
                let totalFee = 0;
                let totalRemaining = 0;

                tbody.innerHTML = data.buyRecords.map(record => {
                    totalWeight += record.weight;
                    totalAmount += record.amount;
                    totalFee += record.fee;
                    totalRemaining += record.remainingWeight;

                    return `
                        <tr>
                            <td>${formatTime(record.timestamp)}</td>
                            <td>${record.price.toFixed(2)}</td>
                            <td>${record.weight.toFixed(4)}</td>
                            <td>${record.amount.toFixed(2)}</td>
                            <td>${record.fee.toFixed(2)}</td>
                            <td>${record.remainingWeight.toFixed(4)}</td>
                            <td><button class="btn-delete" onclick="handleDeleteBuy('${record.id}')">???</button></td>
                        </tr>
                    `;
                }).join('');

                tfoot.innerHTML = `
                    <tr>
                        <td><b>???</b></td>
                        <td>-</td>
                        <td><b>${totalWeight.toFixed(4)}</b></td>
                        <td><b>${totalAmount.toFixed(2)}</b></td>
                        <td><b>${totalFee.toFixed(2)}</b></td>
                        <td><b>${totalRemaining.toFixed(4)}</b></td>
                        <td></td>
                    </tr>
                `;
            }
        }
```

- [ ] **Step 4: ?????????????????**

```javascript
        function handleDeleteBuy(id) {
            if (confirm('???????????????????')) {
                deleteBuyRecord(id);
                refreshPortfolio();
            }
        }

        function handleClearAll() {
            if (confirm('???????????????????????????????')) {
                clearAllBuyRecords();
                refreshPortfolio();
            }
        }
```

---

## Task 5: ?????????????????

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ????????????ß›????**

```javascript
        // ==================== ???????? ====================
        const sellTypeRadios = document.querySelectorAll('input[name="sellType"]');
        const sellQuantityLabel = document.getElementById('sellQuantityLabel');
        const sellQuantityInput = document.getElementById('sellQuantityInput');
        const sellPriceInput = document.getElementById('sellPriceInput');
        const sellFeeRateInput = document.getElementById('sellFeeRateInput');

        sellTypeRadios.forEach(radio => {
            radio.addEventListener('change', function() {
                if (this.value === 'weight') {
                    sellQuantityLabel.textContent = '?????????????';
                    sellQuantityInput.placeholder = '??????????????';
                } else {
                    sellQuantityLabel.textContent = '??????????';
                    sellQuantityInput.placeholder = '?????????????';
                }
                updateSellPreview();
            });
        });
```

- [ ] **Step 2: ??????????????????**

```javascript
        function updateSellPreview() {
            const stats = getPortfolioStats();
            const sellType = document.querySelector('input[name="sellType"]:checked').value;
            const price = parseFloat(sellPriceInput.value) || 0;
            const quantity = parseFloat(sellQuantityInput.value) || 0;
            const feeRate = parseFloat(sellFeeRateInput.value) || 0;

            let sellWeight, sellAmount;

            if (sellType === 'weight') {
                sellWeight = quantity;
                sellAmount = price * sellWeight;
            } else {
                sellAmount = quantity;
                sellWeight = price > 0 ? sellAmount / price : 0;
            }

            const sellFee = sellAmount * feeRate / 100;

            // ???????ó§??????? - ?????? - ????????
            const costBasis = sellWeight * stats.avgCost;
            const netProfit = sellAmount - sellFee - costBasis;

            document.getElementById('previewSellAmount').textContent = sellAmount.toFixed(2) + ' ?';
            document.getElementById('previewSellFee').textContent = sellFee.toFixed(2) + ' ?';

            const netProfitEl = document.getElementById('previewNetProfit');
            netProfitEl.textContent = (netProfit >= 0 ? '+' : '') + netProfit.toFixed(2) + ' ?';
            netProfitEl.className = 'preview-value ' + (netProfit >= 0 ? 'profit-text' : 'loss-text');
        }

        sellPriceInput.addEventListener('input', updateSellPreview);
        sellQuantityInput.addEventListener('input', updateSellPreview);
        sellFeeRateInput.addEventListener('input', updateSellPreview);
```

- [ ] **Step 3: ???????????????????**

```javascript
        function handleSell() {
            const stats = getPortfolioStats();
            const sellType = document.querySelector('input[name="sellType"]:checked').value;
            const price = parseFloat(sellPriceInput.value) || 0;
            const quantity = parseFloat(sellQuantityInput.value) || 0;
            const feeRate = parseFloat(sellFeeRateInput.value) || 0;

            if (price <= 0 || quantity <= 0) {
                alert('????????ßπ????????????????');
                return;
            }

            let sellWeight;
            if (sellType === 'weight') {
                sellWeight = quantity;
            } else {
                sellWeight = price > 0 ? quantity / price : 0;
            }

            if (sellWeight > stats.totalWeight) {
                alert(`???????????????????????${stats.totalWeight.toFixed(4)} ??`);
                return;
            }

            addSellRecord(price, sellWeight, feeRate);

            // ??????
            sellPriceInput.value = '';
            sellQuantityInput.value = '';
            updateSellPreview();

            // ??????
            refreshPortfolio();

            alert('?????????');
        }
```

---

## Task 6: ??????????

**Files:**
- Modify: `index.html`

- [ ] **Step 1: ??????????????**

?????ß‹???????????????

```javascript
        // ==================== ????? ====================
        // ???????????????
        document.addEventListener('DOMContentLoaded', function() {
            refreshPortfolio();
        });

        // Tab?ß›??????????
        tabBtns.forEach(btn => {
            btn.addEventListener('click', function() {
                if (this.dataset.tab === 'portfolio') {
                    refreshPortfolio();
                }
            });
        });
```

- [ ] **Step 2: ????????ß”???????????**

??????—s
1. ??index.html???ß›???"??????"Tab
2. ???????????????????????
3. ?????????ß“????????
4. ????????????????????????
5. ?????ó®??????????
6. ??????????????????????????

- [ ] **Step 3: ??????**

```bash
git add index.html
git commit -m "feat: ?????????????????

- ???Tab?ß›??????¶Ã???/????????
- ????????????õ•??localStorage??
- ????????????????
- ?????????????????????
- ???????ß“????????"
```

---

## ?????çÀ

- [ ] Tab?ß›?????????
- [ ] ???????????????????????
- [ ] ???????ß“???????
- [ ] ??????????????????
- [ ] ?????????????????
- [ ] ??????????????????????/?????????
- [ ] ?????ßµ????????
- [ ] ??????????????????????????????????
- [ ] ????????????????
- [ ] ??????????????
