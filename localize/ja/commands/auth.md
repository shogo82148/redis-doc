パスワードで保護されたRedisサーバーで認証を要求します。クライアントにコマンドの実行を許可する前に、パスワードを要求するようにRedisに指示することができます。これは設定ファイルの`requirepass`ディレクティブを使って行われます。

If `password` matches the password in the configuration file, the server replies
with the `OK` status code and starts accepting commands.
Otherwise, an error is returned and the clients needs to try a new password.

**注** ：Redisは高性能であるため、非常に短時間で多数のパスワードを同時に試行することが可能です。このため、この攻撃が実行不可能になるように強力で非常に長いパスワードを生成するようにしてください。

@return

@simple-string-reply
