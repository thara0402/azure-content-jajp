<properties 
	pageTitle="Data Factory の関数およびシステム変数 | Microsoft Azure" 
	description="Azure Data Factory の関数およびシステム変数の一覧を提供する" 
	documentationCenter="" 
	authors="spelluru" 
	manager="jhubbard" 
	editor="monicar"
	services="data-factory"
/>

<tags 
	ms.service="data-factory" 
	ms.workload="data-services" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="02/18/2016" 
	ms.author="spelluru"/>

# Azure Data Factory - 関数およびシステム変数
この記事では、Azure Data Factory でサポートされている関数および変数に関する情報を提供します。
  
## Data Factory のシステム変数

変数名 | 説明 | オブジェクトのスコープ | JSON のスコープと使用事例
------------- | ----------- | ------------ | ------------------------
WindowStart | 現在のアクティビティ実行ウィンドウの時間間隔の開始 | アクティビティ | <ol><li>データ選択クエリを指定します。[データ移動アクティビティ](data-factory-data-movement-activities.md)の記事で参照されているコネクタの記事を参照してください。</li><li>パラメーターを Hive スクリプトに渡します (上のサンプルを参照してください)。</li>
WindowEnd | 現在のアクティビティ実行ウィンドウの時間間隔の終了 | アクティビティ | 上記と同じ
SliceStart | 生成されているデータ スライスの時間間隔の開始 | アクティビティ<br/>データセット | <ol><li>動的フォルダー パスとファイル名を指定し、[Azure BLOB](data-factory-azure-blob-connector.md) と[ファイル システム データセット](data-factory-onprem-file-system-connector.md)を操作します。</li><li>アクティビティ入力コレクションで Data Factory 関数を使用して入力の依存関係を指定します。</li></ol>
SliceEnd | 生成されている現在のデータ スライスの時間間隔の終了 | アクティビティ<br/>データセット | 上記と同じ。 

> [AZURE.NOTE] 現在、Data Factory では、アクティビティに指定されたスケジュールが、出力データセットの availability に指定されたスケジュールと正確に一致する必要があります。つまり、WindowStart、WindowEnd、SliceStart、および SliceEnd は、常に同じ期間と 1 つの出力スライスにマップされます。
 
## Data Factory の関数 

上記のシステム変数と Data Factory の関数を次の用途で使用できます。

1.	データ選択クエリを指定する ([データ移動アクティビティ](data-factory-data-movement-activities.md)の記事で参照されているコネクタの記事を参照してください。

	データ選択クエリと、アクティビティおよびデータセットの他のプロパティの場合、Data Factory 関数を呼び出す構文は **$$<function>** です。  
2. アクティビティ入力コレクションで、Data Factory 関数を使用して入力の依存関係を指定する (上記のサンプルを参照してください)。

	入力の依存関係式を指定する場合、$$ は不要です。

次のサンプルでは、JSON ファイルの **sqlReaderQuery** プロパティは、**Text.Format** 関数から返される値に割り当てられます。また、**WindowStart** というシステム変数も使用されます。この変数は、アクティビティ実行ウィンドウの開始時刻を表します。
	
	{
	    "Type": "SqlSource",
	    "sqlReaderQuery": "$$Text.Format('SELECT * FROM MyTable WHERE StartTime = \\'{0:yyyyMMdd-HH}\\'', WindowStart)"
	}

### 関数

次の表は、Azure Data Factory の全関数の一覧です。

カテゴリ | 関数 | パラメーター | 説明
-------- | -------- | ---------- | ----------- 
Time | AddHours(X,Y) | X: DateTime <p>Y: int</p> | 指定した時刻 X に Y 時間を追加します。<p>例: 9/5/2013 12:00:00 PM + 2 hours = 9/5/2013 2:00:00 PM</p>
Time | AddMinutes(X,Y) | X: DateTime <p>Y: int</p> | X に Y 分を追加します。<p>例: 9/15/2013 12: 00:00 PM + 15 minutes = 9/15/2013 12: 15:00 PM</p>
Time | StartOfHour(X) | X: Datetime | X の時間コンポーネントで表される時間の開始時刻を取得します。<p>例: StartOfHour of 9/15/2013 05: 10:23 PM is 9/15/2013 05: 00:00 PM</p>
Date | AddDays(X,Y) | X: DateTime<p>Y: int</p> | X に Y 日を追加します。<p>例: 9/15/2013 12:00:00 PM + 2 days = 9/17/2013 12:00:00 PM</p>
Date | AddMonths(X,Y) | X: DateTime<p>Y: int</p> | X に Y か月を追加します。<p>例: 9/15/2013 12:00:00 PM + 1 month = 10/15/2013 12:00:00 PM</p> 
Date | AddQuarters(X,Y) | X: DateTime <p>Y: int</p> | X に Y * 3 か月を追加します。<p>例: 9/15/2013 12:00:00 PM + 1 quarter = 12/15/2013 12:00:00 PM</p>
Date | AddWeeks(X,Y) | X: DateTime<p>Y: int</p> | X に Y * 7 日を追加します。<p>例: 9/15/2013 12:00:00 PM + 1 week = 9/22/2013 12:00:00 PM</p>
Date | AddYears(X,Y) | X: DateTime<p>Y: int</p> | X に Y * 3 年を追加します。<p>例: 9/15/2013 12:00:00 PM + 1 year = 9/15/2014 12:00:00 PM</p>
Date | Day(X) | X: DateTime | X の日付コンポーネントを取得します。<p>例: 9/15/2013 12:00:00 PM の日付は 9 です。</p>
Date | DayOfWeek(X) | X: DateTime | X の曜日コンポーネントを取得します。<p>例: 9/15/2013 12:00:00 PM の DayOfWeek は Sunday です。</p>
Date | DayOfYear(X) | X: DateTime | X の年コンポーネントで表される、その年の日付を取得します。<p>例:<br/>12/1/2015: 2015 年の 335 日目<br/>12/31/2015: 2015 年の 365 日目<br/>12/31/2016: 2016 年の 366 日目 (うるう年)</p>
Date | DaysInMonth(X) | X: DateTime | パラメーター X の月コンポーネントで表される、その月の日数を取得します。<p>例: 9 月は 30 日間なので、9/15/2013 の DaysInMonth は 30 です。</p>
Date | EndOfDay(X) | X: DateTime | X の日 (日コンポーネント) の終了を表す日時を取得します。<p>例: 9/15/2013 05:10:23 PM の EndOfDay は 9/15/2013 11:59:59 PM です。</p>
Date | EndOfMonth(X) | X: DateTime | パラメーター X の月コンポーネントで表される月の終了を取得します。<p>例: 9/15/2013 05:10:23 PM の EndOfMonth は 9/30/2013 11:59:59 PM です (9 月の終了を表す日時)。</p>
Date | StartOfDay(X) | X: DateTime | パラメーター X の日コンポーネントで表される日の開始を取得します。<p>例: 9/15/2013 05:10:23 PM の StartOfDay は 9/15/2013 12:00:00 AM です。</p>
DateTime | From(X) | X: String | 文字列 X を日時にパースします。
DateTime | Ticks(X) | X: DateTime | パラメーター X の ticks プロパティを取得します。1 ティックは 100 ナノ秒です。このプロパティの値は、0001 年 1 月 1 日深夜 12:00:00 以降の経過時間のティック数を表しています。 
Text | Format(X) | X: String 変数 | テキストの書式を設定します。

#### Text.Format の例

	"defines": { 
	    "Year" : "$$Text.Format('{0:yyyy}',WindowStart)",
	    "Month" : "$$Text.Format('{0:MM}',WindowStart)",
	    "Day" : "$$Text.Format('{0:dd}',WindowStart)",
	    "Hour" : "$$Text.Format('{0:hh}',WindowStart)"
	}

使用できるさまざまな書式設定のオプション (例: yyyy と yy) については、「[カスタム日時書式指定文字列](https://msdn.microsoft.com/library/8kb3ddd4.aspx)」のトピックを参照してください。

> [AZURE.NOTE] 別の関数内で関数を使用する場合、内側の関数に **$$** プレフィックスを付ける必要はありません。例: $$Text.Format('PartitionKey eq \\'my\_pkey\_filter\_value\\' and RowKey ge \\'{0:yyyy-MM-dd HH:mm:ss}\\'', Time.AddHours(SliceStart, -6)).この例でわかるように、**Time.AddHours** 関数に **$$** プレフィックスは使用されていません。
  

<!---HONumber=AcomDC_0224_2016-->