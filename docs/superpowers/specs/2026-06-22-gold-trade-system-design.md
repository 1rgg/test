# 积存金多次买入交易系统设计文档

## 概述

在现有积存金收益计算器基础上，扩展支持多次买入功能的交易系统模块。

## 需求确认

| 项目 | 决策 |
|------|------|
| 数据存储 | 浏览器 localStorage |
| 资产类型 | 单一资产（积存金） |
| 卖出逻辑 | 用户可选择卖出克数或金额 |
| 费用类型 | 仅手续费率 |
| 界面风格 | Tab切换模式扩展现有界面 |

## 整体架构

### 界面布局

```
┌─────────────────────────────────────────────┐
│  ? 积存金收益计算器                          │
├─────────────────────────────────────────────┤
│  [单次计算]  [持仓管理]  ← Tab切换            │
├─────────────────────────────────────────────┤
│  Tab内容区域                                 │
│  - 单次计算：现有功能保持不变                  │
│  - 持仓管理：新增的多次买入功能                │
└─────────────────────────────────────────────┘
```

### 数据结构

```javascript
// localStorage key: "goldTradeData"
{
  "buyRecords": [
    {
      id: "uuid",
      timestamp: "2024-01-15 10:30:00",
      price: 580.50,        // 买入单价（元/克）
      weight: 10.5,         // 买入克数
      amount: 6095.25,      // 买入总金额（元）
      feeRate: 0.1,         // 手续费率（%）
      fee: 6.10,            // 手续费（元）
      remainingWeight: 8.5  // 剩余克数（卖出后减少）
    }
  ],
  "sellRecords": [
    {
      id: "uuid",
      timestamp: "2024-01-20 14:00:00",
      price: 590.00,        // 卖出单价（元/克）
      weight: 2.0,          // 卖出克数
      amount: 1180.00,      // 卖出总金额（元）
      feeRate: 0.1,         // 手续费率（%）
      fee: 1.18             // 手续费（元）
    }
  ]
}
```

## 功能模块

### 1. Tab切换

- 页面顶部添加Tab导航
- 默认显示"单次计算"Tab
- 点击切换显示对应内容

### 2. 新增买入表单

**输入字段：**
- 买入价格（元/克）- 必填
- 买入克数（克）- 必填
- 买入手续费率（%）- 默认0.1

**实时计算显示：**
- 预计买入金额 = 买入价格 × 买入克数
- 预计手续费 = 预计买入金额 × 手续费率 / 100

**操作：**
- 确认买入：保存记录到localStorage

### 3. 买入记录列表

**显示字段：**
| 字段 | 说明 |
|------|------|
| 时间 | 买入时间（YYYY-MM-DD HH:mm） |
| 单价 | 买入单价（元/克） |
| 克数 | 买入克数 |
| 金额 | 买入总金额（元） |
| 手续费 | 买入手续费（元） |
| 剩余 | 剩余克数 |

**操作：**
- 删除单条记录
- 清空全部记录

**汇总行：**
- 总克数
- 加权平均单价
- 总金额
- 总手续费
- 总剩余克数

### 4. 卖出操作

**输入字段：**
- 卖出方式：按克数 / 按金额（单选）
- 卖出价格（元/克）- 必填
- 卖出数量（克数或金额，根据方式切换）- 必填
- 卖出手续费率（%）- 默认0.1

**实时计算显示：**
- 预计卖出金额
- 预计手续费
- 预计净收益

**校验：**
- 卖出数量不能超过总剩余克数

**操作：**
- 确认卖出：扣减买入记录的剩余克数，保存卖出记录

### 5. 持仓统计与保本价

**统计信息：**
- 总持仓克数
- 总投入金额（含手续费）
- 加权平均成本（元/克）

**保本卖出价计算公式：**

```
加权平均成本 = Σ(买入金额 + 买入手续费) / 总克数

保本卖出价 = 加权平均成本 × (1 + 卖出手续费率/100) / (1 - 卖出手续费率/100)
```

## 核心计算逻辑

### 加权平均成本计算

```javascript
function calculateWeightedAvgCost(buyRecords) {
  let totalAmount = 0;  // 总投入（含手续费）
  let totalWeight = 0;  // 总克数

  buyRecords.forEach(record => {
    if (record.remainingWeight > 0) {
      totalAmount += record.amount + record.fee;
      totalWeight += record.remainingWeight;
    }
  });

  return totalWeight > 0 ? totalAmount / totalWeight : 0;
}
```

### 保本卖出价计算

```javascript
function calculateBreakEvenPrice(avgCost, sellFeeRate) {
  // 保本条件：卖出净收入 = 总投入
  // 卖出金额 - 卖出手续费 = 总投入
  // 卖出金额 × (1 - 卖出手续费率/100) = 总投入
  // 卖出金额 = 总投入 / (1 - 卖出手续费率/100)
  // 保本单价 = 卖出金额 / 总克数 = 加权平均成本 / (1 - 卖出手续费率/100)

  return avgCost / (1 - sellFeeRate / 100);
}
```

### 卖出时扣减逻辑

```javascript
function processSell(sellWeight, buyRecords) {
  let remainingSell = sellWeight;

  // 按时间顺序扣减（FIFO）
  for (let record of buyRecords) {
    if (remainingSell <= 0) break;
    if (record.remainingWeight > 0) {
      const deduct = Math.min(remainingSell, record.remainingWeight);
      record.remainingWeight -= deduct;
      remainingSell -= deduct;
    }
  }
}
```

## UI样式规范

- 保持现有页面风格
- 使用相同的配色方案（绿色主色调 #27ae60）
- 表格使用斑马纹样式
- 操作按钮使用危险色（红色）表示删除操作

## 实现文件

所有功能在现有 `index.html` 文件中实现，包括：
- HTML结构扩展
- CSS样式扩展
- JavaScript逻辑扩展
