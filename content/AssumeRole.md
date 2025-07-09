```
aaaaa
```
### **Assume Role手順**

**1. プロファイル情報の確認**

まず、`~/.aws/config`ファイルを開いて、使用するプロファイルの情報を確認。

[default] 
region = ap-northeast-1 
output = json 

[profile my-profile]
role_arn = arn:aws:iam::xxxxxxxxxxxx:role/my-role 
mfa_serial = arn:aws:iam::xxxxxxxxxxxx:mfa/my-mfa-device
source_profile = default 
region = ap-northeast-1

**2. MFAコードの取得**

MFAデバイスから6桁のMFAコードを取得。

**3. AWS CLIを使用して一時的な認証情報を取得**  
`aws sts assume-role`コマンドを手動で実行。以下はその例です。各オプションはそれぞれ置き換えてください。

$ aws sts assume-role \
 --role-arn arn:aws:iam::xxxxxxxxxxxx:role/my-role \
 --serial-number arn:aws:iam::xxxxxxxxxxxx:mfa/my-mfa-device \
 --role-session-name my-session \
 --profile default \
 --duration-seconds 43200 \
 --token-code ${取得したMFAコード} \
 --region ap-northeast-1

### **各オプションの説明**

- **`--role-arn`**: Assume Roleを実行するためのIAMロールのARN。
- **`--serial-number`**: MFAデバイスのARN。
- **`--role-session-name`**: セッションの名前。任意の名前を指定可能。
- **`--profile`**: 使用するAWS CLIプロファイル。このプロファイルに設定されている認証情報を用いてIAMロールを引き受けることができる。
- **`--duration-seconds`**: セッションの有効期間（秒単位）。[デフォルトだと1時間で、最大12時間(43200秒)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use.html)まで指定することができます。
- **`--token-code`**: MFAデバイスから取得した6桁のコード。
- **`--region`**: AWSのリージョン。

このコマンドを実行すると、以下のようなJSON形式の出力が得られます。

取得したJSONに含まれる情報は、AWSから発行された一時的な認証情報です。これらの認証情報は、指定された期間にわたって有効です。

```
{
  "Credentials": {
    "AccessKeyId": "ASIAxxxxxx",
    "SecretAccessKey": "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy",
    "SessionToken": "zzzzzzzzzzzz",
    "Expiration": "2024-5-17T12:34:56Z"
  }
}
```

**5. 取得した認証情報を環境変数に設定**  
取得した一時的な認証情報は、環境変数として設定することで、AWS CLIを使用してAWSリソースにアクセスする際に使用できます。 これらを手動で環境変数に設定。

$ export AWS_ACCESS_KEY_ID="ASIAxxxxxx"
$ export AWS_SECRET_ACCESS_KEY="yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
$ export AWS_SESSION_TOKEN="zzzzzzzzzzzz"

**6. 正しく設定されているかを確認**

$ aws sts get-caller-identity

正しくAssume Roleできているかを確認します。以上でCLI上でのAssume Roleをすることができました。

しかし、これをセッションが切れるたびに毎回やるのはめんどくさいですよね…。コマンドの入力ミスや、環境変数の設定ミスが発生しやすくなります。