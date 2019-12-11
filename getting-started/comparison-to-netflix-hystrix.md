Netflix Hystrixとの比較
======================

Netflix Hystrixとの違い:

- Hystrixでは、外部システム呼び出しはHystrixCommandでラップされる必要があります。対してこのライブラリは、関数型インタフェース、ラムダ式、メソッド参照をサーキットブレイカー、流量制限、リトライ、バルクヘッドで拡張するための高階関数（デコレーター）を提供します。それに加えて、失敗した呼び出しをリトライしたり、呼び出し結果をキャッシュするデコレーターも提供します。関数型インタフェース、ラムダ式、メソッド参照には、1つ以上のデコレーターをスタックできます。つまり、サーキットブレイカーデコレーターにバルクヘッド、流量制御、リトライのデコレーターを組み合わせることができます。これにより、必要なデコレーターを使うか、何も使わないかを選ぶことができます。全てのデコレートされた関数は、同期または非同期（CompletableFutureまたはRxJavaを利用する）で実行できます。
- Hystrixは、デフォルトでは10個の1秒ウィンドウバケツに実行結果を保存します。1秒ウィンドウバケツが経過したら、新しいバケツが作られて古いものは破棄されます。このライブラリは、ローリング時間ウィンドウ無しのリングビットバッファーに実行結果を保存します。成功した呼び出しは0のビット、失敗した呼び出しは1のビットとして保存されます。リングビットバッファーは設定可能な固定長とlong[]配列（boolean配列と比較すると省メモリ）を持っています。つまり、リングビットバッファーは1024個の呼び出しを保存するのに長さ16のlong（64ビット）配列しか必要としません。これにより、このサーキットブレイカーは呼び出しが頻繁な（またはそうでない）バックエンドシステムで使うことが出来ます。時間ウィンドウが経過しても、実行結果が破棄されないからです。
- Hystrixはhalf-open状態の際、サーキットブレイカーをcloseにするか否かを決めるために1回しか実行をしません。このライブラリは実行回数を設定可能です。サーキットブレイカーをcloseするか否かを決定するために、設定可能な閾値と結果を比較します。
- このライブラリはReactorおよびRxJavaのカスタムオペレーターを提供しています。これにより、全リアクティブ型をサーキットブレイカー、バルクヘッド、流量制御でデコレートできます。
- Hystrixとこのライブラリは、システム管理者が実行結果やレイテンシーのメトリクスを監視するのに便利なイベントストリームを出力します。