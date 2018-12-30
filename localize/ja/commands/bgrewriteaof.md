[追記専用ファイル](/topics/persistence#append-only-file)の書き換えプロセスを開始するようにRedisに指示します。この書き換えは、現在の追記専用ファイルの小さな最適化バージョンを作成します。

If `BGREWRITEAOF` fails, no data gets lost as the old AOF will be untouched.

The rewrite will be only triggered by Redis if there is not already a background
process doing persistence.
Specifically:

- If a Redis child is creating a snapshot on disk, the AOF rewrite is*scheduled* but not started until the saving child producing the RDB fileterminates.In this case the `BGREWRITEAOF` will still return an OK code, but with anappropriate message.You can check if an AOF rewrite is scheduled looking at the `INFO` commandas of Redis 2.6.
- If an AOF rewrite is already in progress the command returns an error and noAOF rewrite will be scheduled for a later time.

Since Redis 2.4 the AOF rewrite is automatically triggered by Redis, however the
`BGREWRITEAOF` command can be used to trigger a rewrite at any time.

Please refer to the [persistence documentation](/topics/persistence) for detailed information.

@return

@simple-string-reply: 常に `OK` を返します。
