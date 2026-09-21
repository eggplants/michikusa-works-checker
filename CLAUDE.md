# michikusa-works-checker

桃色CODE「【耳かき】道草屋【安眠】」シリーズ（DLsite）の買い忘れチェックページ。

## 作品一覧の更新手順

新作が出たとき、価格が変わったときは `WORKS` を更新する。

1. `/chrome` で[サークルページ](https://www.dlsite.com/maniax/circle/profile/=/order%5B0%5D/release_d/options%5B0%5D/JPN/options%5B1%5D/NM/per_page/100/show_type/1/hd/1/lang_options%5B0%5D/%E6%97%A5%E6%9C%AC%E8%AA%9E/lang_options%5B1%5D/%E8%A8%80%E8%AA%9E%E4%B8%8D%E8%A6%81/page/1/maker_id/RG24350.html)を開く。「56件中」のような件数表示が `WORKS.length` と一致すれば新作なし。
2. `javascript_tool` で ID・タイトル・価格・販売日を抜く。`RJ` 番号だけを取り出してタブ区切りで返す。出力は途中で切れるので `slice()` で 20 件ずつ取る。
   ```js
   [...document.querySelectorAll('.work_name a')].slice(0, 20).map(a =>
     (a.getAttribute('href').match(/RJ\d+/) || ['?'])[0] + '\t' + a.textContent.trim()
     + '\t' + a.closest('li,tr').querySelector('.work_price')?.textContent.trim().replace(/\s+/g, '')
     + '\t' + a.closest('li,tr').querySelector('.sales_date')?.textContent.trim()
   ).join('\n')
   ```
   年齢区分は `get_page_text` の出力で各作品のタグ列に `全年齢` / `R-15` があるかで判定する。どちらも無ければ `R-18`。
3. `WORKS` の先頭（販売日降順）に追記する。フィールドは `id`, `title`, `date`(YYYY-MM-DD), `price`(数値), `age`(`全年齢` / `R-15` / `R-18`)。
   - `age` が `全年齢` の作品だけ DLsite リンクが `home` フロアになる（`workUrl()`）。それ以外は `maniax`。
4. `FETCHED_AT` を更新日に変える。
5. 検証: Node で件数・重複・並び順を確認する。
   ```sh
   node -e '
   const html=require("fs").readFileSync("index.html","utf8");
   const W=eval(html.match(/const WORKS = (\[[\s\S]*?\n      \]);/)[1]);
   const ids=new Set(W.map(w=>w.id));
   console.log(W.length, ids.size, W.every((w,i)=>i==0||W[i-1].date>=w.date));
   for(const w of W) for(const c of w.includes??[]) if(!ids.has(c)) console.log("MISSING",c);'
   ```

## 方針

- 外部依存・ビルド工程は増やさない。
- チェック状態の保存先は `localStorage`（キー `michikusa-owned`）。キー名を変えると既存ユーザーのチェックが消えるので変えない。
