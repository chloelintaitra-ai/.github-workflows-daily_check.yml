# .github-workflows-daily_check.yml
VITAFOODS WATCH
name: Vitafoods Watchdog

on:
  schedule:
    - cron: '0 0 * * *' # 每天台灣時間 08:00 自動執行
  workflow_dispatch: # 支援在網頁上隨時手動點擊執行

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: pip install requests beautifulsoup4
      - name: Run script
        run: python watchdog.py
