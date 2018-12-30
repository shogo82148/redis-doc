DBをバックグラウンドで保存します。 OKコードがすぐに返されます。 Redisはフォークし、親はクライアントにサービスを提供し続け、子はDBをディスクに保存して終了します。クライアントは、 `LASTSAVE`コマンドを使用して操作が成功したかどうかを確認することができます。

Please refer to the [persistence documentation](/topics/persistence) for detailed information.

@return

@simple-string-reply
