# Jリーグ試合結果分析スクリプト ドキュメント

このドキュメントは、Google Apps Scriptで作成された「Jリーグ試合結果分析ツール（Ver 15.1）」の全体像、仕様、および利用方法を定義するものです。

---
---

## 【A】 人間向けドキュメント：運用・活用ガイド

このドキュメントは、このツールを**「使う人」「引き継ぐ人」**が、ツールの全体像と使い方を直感的に理解することを目的とします。

### 第1章：はじめに - このツールでできること

#### 1.1. プログラムの概要と目的
このツールは、Googleスプレッドシート上で動作するカスタムスクリプトです。Jリーグの試合結果データ（年度、大会、節、試合日、ホーム、スコア、アウェイ、入場者数などを含むリスト）をインプットとして、ボタン一つで以下のような複数の分析レポートを、**新しい別のスプレッドシートファイルとして自動生成**します。手動での集計作業をなくし、迅速かつ正確なデータ分析を支援することが目的です。

#### 1.2. システム構成図
本システムは、**データと設定を管理する「マスターファイル」**と、**分析結果を出力する「レポートファイル」**が明確に分離されています。

-   **マスターファイル（このスプレッドシート）**
    -   **入力シート**: あなたが試合結果データを貼り付けるためのシートです。
    -   **`ClubList`シート**: チーム名の表記揺れ（例：「川崎Ｆ」と「川崎フロンターレ」）を吸収するための、非常に重要な「名寄せリスト」を管理します。
    -   **Apps Script**: すべてのデータ処理ロジックが格納されています。

-   **レポートファイル（自動生成される新規ファイル）**
    -   スクリプトを実行すると、あなたのGoogleドライブのルートフォルダに、分析結果が格納された新しいスプレッドシートが作成されます。
    -   これにより、元のマスターファイルを汚すことなく、安全に分析結果を共有・保管できます。

#### 1.3. 分析結果シートの紹介
生成されるレポートファイルには、以下のシートが含まれます。
-   **`最新結果`**: Jリーグ公式サイトのような、勝ち点・得失点差・総得点でソートされた順位表です。
-   **`勝ち点表`**: チームごと、節ごとの勝ち点の推移がわかる表です。
-   **`得失点差表`**: 各節終了時点での、チームごとの累積得失点差の推移がわかる表です。
-   **`ホーム入場者数`**: 各節終了時点での、チームごとの累積ホームゲーム入場者数がわかる表です。
-   **`ログ`**: どのデータ行が、なぜ処理できなかったか（名寄せ失敗、スコア形式不正など）が記録されるデバッグ用のシートです。
-   **`テスト結果`**: スクリプト内部の計算ロジックが正しいかを自動検証した結果が出力されるシートです。

### 第2章：ユースケース - こんな時に役立ちます

-   **分析担当者のストーリー**:
    毎週更新されるJリーグの試合結果。これを手作業で順位表や勝ち点推移にまとめるのは時間がかかり、計算ミスも怖い...。このツールを使えば、最新データをマスターファイルに貼り付けてメニューから「分析結果を新規ファイルで作成」をクリックするだけ。数秒後には、整形された全レポートが格納された新しいスプレッドシートが完成。URLをコピーして、すぐにチーム内に共有できます。

-   **想定される活用シーン**:
    -   週次の定例ミーティング用の最新順位表作成。
    -   シーズン終了後の、全チームの勝ち点や得失点差の推移を比較・分析する際の基礎データとして。
    -   特定のチームのファンが、応援するチームの戦績を多角的に可視化するために。

### 第3章：利用マニュアル

#### 3.1. 【初回のみ】環境構築の手順
1.  **マスターファイルの準備**: このスクリプトが格納されたスプレッドシートをマスターファイルとします。
2.  **`ClubList`シートの作成**:
    -   シート名を`ClubList`とします。
    -   **A列**: リーグ名 (例: J1), **B列**: 正式名称 (例: 川崎フロンターレ), **C列以降**: 略称 (例: 川崎Ｆ, 川崎) を入力します。
3.  **名前付き範囲の設定**:
    -   `ClubList`シートのデータ範囲全体（ヘッダー含む）を選択します。
    -   メニューの`データ` > `名前付き範囲`から、範囲に`tbl_ClubList`という名前を付けます。
4.  **スクリプトの導入**:
    -   `拡張機能` > `Apps Script`を開き、提供された**Ver 15.1**の`コード.gs`を貼り付けて保存します。
5.  **マニフェストファイルの設定**:
    -   `プロジェクトの設定`（歯車アイコン） > `「appsscript.json」マニフェスト ファイルをエディタで表示する`にチェック。
    -   表示された`appsscript.json`ファイルに、指定された`oauthScopes`を追記して保存します。

#### 3.2. 【重要】初回実行時の権限承認ガイド
1.  初めてメニューからスクリプトを実行すると、「**承認が必要です**」というダイアログが表示されます。「**権限を確認**」をクリックします。
2.  自分のGoogleアカウントを選択します。
3.  「**このアプリは Google で確認されていません**」という警告画面が表示されます。これは、あなたが作成したカスタムスクリプトを実行するための正常なセキュリティチェックです。
4.  左下の「**詳細**」リンクをクリックします。
5.  画面下部に表示される「**（プロジェクト名）（安全ではないページ）に移動**」というリンクをクリックします。
6.  スクリプトが要求する権限の一覧が表示されます。内容を確認し、右下の「**許可**」ボタンをクリックします。これで認証は完了です。

#### 3.3. 日々の運用手順
1.  マスターファイル内の入力データ用シートに、最新の試合結果データを貼り付けます。
2.  メニューの「**Jリーグデータ処理**」>「**分析結果を新規ファイルで作成**」をクリックします。
3.  確認ダイアログで「OK」をクリックします。
4.  処理完了後、表示されるメッセージボックス内のURLをクリックし、新しく生成されたレポートファイルを確認します。
5.  必要に応じて、レポートファイルを任意のフォルダに移動・整理します。

#### 3.4. メンテナンスとトラブルシューティング
-   **データが欠損する場合**: まず`ログ`シートを確認します。「名寄せ失敗」とあれば、そのチームの略称を`ClubList`シートに追加します。「スコア形式不正」とあれば、入力データのスコアの表記を確認します。
-   **チームの昇格・降格があった場合**: `ClubList`シートのA列（リーグ名）を修正します。新しいチームが追加された場合は、行を追加して情報を入力します。
-   **名前付き範囲の更新**: `ClubList`シートを更新した後は、手動で`tbl_ClubList`の範囲を更新する必要はありません。スクリプトが実行時に自動で範囲を更新します。

<br>
<hr>
<br>

## 【B】 AI向けドキュメント：システム仕様書 (Specification)

---

### 1. 基本情報 (Basic Information)
-   **1.1. Script Name**: Jリーグ試合結果分析スクリプト (J-League Match Result Analysis Script)
-   **1.2. Version**: 15.1
-   **1.3. Environment**: Google Apps Script (Container-bound to Google Sheets)
-   **1.4. Required Scopes (`appsscript.json`)**:
    -   `https://www.googleapis.com/auth/spreadsheets.create`: スプレッドシートを新規に作成するために必要。
    -   `https://www.googleapis.com/auth/drive.file`: スクリプト自身が作成したファイルへのアクセス（シート追加や編集）のために必要。

-   **1.5. セキュリティに関する注意喚起 (Security Advisory)**:
    -   **権限の範囲**: 上記スコープは、**既存のファイルやフォルダへのアクセスを許可しません。** スクリプトは、新しいスプレッドシートを作成し、その作成したファイル内を編集する権限のみを持ちます。
    -   **共有ポリシー**: 生成されたレポートファイルの共有設定は、実行者のGoogleドライブのデフォルト設定に依存します。機密情報を含む場合は、生成されたファイルの共有設定を個別に確認し、意図しない相手に公開されていないかを確認する必要があります。
    -   **信頼性**: このスクリプトのコードは、ユーザー（あなた）とAIの対話によって作成されたものです。第三者が作成したものではないため、悪意のあるコードは含まれていません。権限承認は、この事実を理解した上で実行してください。

### 2. 定数定義 (Constants Definition - `CONFIG` Object)
-   **2.1. Sheet Names**:
    -   `LATEST_RESULT_SHEET_NAME`: '最新結果'
    -   `POINT_TABLE_SHEET_NAME`: '勝ち点表'
    -   `GOAL_DIFF_SHEET_NAME`: '得失点差表'
    -   `ATTENDANCE_SHEET_NAME`: 'ホーム入場者数'
    -   `LOG_SHEET_NAME`: 'ログ'
    -   `TEST_RESULT_SHEET_NAME`: 'テスト結果'
-   **2.2. Normalization Settings**:
    -   `NORMALIZATION_TABLE_NAME`: 'tbl_ClubList' (The Named Range for the club list table)
    -   `NORMALIZATION_SHEET_NAME`: 'ClubList' (The sheet containing the club list)
-   **2.3. Header Keywords**:
    -   Keywords to identify columns in the input data sheet: '年度', '大会', '節', '試合日', 'ホーム', 'スコア', 'アウェイ', '入場者数'.
-   **2.4. Header Definitions**:
    -   Defines the header names for output sheets (e.g., 'チーム名', '順位', '勝点').
-   **2.5. Points System**:
    -   `WIN`: 3, `DRAW`: 1, `LOSE`: 0.

### 3. 機能要件 (Functional Requirements)
-   **3.1. 入力 (Input)**
    -   The script must read data from the currently active sheet in the spreadsheet.
    -   It must use `getDisplayValues()` to retrieve data as strings to prevent automatic type conversion.
    -   It must dynamically locate the header row based on `CONFIG.HEADER_KEYWORDS`.
    -   It must extract the 'Year' and 'Competition' from the first valid data row for use in the output filename.
-   **3.2. データ正規化 (Data Normalization)**
    -   The script must create a normalization map (a `Map` object) from the sheet named `CONFIG.NORMALIZATION_SHEET_NAME` and the Named Range `CONFIG.NORMALIZATION_TABLE_NAME`.
    -   The map's keys shall be team abbreviations (from column C onwards) and the formal team name itself (from column B). The map's values shall be the formal team name (from column B).
    -   This map must be used to convert team names from the input data into their formal, unified names.
-   **3.3. データ処理 (Data Processing)**
    -   The script shall iterate through each row of the input data.
    -   For each valid match row, it must calculate and aggregate the following data for both home and away teams: Points, Win/Draw/Lose counts, Goals For (GF), Goals Against (GA), Goal Difference (GD) for that match, and Home Attendance.
    -   All aggregated data should be stored in structured objects (`points`, `goalDiffs`, `attendances`, `summary`).
-   **3.4. 出力 (Output)**
    -   The script must create a new Google Spreadsheet file using `SpreadsheetApp.create()`.
    -   The filename must be in the format: `Jリーグ結果分析出力_[Competition]_[Year]_[YYYY-MM-DD_HH-mm-ss]`.
    -   The new file must be saved to the user's root Google Drive folder.
    -   The script must generate the following sheets within the new spreadsheet, with data sorted according to the ranking logic:
        -   **`最新結果`**: A summary table. Rows must be sorted by: 1. Total Points (desc), 2. Total Goal Difference (desc), 3. Total Goals For (desc).
        -   **`勝ち点表`**: A table showing points per section for each team.
        -   **`得失点差表`**: A table showing the cumulative goal difference at the end of each section.
        -   **`ホーム入場者数`**: A table showing the cumulative home attendance at the end of each section.
    -   The `勝ち点表`, `得失点差表`, and `ホーム入場者数` sheets must have their rows sorted in the same order as the `最新結果` sheet.
-   **3.5. 補助機能 (Sub-features)**
    -   **Logging**: Generate a `ログ` sheet that records the processing status (`Success` or `Failure` with reasons) for every single row of the input data.
    -   **Testing**: Generate a `テスト結果` sheet that performs an internal consistency check, comparing data calculated via different methods (e.g., total points from `summary` vs. sum of points from `points` table).

### 4. 非機能要件 (Non-functional Requirements)
-   **4.1. UI/UX**:
    -   A custom menu named `Jリーグデータ処理` must be created upon opening the spreadsheet.
    -   The script must display a confirmation dialog before starting the main process.
    -   Upon completion, it must display a final notification dialog showing the processing time, the URL of the newly created spreadsheet, and a warning if any data integrity tests failed.
-   **4.2. メンテナンス性 (Maintainability)**:
    -   The Named Range for the club list (`tbl_ClubList`) must be updated automatically at the beginning of the script execution to reflect any additions or deletions.
    -   Configuration values (sheet names, keywords, etc.) must be centralized in the `CONFIG` object for easy modification.