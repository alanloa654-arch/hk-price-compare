# 香港超市格價

手機／電腦都用得嘅單頁格價工具，比較消委會「格價資訊通」公開數據入面相同貨品喺各超市嘅售價。

## 點樣打開

1. 用瀏覽器直接打開本目錄嘅 `index.html`（連埋同目錄嘅 `data.js`）。
2. 唔使上網、唔使伺服器：資料已嵌入 `data.js`，離線都用得。
3. 建議用 Chrome／Safari／Firefox；手機加到主畫面亦可（已設 theme-color／apple-mobile-web-app）。

本機路徑示例：

```text
/workspace/hk-price-compare/index.html
```

## 功能摘要

- 以品牌／貨品名搜尋（約 150ms debounce）
- 大類篩選、超市多選篩選、最平／差價／名稱排序
- 標示最平價錢同「慳 $X」差價；有優惠字句會顯示
- HKTVmall 唔喺公開數據內：提供搜尋連結另開對價

## 資料來源

- Consumer Council Online Price Watch（消委會「格價資訊通」）
- 本頁為非官方工具，與消委會、各超市及 HKTVmall 無關

## 更新數據

若有新嘅 `pricewatch.json`：

```bash
python3 - <<'PY'
import json, os, datetime
from pathlib import Path
src = "/path/to/pricewatch.json"   # 改成新檔路徑
out = Path("/workspace/hk-price-compare/data.js")
with open(src, encoding="utf-8") as f:
    raw = json.load(f)
slim = []
for p in raw:
    item = {
        "c": p.get("code") or "",
        "b": (p.get("brand") or {}).get("zh-Hant") or (p.get("brand") or {}).get("en") or "",
        "n": (p.get("name") or {}).get("zh-Hant") or (p.get("name") or {}).get("en") or "",
        "c1": (p.get("cat1Name") or {}).get("zh-Hant") or "",
        "c2": (p.get("cat2Name") or {}).get("zh-Hant") or "",
        "p": [],
    }
    for pr in p.get("prices") or []:
        try:
            v = float(pr["price"])
        except (KeyError, TypeError, ValueError):
            continue
        item["p"].append({"s": pr.get("supermarketCode") or "", "v": v})
    offers = []
    for o in p.get("offers") or []:
        t = o.get("zh-Hant") or o.get("en") or ""
        if t:
            offers.append({"s": o.get("supermarketCode") or "", "t": t})
    if offers:
        item["o"] = offers
    slim.append(item)
mtime = datetime.datetime.fromtimestamp(os.stat(src).st_mtime)
meta = {"count": len(slim), "updated": mtime.strftime("%Y-%m-%d %H:%M"), "source": "Consumer Council Online Price Watch"}
out.write_text(
    "/* Auto-generated from Consumer Council Online Price Watch — do not edit by hand */\n"
    f"window.PRICE_META={json.dumps(meta, ensure_ascii=False, separators=(',',':'))};\n"
    f"window.PRICE_DATA={json.dumps(slim, ensure_ascii=False, separators=(',',':'))};\n",
    encoding="utf-8",
)
print("wrote", out, "products", len(slim))
PY
```

然後重新打開 `index.html` 即可。
