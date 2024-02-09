
本次的挑戰來自KKcompany的Kaggle競賽。起因來自於大四陳亦中開的人工智慧課程，KKcompany跟多間學校合作參與這場競賽，其中一間就是我們老師開的這堂課。[比賽網址](https://www.kaggle.com/competitions/datagame-2023/data)

### 官方說明
在這場競賽中，您將獲得用戶的歌曲播放記錄和歌曲相關資訊的數據集。您需要建立一個預測模型，該模型基於用戶在同一個聆聽 session 內聆聽的前 N (=20) 首歌曲，預測接下來會聆聽哪 K (=5) 首歌曲。我們將使用 DCG（Discounted Cumulative Gain）和預測歌曲的覆蓋率（Coverage）來評估模型的性能。請注意，避免過度集中於熱門歌曲是本競賽的一個關鍵要求。參賽者可以使用任何合法外部資源和工具，以開發創新的解決方案，以改進音樂串流體驗，提高用戶滿意度。

### 做法
- **方法一:** 專門處理聽的歌曲數<=5(不包含重複的)的session_id
- **方法二:** 認為有一部分的歌曲播放不是來自於session_id自己點選的，而是來自系統自動播放的。所以先填補session_id自己想聽的歌曲，再找出系統自動播放的循環歌曲填補。但運算速度太慢就放棄。最後是用方法四完成
- **方法三:** 專門處理聽的歌曲數>5(不包含重複的)的session_id。分成低、中、高三種密度群體，然後每一個群體都訓練一個模型。輸入為20個str list，輸出為5個str list，輸入輸出都是song_id。
- **方法四:**  
  1. session_id常重複聽的歌為優先先排  
  2. session_id最後一首歌若只聽過一次(在前面的20次以內)，我認為接下來聽的歌是來自系統推薦的歌單。所以找出其他session_id同樣只播過一次這首歌的紀錄(代表為系統推薦的，而不是session_id自己想重複聽的)，預測下一首系統推薦的歌
### 預處理
要把官方的資料包改名為 data 後放在主目錄下

### 結果
分數最高的是方法四。得到了  
**Public Score : 0.25318 / Private Score : 0.2473**

### 問題

1. **tokenize:** 因為輸出的song_id必須是官方的格式，但是他們的id跟亂碼一樣，導致生成的id也是亂碼的樣子但不是官方格式的id。原本想用1 to 1的方式對應song_id去做tokenize，但是song_id共有一百多萬首，而一般模型的單字量沒有到那麼大。舉GPT-2為例，它的單字量為三萬多，雖然有參數可以調整模型的參數量，但調整到一百萬之後訓練模型時的記憶體爆掉了。這個問題即使到最後仍沒解決，所以使用尋找規則的方式處理。

2. **找循環規則運算太久:** 至少要跑四五個小時以上才會跑完，還是有點久。

3. **模型不會架構:** 其中一個最嚴重的問題!!!沒啥好講的，就是沒實力罷了，只能靠chatGPT跟copilot幫忙，摳連。

### 可改進流程

1. **AI模型提早著手處理:** 這次為了做出fixed的seq2seq model **<font color="red">(20 str list $\rightarrow$ 5 str list)</font>** 花了超多時間在弄。除了模型架構之外，最麻煩的是tokenize的問題。

2. **使用goto:** 因為迴圈太多，只透過break會忘記下一個迴圈也要寫break，而且程式看起來很冗，所以以後還是用goto比較好。 

3. **從大範圍的開始著手:** 這次我先從歌曲數 <=5 的session_id開始處理，但是後來發現符合規則的人數十分的少，導致做了很久後得到的分數提升非常的弱。

4. **分析數據太久且沒用:** 分析了很多distribution間的關係，但到最後幾乎都沒用上。即使有用可能也不知道怎麼用。與其花那麼多時間在上面，不如學習模型的架構該如何建立。

5. **看太多其他人的:** 雖然看了，也大致理解原理，但到了自己的case上還是用不出來XD 剛開始還是可以看，只是大致有自己的想法後就可以開始著手處理。依據自己想做的方法再深入查詢跟自己同樣的做法可能是比較好的流程。
