# 21點（Blackjack）單檔版開發規格（Spec.md）

> 目標：依據前述 SA/SD（去除金錢概念），實作一個**單檔 `index.html`** 的 21 點小遊戲。  
> 內容包含：**基本 UI、隨機抽牌、簡單敘述（事件/說明文字）**。  
> 不包含：下注/金錢/賠率/帳務。

---

## 1. 範圍（Scope）

### 1.1 必做（Must Have）
- 單一檔案：`index.html`（內含 HTML + CSS + JavaScript，可用 `<style>` 與 `<script>` 內嵌）
- 基本 UI：
  - 顯示玩家手牌、莊家手牌（莊家暗牌在玩家回合需隱藏）
  - 顯示目前點數（玩家需即時計算；莊家在暗牌揭露前僅顯示明牌點數或顯示 `?`）
  - 操作按鈕：**開始新回合、要牌(Hit)、停牌(Stand)**
- 遊戲流程：
  - 開局發牌：玩家 2 張、莊家 2 張（1 明 1 暗）
  - 玩家回合可多次 Hit，或 Stand 結束玩家回合
  - 玩家爆牌立即結束玩家回合並進入結算（可直接顯示結果）
  - 玩家 Stand 後，莊家依規則補牌直到停止，然後結算
- 隨機抽牌：
  - 使用標準 52 張牌
  - 每張牌**不可重複**（一回合使用一副牌即可；牌用完可自動重洗或直接重開新回合）
  - 洗牌採 Fisher–Yates shuffle
- 簡單敘述（文字日誌/事件描述）：
  - 顯示每一步操作與結果，例如：
    - 「新回合開始，發初始牌」
    - 「玩家要牌：♣7」
    - 「玩家爆牌」
    - 「莊家補牌至 17 點停牌」
    - 「結算：玩家 WIN / LOSE / PUSH」

### 1.2 不做（Out of Scope）
- 下注、籌碼、金流、賠率、線上帳號系統
- 多玩家、AI 玩家、牌靴多副牌（可列為擴充）
- Split/Double/Surrender（本規格為「簡化版」，先不納入）
- 動畫、音效（可列為擴充）

---

## 2. 遊戲規則（Rules）

### 2.1 牌面點數
- 2–10：牌面數字
- J/Q/K：10 點
- A：可算 1 或 11，取「不爆牌且點數最大」的計算方式

### 2.2 Blackjack（天生 21）
- 玩家或莊家在**初始兩張牌**為 `A + 10點牌(10/J/Q/K)` 視為 Blackjack
- 本單檔版不含賠率，只用於判定勝負：
  - 玩家 Blackjack 且莊家非 Blackjack → 玩家 WIN
  - 莊家 Blackjack 且玩家非 Blackjack → 玩家 LOSE
  - 雙方 Blackjack → PUSH

### 2.3 爆牌（Bust）
- 任一方手牌點數 > 21 為 bust
- 玩家 bust：立即 LOSE（本回合結束並揭露莊家暗牌）
- 莊家 bust：玩家 WIN（前提是玩家未 bust）

### 2.4 莊家補牌規則（Dealer Rule）
- 莊家點數 **<= 16 必須補牌**
- 莊家點數 **>= 17 必須停牌**
- 本版採 **S17**：莊家 Soft 17（含 A 且算 11 的 17）也停牌

### 2.5 比點數結算（Settlement）
- 在雙方都未 bust 且非 Blackjack 特例時：
  - 玩家點數 > 莊家點數 → WIN
  - 玩家點數 < 莊家點數 → LOSE
  - 相等 → PUSH

---

## 3. 使用流程（User Flow）

### 3.1 開始新回合
1. 使用者點擊「New Round」
2. 系統：
   - 建立牌組（52 張）並洗牌
   - 清空 UI 與日誌
   - 發牌：玩家 2 張、莊家 2 張（莊家第 2 張為暗牌）
   - 檢查 Blackjack：
     - 若有結果（玩家/莊家 blackjack），立刻揭露暗牌並顯示結算
     - 否則進入玩家回合

### 3.2 玩家回合
- 可執行動作：
  - Hit：玩家抽 1 張
    - 若 bust → 結算
  - Stand：結束玩家回合，進入莊家回合

### 3.3 莊家回合
- 揭露莊家暗牌
- 莊家依規則補牌直到停牌
- 進行結算並顯示結果
- 禁用 Hit/Stand，直到 New Round

---

## 4. UI 規格（單檔 index.html）

### 4.1 版面元素（必備）
- 標題：`21 點（簡化版）`
- Dealer 區：
  - 手牌顯示（牌面文字即可，例如 `A♠`、`10♦`）
  - 點數顯示（玩家回合：暗牌未揭露時顯示 `Dealer: ?` 或只顯示明牌點數）
- Player 區：
  - 手牌顯示
  - 點數顯示（即時更新）
- 控制按鈕：
  - `New Round`
  - `Hit`
  - `Stand`
- 日誌區（Game Log）：
  - 顯示最新事件（建議可累積 10–50 行，或可捲動）

### 4.2 UI 狀態與按鈕可用性
- INIT/END 狀態：
  - Hit：disabled
  - Stand：disabled
  - New Round：enabled
- PLAYER_TURN 狀態：
  - Hit：enabled
  - Stand：enabled
  - New Round：enabled（允許強制重開，重置狀態）
- DEALER_TURN / SETTLEMENT：
  - Hit：disabled
  - Stand：disabled
  - New Round：enabled

---

## 5. 資料結構與核心函式（JavaScript）

### 5.1 資料結構（建議）
- `deck: Card[]`
- `playerHand: Card[]`
- `dealerHand: Card[]`
- `phase: "INIT" | "PLAYER_TURN" | "DEALER_TURN" | "SETTLED"`

`Card` 建議表示：
```js
{ rank: "A"|"2"|...|"10"|"J"|"Q"|"K", suit: "♠"|"♥"|"♦"|"♣" }
```

### 5.2 核心函式（Must）
- `createDeck(): Card[]`
- `shuffle(deck: Card[]): void`（Fisher–Yates）
- `drawCard(deck): Card`（pop）
- `handScore(hand: Card[]): { best: number, isSoft: boolean, isBust: boolean }`
- `isBlackjack(hand: Card[]): boolean`
- `startNewRound(): void`
- `playerHit(): void`
- `playerStand(): void`
- `dealerPlay(): void`
- `settle(): void`
- `render(): void`（更新 UI）
- `log(message: string): void`

### 5.3 點數計算規則（handScore）
- 先將所有 A 當 1 加總
- 若有 A 且 `sum + 10 <= 21` 則 `best = sum + 10` 並 `isSoft=true`
- 否則 `best=sum`
- 若 `best > 21` → `isBust=true`

---

## 6. 結果輸出（Outcome）
- 顯示於 UI（可在 log 與獨立結果欄位呈現）：
  - `WIN` / `LOSE` / `PUSH`
  - 原因（可選，建議顯示）：`BLACKJACK` / `BUST` / `HIGHER_SCORE` / `TIE`

---

## 7. 驗收條件（Acceptance Criteria）

1. **單檔運作**：下載 `index.html` 直接用瀏覽器開啟即可遊玩（不需伺服器）。
2. **不重複抽牌**：同一回合內抽出的牌不會再次出現。
3. **流程正確**：
   - New Round 後必定發到玩家 2 張、莊家 2 張（莊家 1 暗）
   - 玩家 Hit 可多次抽牌，超過 21 立刻 bust 並結束
   - 玩家 Stand 後，莊家依 <=16 補牌 >=17 停牌（S17）
   - 正確結算 WIN/LOSE/PUSH
4. **UI 正確隱藏暗牌**：玩家回合期間莊家暗牌不可見；進入莊家回合或結算後必須揭露。
5. **日誌可讀**：每次發牌/要牌/停牌/莊家補牌/結算都有文字敘述。

---

## 8. 擴充項目（Optional / Future）
- 加入 Split / Double / Surrender（仍不含金錢）
- 多副牌 Shoe、切牌位置、牌用盡自動重洗
- 回合事件匯出（JSON）與重播
- 更完整的牌面 UI（卡片樣式、簡易動畫）

---

## 9. 交付物（Deliverables）
- `index.html`（單檔，含內嵌 CSS/JS）
- `Spec.md`（本文件）
