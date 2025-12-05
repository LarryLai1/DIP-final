# Annotation → 覆蓋標註並輸出影片

簡介
- 本資料夾包含 `annotate.ipynb`，可把自動或手動產生的標註（polygon / bbox / YOLO / JSON）疊加回影片並輸出為標註後影片。

必要套件
- Python 3.x
- OpenCV (cv2)
- numpy
- tqdm
- ultralytics (for auto_annotate)
- Jupyter / IPython

主要流程（Notebook cell 執行順序）
1. 匯出影片每一幀到 frames 目錄（預設：4DGS/{video_name}_frames/）。  
2. 使用 `auto_annotate`（ultralytics）產生標註，輸出至 `annotations/{frames_basename}/`。  
3. 讀取每個 frames 目錄，逐幀尋找對應標註（annotations/{frames_basename}/{frame_stem}.(txt|json)），解析並繪製標註，最後輸出影片至 `output_videos/annotated_{frames_basename}.mp4`。

支援的標註格式（.txt / .json）
- .txt：
  - 若是偶數個數字且 >=6：視為 polygon (x1 y1 x2 y2 ...)，若數值在 [0,1] 則視為 normalized，會依影像尺寸還原到像素座標。  
  - 若 5 個數字：視為 YOLO 格式 center (xc yc w h)（會轉成 bbox）。  
  - 若 4 個數字：視為 absolute bbox (x y w h)。  
  - 第一欄若為非數字，則當作 label 字串；否則當作 class id。
- .json：
  - 支援常見 keys（`annotations`, `shapes`, `objects`, `labels`），處理 `bbox`, `points`, `segmentation` 等欄位。

繪製規則
- bbox：綠色邊框 (0,255,0)，並顯示類別/label。  
- polygon：先使用 #00FFFF 填滿（BGR=(255,255,0)），以 alpha=0.4 與原影像混合，再繪製紅色邊界 (0,0,255) 與 label。

輸出
- 所有標註後影片會放到 `output_videos/`，檔名格式：`annotated_{frames_basename}.mp4`。

常見設定變數（在 notebook 中）
- video_dir = "4DGS"
- frames 目錄：4DGS/{name}_frames
- annotations 目錄：annotations/{frames_basename}
- output 目錄：output_videos/
- 填色顏色：#00FFFF（在程式碼以 BGR=(255,255,0) 使用）

排錯建議
- 確認 frames 已正確匯出且檔名順序對應（frame stem 對應 annotation 檔名）。  
- 若缺少標註檔，程式會跳過該幀。  
- 若 fps 錯誤，可手動在 notebook 中指定。  
- 若 polygon 未對齊，檢查 .txt 是否使用 normalized coords（0..1）或絕對座標（像素）。

範例快速步驟
1. 在 notebook 執行匯出 frames 的 cell（Cell 2）。  
2. 執行 auto_annotate 的 cell（Cell 3）。  
3. 執行批次疊加並輸出影片的 cell（最後一個 cell），結果會輸出到 `output_videos/`。

如需自訂或整合其他標註格式，可在 notebook 中擴充 `parse_txt` / `parse_json` 函數。
