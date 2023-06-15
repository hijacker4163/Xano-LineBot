# Xano-LineBot
## 使用Xano製作LINE Bot
  
1. 您需要先準備什麼
  - 註冊 LINE Developer
    - https://developers.line.biz
  
  - 建立 Providers
    - Create a new channel
    - Channel type -> Messaging API
  
  - 輸入頻道的基本資料
  - 建立完成在 Basic settings 記下 Your user ID
  - 接著進入 Messaging API settings
  - Webhook URL 我們先留空，稍後會在 Xano 那邊生成網址
  - 設定 LINE Official Account features (點選任意 Edit 會跳轉到 LINE official Account Manager) 
    - 帳號設定 - 如果您的機器人會加入群組，請將 Allow bot to join group chats (加入群組或多人聊天室) 開啟
    - 回應設定 - 關閉【聊天】、【加入好友的歡迎訊息】、【自動回應訊息】
    - 回應設定 - 開啟【Webhook】

2. 開始 Xano 註冊及設定
   - https://www.xano.com
   - 註冊完畢即進入開發頁面 (https://app.xano.com/admin/instance)
   - 點擊 Instances
   
3. Database (資料庫) 
   - 您可以選擇自己從頭開始建立資料表，也可以選擇匯入 CSV
   - 資料庫特別要說明的建立索引 (Indexes)
      - 基本上 id 是唯一值 (Primary)，但是您可能有其他需求需要將欄位設為唯一值，此時可以 Create Index。
      - Index Type 選擇 unique 即可
    
4. API
   - 接下來就是本篇的主要內容，如何實現 No Code 的後台
   1. Add API Group
   2. Add API Endpoint
      - 可以快速選擇您想要做的功能，但我們選擇從無開始 (Start From scratch)
      - 輸入 API 的名稱及請求方式
      - 請求方式選擇 POST   
   3. API 設定說明
      - Inputs 表示您可以在這裡建立參數，也就是我們在 GET/POST 的 Query String ，這邊我們暫時不會使用到。
      - Function Stack 則是開始進行想要完成的工作
      - 先建立一個 Get All Input，因為我們必須從 LINE Platform 取得 Webhook
          1. Utility Functions -> Get All Input (Webhook)
          2. 在 Response 內會預設一個 RETURN，我們將他先刪除。
      - 建立完成後先 Publish，因為我們要將這個 API 生成網址並寫入 LINE Developer 的 Webhook URL 內。
          1. 點擊上方列的 Copy Endpoint URL
          2. 將複製好的 URL 貼在 Webhook settings 的 Webhook URL，此時可以點選 Verify，如果回傳 Success 表示正常。
              - 記得 Use webhook 是要開啟的狀態！
          3. 回到 Xano 點擊上方列的 Request History 就可以看到剛剛進行測試記錄。
          4. 接著我們將 LINE BOT 加入群組或是個別進行對話 (可以在 Messaging API 掃描 QR code 加入 也可以在 LINE 上搜尋好友輸入 Bot basic ID)
          5. 此時傳送一段字串給機器人，再回到 Request History，有了 Input 的資訊後就可以針對使用者發送的文字進行作業處理。
              - 我們先將 Input 進行複製
          6. Input 解析並建立參數：我們在 Get All Input 下方加入一個變數
              - Data Manipulation -> Create Variable -> VALUE 輸入 var_1，後面會出現 Sub Path，點擊後將剛剛 Input 複製的內容貼上並 Define，就可以直接選擇你要取得的參數並生成變數。
              - 因為我們要做的是回覆機器人所以再將參數【replyToken】進行擷取
          7. 建立 API Request
              - External API Request
              - API 的部份您可以到 LINE Developers 的 Documentation 選擇您要進行的作業，但目前請直接複製以下代碼直接 IMPORT CURL 即可
              ```
              curl -v -X POST https://api.line.me/v2/bot/message/reply \
              -H 'Content-Type: application/json' \
              -H 'Authorization: Bearer {channel access token}' \
              -d '{
                  "replyToken":"",
                  "messages":[
                      {
                          "type":"text",
                          "text":""
                      }
                  ]
              }'
              ```
              - 接著我們需要修改三個地方：
                1. replyToken value 改為我們上面建立的變數 replyToken
                2. messages -> text 改為變數 user_text (後續請依照自己的需求去修改)
                3. Authorization 需要將 channel 的 token 填入 (token 請到 LINE Developer Messaging API -> Channel access token 進行建立)
              - 修改完後再次 Publish，並發送訊息給機器人，即可得到回覆。

5. Library -> Function 範例說明
   - Add Function取名為getGas
   - 在Function Stack 新增指令
   - External API Request
   1. Inputs內容如下
      - TEXT:https://gas.goodlife.tw/gas.json?version=2
      - ENUM:GET
      - ANY:{}
      - TEXT[]:[]
      - INTEGER:10
      - BOOL:true
   2. Output內容如下      
      - AS:api_1
            
   - External API Request結束後可以Run & Debug進行測試
   - 回傳的資料基本上都在response內
   
   - Data Manipulation -> Create Variable 
   - 接下來我們將需要的資訊儲存在Variable內
   1. Inputs內容如下
      - VARIABLE NAME:gas_price
      - VALUE:api_1.response.result.gas_price
      
   - Data Manipulation -> Conditional
   1. 建立if判斷功能
      - 條件內容：gas_price >= 0
      - 建立Variable命名為gas_price_text
      - VALUE部分設為text並輸入：下週油價預計上漲 %s 元
      - 透過ADD FILTER新增sprintf並將gas_price當做參數
   2. 功能說明：這邊主要是透過if去判斷gas_price（油價波動）如果大於0表示下週油價將漲價，反之。
   
   - Response
   - 最後將建立的參數gas_price_text當做這個Function的AS（Return的值）就完成了

