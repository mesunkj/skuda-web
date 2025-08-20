
### Sudoku Web Solver: A Hybrid GAS-Based Solution

[cite\_start]這個專案是一個以 Google Apps Script (GAS) 為後端、純瀏覽器（HTML + jQuery）為前端的互動式數獨解題系統。它採用了獨特的混合式架構，旨在平衡運算效率與使用者互動體驗 [cite: 8]。

#### 專案特色

  * [cite\_start]**混合式架構**：將昂貴的數獨求解運算（回溯法）放在 GAS 後端一次性執行，而高頻率、輕量的互動操作（如「下一步」、「填入 N 筆」）則完全在前端處理 [cite: 21]。
  * [cite\_start]**高效互動**：系統僅在初始化階段向 GAS 請求並計算完整解答（`answer`）。之後的所有步驟，前端皆透過比對當前盤面 (`board`) 與完整解答 (`answer`) 的差異來進行漸進式更新與高亮顯示，大幅減少了後端呼叫次數，提高了互動流暢性 [cite: 8, 103]。
  * [cite\_start]**狀態管理**：核心狀態模型包含 `qboard`（原始題目）、`board`（當前盤面）和 `answer`（完整解答） [cite: 45, 46, 48]。
  * [cite\_start]**核心演算法**：後端採用典型的\*\*回溯法（Backtracking）\*\*來尋找數獨解答 [cite: 53, 54]。
  * [cite\_start]**簡潔部署**：專案以 Web App 形式部署於 Google Apps Script，執行身分為「我」，可供「任何人」存取 [cite: 95, 96, 97]。

#### [cite\_start]系統架構與流程 [cite: 22]

1.  [cite\_start]**使用者輸入題目**：在 9x9 的表格中輸入數獨題目 [cite: 24]。
2.  [cite\_start]**初始化解題**：前端將題目 (`qboard`) 傳送至 GAS 後端 [cite: 25]。
3.  [cite\_start]**後端計算與回傳**：GAS 使用回溯法求解並產生完整的 `answer`，隨後將其一次性回傳給前端 [cite: 25]。
4.  [cite\_start]**前端互動模式**：進入互動模式後，使用者可點擊以下按鈕[cite: 26, 63]:
      * [cite\_start]**「下一步」**：前端掃描 `board`，找到第一個空格 (`0`)，並以 `answer` 中的值填入，同時高亮顯示 [cite: 27, 65]。
      * [cite\_start]**「填入 N 筆答案」**：一次性填入多個空格 [cite: 29, 66]。
      * [cite\_start]**「重置」**：將 `board` 恢復為原始題目 `qboard` [cite: 30, 67]。

#### [cite\_start]資料交換格式 (JSON) [cite: 73]

  * [cite\_start]**請求 (前端 -\> 後端)**[cite: 74]:
    ```json
    {
      "action": "init",
      "data": {
        "qboard": [[...9x9...]],
        "board": [[...9x9...]],
        "answer": []
      }
    }
    ```
  * [cite\_start]**回應 (後端 -\> 前端)**[cite: 83]:
    ```json
    {
      "status": "ok",
      "message": "初始化完成",
      "data": {
        "qboard": [[...9x9...]],
        "board": [[...9x9...]],
        "answer": [[...9x9...]]
      }
    }
    ```

#### 如何執行 (GAS 專案部署)

1.  在 Google Apps Script 中建立一個新專案。
2.  **核心前端邏輯**：將 `附錄A` 中的 JavaScript 程式碼複製到一個 `.html` 檔案中 (例如 `Index.html`)。
    ```javascript
    //「下一步」:找第一個0.用 answer 填入並 highlight
    function fillOneStep() {
      $("#sudoku-grid input").removeClass("highlight");
      for (let r=0; r<9; r++) {
        for (let c=0; c<9; c++) {
          if (sudokuState.board[r][c] === 0) {
            sudokuState.board[r][c] = sudokuState.answer[r][c];
            const $cell = $("#sudoku-grid tr").eq(r).find("input").eq(c);
            $cell.val(sudokuState.answer[r][c]).addClass("highlight");
            return true;
          }
        }
      }
      return false; //已無可填
    }
    ```
3.  **核心後端邏輯**：將 `附錄B` 中的 Google Apps Script 程式碼複製到一個 `.gs` 檔案中 (例如 `Code.gs`)。
    ```javascript
    function initSudoku() {
      sudokuState.answer = JSON.parse(JSON.stringify(sudokuState.qboard));
      solve(sudokuState.answer);
      return JSON.stringify({ status: "ok", data: sudokuState });
    }

    function solve(board) {
      for (let r=0; r<9; r++)
        for (let c=0; c<9; c++)
          if (board[r][c] === 0) {
            for (let n=1; n<=9; n++)
              if (isSafe(board, r, c, n)) {
                board[r][c] = n;
                if (solve(board)) return true;
                board[r][c] = 0;
              }
            return false;
          }
      return true;
    }
    // isSafe() 函數也應在此處包含，報告中未提供完整程式碼，但為數獨解題必需。
    ```
4.  **部署為 Web App**：
      * 在 GAS 編輯器中，點擊「部署」\>「新增部署」。
      * 選擇部署類型為「網路應用程式」。
      * 「執行身分」設定為「我」。
      * 「可存取對象」設定為「任何人」。
      * [cite\_start]部署後，複製產生的 URL，即可在瀏覽器中開啟並使用 [cite: 95, 96, 97]。
5.  [cite\_start]每次更新程式碼後，記得點擊「管理部署」\>「編輯」\>「部署」來更新部署，確保 Web App 指向最新版本 [cite: 98]。

#### [cite\_start]未來改進方向 [cite: 114]

  * [cite\_start]**多使用者隔離**：以 token/session key 方式區分 GAS 端狀態 [cite: 115]。
  * [cite\_start]**合法性驗證與提示**：前端輸入題目時立即檢查行列宮是否存在矛盾 [cite: 116]。
  * [cite\_start]**多解答列舉**：支援「列出前N個完整解答」並提供切換顯示 [cite: 117]。
  * [cite\_start]**更豐富的前端提示**：例如候選數標註（pencil marks）、邏輯推理過程視覺化 [cite: 118]。
  * [cite\_start]**錯誤復原/撤銷 (Undo/Redo)**：提供操作歷程回放 [cite: 119]。
  * [cite\_start]**單元測試/端到端測試**：以 GAS + Jest (或 clasp + 本地模擬) 強化可靠性 [cite: 120]。
  * [cite\_start]**權限與安全**：更細緻地限制 Web App 的可訪問對象與頻率 [cite: 121]。
