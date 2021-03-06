[追記専用ファイル](/topics/persistence#append-only-file)の書き換えプロセスを開始するようにRedisに指示します。この書き換えは、現在の追記専用ファイルの小さな最適化バージョンを作成します。

`BGREWRITEAOF`が失敗した場合、古いAOFは変更されないのでデータは失われません。

永続化を行うバックグラウンドプロセスがまだない場合にのみ、書き換えはRedisによってトリガーされます。具体的には：

- Redisの子がディスク上にスナップショットを作成している場合、AOFの書き換えは*スケジュールされます*が、RDBファイルを作成している保存中の子が終了するまで開始されません。この場合、 `BGREWRITEAOF`はまだOKコードを返しますが、適切なメッセージを伴っています。 Redis 2.6以降の`INFO`コマンドでAOFの書き換えがスケジュールされているかどうかを確認できます。
- AOF書き換えがすでに進行中の場合、コマンドはエラーを返し、AOF書き換えは後でスケジュールされません。

Redis 2.4以降、AOF書き換えはRedisによって自動的に起動されますが、 `BGREWRITEAOF`コマンドを使用していつでも書き換えを起動できます。

詳細については、 [永続化ドキュメント](/topics/persistence)を参照してください。

@return

@simple-string-reply: 常に `OK` を返します。
