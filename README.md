import requests
from bs4 import BeautifulSoup
import time

# 1. 監控的台灣參展商清單與大會 URL (範例)
TAIWAN_EXHIBITORS = [
    {
        "name": "HAN-SIENT TRADING CO., LTD.",
        "url": "https://exhibitors.vitafoods.eu.com/vfe26/exhibitor/han-sient" # 實例 URL
    },
    {
        "name": "VEDAN BIOTECHNOLOGY CORPORATION",
        "url": "https://exhibitors.vitafoods.eu.com/vfe26/exhibitor/vedan-bio"
    },
    {
        "name": "Microbio Co., Ltd.",
        "url": "https://exhibitors.vitafoods.eu.com/vfe26/exhibitor/microbio"
    }
]

# 2. 敏感違規字眼黑名單
BLACK_LIST_KEYWORDS = [
    "Province of China", 
    "Taiwan, Province of China", 
    "Taiwan, China", 
    "China, Taiwan"
]

# 3. 發送 LINE / Teams 警示函數 (以 LINE Notify / Webhook 為例)
def send_alert(company_name, current_text, page_url):
    message = (
        f"\n⚠️【Vitafoods 臺灣館國別名稱異常警示】⚠️\n"
        f"------------------------------------\n"
        f"🏢 參展廠商：{company_name}\n"
        f"🚨 偵測到異常國別標示：[{current_text}]\n"
        f"🔗 大會網址：{page_url}\n"
        f"⏰ 偵測時間：{time.strftime('%Y-%m-%d %H:%M:%S')}\n"
        f"請承辦同仁儘速聯繫大會主辦單位交涉修正！"
    )
    
    # 範例：發送至團隊的 Webhook (Teams / LINE / Slack)
    webhook_url = "YOUR_WEBHOOK_URL_HERE"
    payload = {"text": message}
    try:
        requests.post(webhook_url, json=payload, timeout=5)
        print(f"[SUCCESS] 警示已發送：{company_name}")
    except Exception as e:
        print(f"[ERROR] 發送警示失敗：{e}")

# 4. 主爬蟲與檢測邏輯
def run_watchdog():
    print("🤖 Vitafoods 國別監控機器人啟動...")
    headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"}
    
    for company in TAIWAN_EXHIBITORS:
        try:
            res = requests.get(company["url"], headers=headers, timeout=10)
            if res.status_code == 200:
                soup = BeautifulSoup(res.text, "html.parser")
                
                # 抓取大會網頁上顯示國別欄位的 HTML 節點 (依據大會實際 DOM 調整)
                country_element = soup.find("span", class_="country-name") or soup.find("div", class_="exhibitor-country")
                country_text = country_element.get_text(strip=True) if country_element else res.text

                # 檢視是否包含黑名單字眼
                for bug_word in BLACK_LIST_KEYWORDS:
                    if bug_word.lower() in country_text.lower():
                        print(f"❌ 發現異常！{company['name']} 標示為 {bug_word}")
                        send_alert(company["name"], bug_word, company["url"])
                        break
                else:
                    print(f"✅ {company['name']} 國別標示正常：Taiwan")
            else:
                print(f"⚠️ 無法讀取頁面 {company['name']} (Status: {res.status_code})")
        except Exception as e:
            print(f"⚠️ 檢查過程發生例外狀況 ({company['name']}): {e}")

if __name__ == "__main__":
    run_watchdog()
