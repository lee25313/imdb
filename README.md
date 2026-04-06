```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.service import Service
import re
import time


print("IMDb 評分查詢工具啟動中...")

# 1. 設置 Chrome 設定 
chrome_options = Options()
chrome_options.add_argument("--disable-blink-features=AutomationControlled")
chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
chrome_options.add_argument("start-maximized")
chrome_options.add_argument("user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36")

# 無限循環，直到使用者想退出
while True:
    movie_name = input("\n 請輸入電影名稱 (輸入 q 退出): ").strip()

    # 退出判斷
    if movie_name.lower() in ['q', 'exit', 'quit']:
        print("掰掰！")
        break
    if not movie_name:
        continue
    
    # 檢查有沒有奇怪的指令字元
    if any(x in movie_name.lower() for x in ['python', '.exe', 'cmd', '&']):
        print("無效輸入。請直接輸入電影名稱")
        continue

    print(f"查詢中: {movie_name}...")

    # 2. 啟動瀏覽器 
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=chrome_options)
    
    # 保留移除 WebDriver 標誌功能
    driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {
        'source': 'Object.defineProperty(navigator, "webdriver", {get: () => undefined})'
    })

    try:
        # 3. 前往網頁
        driver.get("https://www.imdb.com")
        wait = WebDriverWait(driver, 10)

        # 4. 搜尋電影 
        # 使用 CSS Selector
        search_box = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, "input[data-testid='search-input'], #suggestion-search")))
        search_box.send_keys(movie_name + Keys.RETURN)
        
        # 5. 點擊第一個搜尋結果
        first_link = wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, "a[href*='/title/tt']")))
        first_link.click()
        
        # 6. 提取評分 
        rating_element = wait.until(EC.presence_of_element_located((
            By.CSS_SELECTOR, "[data-testid='hero-rating-bar__aggregate-rating__score'] > span:first-child"
        )))
        
        raw_text = rating_element.text
        
        # 判斷結果並印出
        if re.match(r'^\d+\.?\d*$', raw_text):
            res = float(raw_text)
            print(f"找到！IMDb 評分: {res}/10 ")
        else:
            print("找不到電影或解析失敗")

    except Exception as e:
        print(f" 搜尋過程中出錯: {e}")
    
    # 7. 每查完一次就關閉瀏覽器 
    driver.quit()

print("\n程序結束。")
```
