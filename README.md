# Snyk  Infrastruture as Code (IaC) ワークショップ

Snyk Infrastructure as Codeにより、Kubernetes、Helm、Terraform、CloudFormationの設定ファイルの脆弱性を発見し修正することができます。

Snyk を使用した開発者向けの Infrastructure as Codeによるセキュリティでは、Terraformモジュールや Kubernetes YAML、JSON、Helm チャートをテストおよび監視して、プロビジョニングにおいて外部からの悪意のある動作にさらす可能性のある設定ミスを検出することが可能です。

この**ハンズオン**では以下をカバーします。

- [Snyk  Infrastruture as Code (IaC) ワークショップ](#snyk--infrastruture-as-code-iac-ワークショップ)
  - [事前準備](#事前準備)
- [Workshop Steps](#workshop-steps)
  - [Step 1 - GitHub IaCリポジトリのFork](#step-1---github-iacリポジトリのfork)
  - [Step 2 - GitHub Integrationの設定](#step-2---github-integrationの設定)
  - [Step 3 脆弱性を発見するプロジェクトの追加](#step-3-脆弱性を発見するプロジェクトの追加)
  - [Step 4 - Snyk CLIでスキャンをしてみる](#step-4---snyk-cliでスキャンをしてみる)
  - [Step 5 - Snyk CLIでAWS CloudFormationをスキャンしてみる](#step-5---snyk-cliでaws-cloudformationをスキャンしてみる)
  - [Step 6 - Snyk CLIでKubernetesのマニフェストファイルをスキャンしてみる](#step-6---snyk-cliでkubernetesのマニフェストファイルをスキャンしてみる)
  - [Step 7 - Snyk IaCの検知するルールを確認してみよう](#step-7---snyk-iacの検知するルールを確認してみよう)

## 事前準備 

* GitHubアカウント（パブリックであること） - http://github.com
* git CLI - https://git-scm.com/downloads
* snyk CLI - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
* Snykアカウント - http://app.snyk.io

# Workshop Steps

_**注: この手順はMacを想定していますが、WindowsやLinuxでもスクリプトを一部修正することで対応できます。**_

## Step 1 - GitHub IaCリポジトリのFork

このGitHubリポジトリを開いてください。- https://github.com/snyk-japan/snyk-iac-workshop

* "**Fork**"ボタンをクリックします
* 皆様の個人用パブリックアカウントへ**Fork**先が選択されていることを確認ください。
* **done**を押してForkします

## Step 2 - GitHub Integrationの設定

まず、Snyk を GitHub に接続し、リポジトリをインポートする必要があります。以下の手順で行います。

* http://app.snyk.ioにログインします。もしアカウントがなければ、ここでサインアップしてください。
* Integrations → Source Control → GitHubに進んでください。
* GitHub Integrationに必要なGitHubのクレデンシャルを入力してください。

<img width="1246" alt="github_integration" src="https://user-images.githubusercontent.com/45160975/207768680-195af3ef-0c65-4773-867e-3d1b6648ac3b.png">

## Step 3 脆弱性を発見するプロジェクトの追加

* このステップを行う前にIaCスキャンが有効になっているか確認ください。"**Settings → Infrastructure as code**"から確認および有効化ができます。

<img width="1885" alt="iac_setting" src="https://user-images.githubusercontent.com/45160975/207768857-863cdccf-ecd2-45e3-a0ae-6f94b44ae098.png">

GitHubとの連携ができたので、次はGitHubレポジトリをSnykのプロジェクトとしてインポートします。

* Snyk UI上でProjectsをクリックします
* "**Add Project**" → "**GitHub**"
* Forkした"**snyk-iac-workshop**"を選択します

![alt tag](https://i.ibb.co/pWJW1VK/snyk-iac-1.png)

インポートとスキャンが成功すると以下のような画面が表示されます。

![alt tag](https://i.ibb.co/YNC5rfd/snyk-iac-2.png)

* "**terraform/big_data.tf**"というTerraformファイルを見てみましょう。

![alt tag](https://user-images.githubusercontent.com/45160975/193725351-2cbf2c32-e017-4229-a06c-d8b6b3ed9b93.png)

発見された脆弱性について、Snykは以下の情報を表示します。

1. 脆弱性の見つかったコードのスニペットと深刻度
1. その脆弱性についてのSnyk Policyへのリンクと修正方法
1. Ignore設定

![alt tag](https://user-images.githubusercontent.com/45160975/193725099-49ac9d4c-30a2-43db-aaea-f150c701c506.png)

注: このあとのステップで発見された脆弱性の一部を修正していきます。今はSnyk UIに慣れるために、Snyk policyのリンクや **Show more details**　などにどんな情報があるのか確認してみてください。

## Step 4 - Snyk CLIでスキャンをしてみる

前のステップでやった通り、Snykはソースコードリポジトリから Terraformファイルをテスト・監視します（GitHub連携)。そして、クラウド環境のセキュリティを向上させるためのアドバイスを提供し、本番環境に移行する前に設定ミスを検出し、修正を支援します。

SnykのGitHub連携に加えて、手元のコードベースに対してスキャンを行うCLIもあります。
このステップではCLIを使ってスキャンを行なってみます。

_**注: インストールしたSnyk CLIのバージョンが以下のバージョンと同じかそれ以上か確認してください。Snyk CLIのインストールおよびアップグレードはこちらになります。https://docs.snyk.io/snyk-cli/install-the-snyk-cli**_

```bash
$ snyk --version
1.675.0
```

* Snyk CLIで認証を行う

```bash
$ snyk auth

Now redirecting you to our auth page, go ahead and log in,
and once the auth is complete, return to this prompt and you'll
be ready to start using snyk.

If you can't wait use this url:
https://snyk.io/login?token=ff75a099-4a9f-4b3d-b75c-bf9847672e9c&utm_medium=cli&utm_source=cli&utm_campaign=cli&os=darwin&docker=false


Your account has been authenticated. Snyk is now ready to be used.
```

ブラウザでSnyk UIが開きますので、そちらから認証を行なってください。

* Forkしたレポジトリをローカル環境へCloneしてください。

_**注： このレポジトリではなく、皆様がForkしたGitレポジトリをCloneしてください。**_

```bash
$ git clone https://github.com/snyk-japan/snyk-iac-workshop
Cloning into 'snyk-iac-workshop'...
remote: Enumerating objects: 27, done.
remote: Counting objects: 100% (27/27), done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 27 (delta 5), reused 25 (delta 3), pack-reused 0
Receiving objects: 100% (27/27), 12.64 KiB | 1.05 MiB/s, done.
Resolving deltas: 100% (5/5), done.
```

* "**snyk-iac-workshop**"ディレクトリへ移動してください

```bash
$ cd snyk-iac-workshop
```

* それでは"**big_data.tf**"をスキャンしてみましょう。以下のコマンドでスキャンを行います。ここでは、単一のTerraformファイルを指定していますが、ディレクトリ単位でスキャンすることも可能です。

```bash
$ snyk iac test terraform/big_data.tf 

Snyk Infrastructure as Code

✔ Test completed.

Issues

Low Severity Issues: 7

  [Low] The log_checkpoints is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_checkpoints' is disabled. Verbose logging information
           of database will not be collected
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-287
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_checkpoints"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] The log_connections setting is disabled on Postgresql DB
  Info:    PostgreSQL 'log_connections' is disabled. Connection logs will not be
           available for troubleshooting or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-288
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_connections"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] The log_disconnections setting is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_disconnections' is disabled. Disconnection logs will
           not be available for troubleshooting or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-289
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_disconnections"` and `settings.database_flags.value` attribute
           to `"on"`

  [Low] The log_lock_waits setting is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_lock_waits' is disabled. Deadlock timeouts logs will
           not be available for troubleshooting, or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-290
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_lock_waits"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] Temporary file information is not logged
  Info:    PostgreSQL 'log_temp_files' is not set to 0. Some temporary files
           information may not be logged, which may impact ability to identify
           potential performance issues
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-291
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_temp_files"`, and `settings.database_flags.value` attribute to
           `0`

  [Low] SQL statements may be logged
  Info:    PostgreSQL 'log_min_duration_statement' is not set to -1. Some SQL
           statements may be logged and expose sensitive information
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-292
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_min_duration_statement"` and `settings.database_flags.value`
           attribute to `-1`

  [Low] PostgreSQL instance database flag `log_min_messages` not set appropriately
  Info:    PostgreSQL instance database flag `log_min_messages` not set
           appropriately. Not configuring the appropriate log level could lead
           to missing important information that negatively impact operations of
           the database
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-687
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           database_flags
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` to `log_min_messages` and
           `settings.database_flags.value` to an appropriate value for your
           needs. We recommend at least `ERROR` or below.

Medium Severity Issues: 2

  [Medium] Cloud SQL instance backup disabled
  Info:    Automated backup is explicitly disabled. Data will not be recoverable
           in the event of failure or malicious attack
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-283
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           backup_configuration
  File:    terraform/big_data.tf
  Resolve: Set `settings.backup_configuration.enabled` attribute to `true`

  [Medium] Cloud IAM not configured for CloudSQL instance
  Info:    Cloud IAM not configured for CloudSQL instance. Dismissing Cloud IAM
           makes access control management more difficult and error-prone
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-693
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           database_flags
  File:    terraform/big_data.tf
  Resolve: Configure a `settings.database_flags` block, with `name` as
           `cloudsql_iam_authentication` and `value` as `On`

High Severity Issues: 3

  [High] Cloud SQL instance is publicly accessible
  Info:    Cloud SQL database instance allows public access. Potentially anyone
           can establish network connectivity to the database instance
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-235
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           ip_configuration > authorized_networks[0]
  File:    terraform/big_data.tf
  Resolve: Set `settings.ip_configuration.authorized_networks` attribute to
           specific value e.g. `192.168.0.0/24`

  [High] BigQuery dataset is publicly accessible
  Info:    BigQuery dataset is publicly accessible. Potentially anyone can
           access data in the dataset
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-236
  Path:    resource > google_bigquery_dataset[dataset] > access[0] >
           special_group
  File:    terraform/big_data.tf
  Resolve: Remove `allAuthenticatedUsers` and `allUsers` values from
           `access.special_group` attribute

  [High] Public IP assigned to SQL database instance
  Info:    Public IP will be assigned to the SQL database. Database can be
           accessed remotely over public Internet
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-242
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           ip_configuration > ipv4_enabled
  File:    terraform/big_data.tf
  Resolve: Set `settings.ip_configuration.ipv4_enabled` attribute to `false`

-------------------------------------------------------

Test Summary

  Organization: masatomo.ito-qx0
  Project name: snyk-japan/snyk-iac-workshop

✔ Files without issues: 0
✗ Files with issues: 1
  Ignored issues: 0
  Total issues: 12 [ 0 critical, 3 high, 2 medium, 7 low ]

-------------------------------------------------------

Tip

  New: Share your test results in the Snyk Web UI with the option --report

```

* いくつかを修正してみましょう。

```bash
  [Medium] Cloud SQL instance backup disabled
  Info:    Automated backup is explicitly disabled. Data will not be recoverable
           in the event of failure or malicious attack
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-283
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           backup_configuration
  File:    terraform/big_data.tf
  Resolve: Set `settings.backup_configuration.enabled` attribute to `true`
```

Cloud SQL instanceのバックアップが無効になっています、と警告しています。
また、これに紐づいているSnyk Policyへのリンクも載っています。https://snyk.io/security-rules/SNYK-CC-GCP-283

それらとともに、どのように修正すればいいか `Resolve` という情報も表示しています。

*  修正方法にしたがって、"**terraform/big_data.tf**" を以下のように修正してみましょう。`settings.backup_configureation.enabled` の値を `true` にするだけです。

```hcl
resource google_sql_database_instance "master_instance" {
  name             = "terragoat-${var.environment}-master"
  database_version = "POSTGRES_11"
  region           = var.region

  settings {
    tier = "db-f1-micro"
    ip_configuration {
      ipv4_enabled = true
      require_ssl = true
      authorized_networks {
        name  = "WWW"
        value = "0.0.0.0/0"
      }   
    }   
    backup_configuration {
      enabled = true
    }   
  }
}
```

* それでは再度 "**./terraform/big_data.tf**" をスキャンしてみましょう。

```bash
$ snyk iac test terraform/big_data.tf 

Snyk Infrastructure as Code

✔ Test completed.

Issues

Low Severity Issues: 7

  [Low] The log_checkpoints is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_checkpoints' is disabled. Verbose logging information
           of database will not be collected
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-287
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_checkpoints"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] The log_connections setting is disabled on Postgresql DB
  Info:    PostgreSQL 'log_connections' is disabled. Connection logs will not be
           available for troubleshooting or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-288
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_connections"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] The log_disconnections setting is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_disconnections' is disabled. Disconnection logs will
           not be available for troubleshooting or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-289
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_disconnections"` and `settings.database_flags.value` attribute
           to `"on"`

  [Low] The log_lock_waits setting is disabled on PostgreSQL DB
  Info:    PostgreSQL 'log_lock_waits' is disabled. Deadlock timeouts logs will
           not be available for troubleshooting, or investigations
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-290
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` attribute to `"log_lock_waits"`,
           and `settings.database_flags.value` attribute to `"on"`

  [Low] Temporary file information is not logged
  Info:    PostgreSQL 'log_temp_files' is not set to 0. Some temporary files
           information may not be logged, which may impact ability to identify
           potential performance issues
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-291
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_temp_files"`, and `settings.database_flags.value` attribute to
           `0`

  [Low] SQL statements may be logged
  Info:    PostgreSQL 'log_min_duration_statement' is not set to -1. Some SQL
           statements may be logged and expose sensitive information
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-292
  Path:    resource > google_sql_database_instance[master_instance] > settings
  File:    terraform/big_data.tf
  Resolve: Set the `settings.database_flags.name` attribute to
           `"log_min_duration_statement"` and `settings.database_flags.value`
           attribute to `-1`

  [Low] PostgreSQL instance database flag `log_min_messages` not set appropriately
  Info:    PostgreSQL instance database flag `log_min_messages` not set
           appropriately. Not configuring the appropriate log level could lead
           to missing important information that negatively impact operations of
           the database
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-687
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           database_flags
  File:    terraform/big_data.tf
  Resolve: Set `settings.database_flags.name` to `log_min_messages` and
           `settings.database_flags.value` to an appropriate value for your
           needs. We recommend at least `ERROR` or below.

Medium Severity Issues: 1

  [Medium] Cloud IAM not configured for CloudSQL instance
  Info:    Cloud IAM not configured for CloudSQL instance. Dismissing Cloud IAM
           makes access control management more difficult and error-prone
  Rule:    https://snyk.io/security-rules/SNYK-CC-GCP-693
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           database_flags
  File:    terraform/big_data.tf
  Resolve: Configure a `settings.database_flags` block, with `name` as
           `cloudsql_iam_authentication` and `value` as `On`

High Severity Issues: 3

  [High] Cloud SQL instance is publicly accessible
  Info:    Cloud SQL database instance allows public access. Potentially anyone
           can establish network connectivity to the database instance
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-235
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           ip_configuration > authorized_networks[0]
  File:    terraform/big_data.tf
  Resolve: Set `settings.ip_configuration.authorized_networks` attribute to
           specific value e.g. `192.168.0.0/24`

  [High] BigQuery dataset is publicly accessible
  Info:    BigQuery dataset is publicly accessible. Potentially anyone can
           access data in the dataset
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-236
  Path:    resource > google_bigquery_dataset[dataset] > access[0] >
           special_group
  File:    terraform/big_data.tf
  Resolve: Remove `allAuthenticatedUsers` and `allUsers` values from
           `access.special_group` attribute

  [High] Public IP assigned to SQL database instance
  Info:    Public IP will be assigned to the SQL database. Database can be
           accessed remotely over public Internet
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-242
  Path:    resource > google_sql_database_instance[master_instance] > settings >
           ip_configuration > ipv4_enabled
  File:    terraform/big_data.tf
  Resolve: Set `settings.ip_configuration.ipv4_enabled` attribute to `false`

-------------------------------------------------------

Test Summary

  Organization: masatomo.ito-qx0
  Project name: snyk-japan/snyk-iac-workshop

✔ Files without issues: 0
✗ Files with issues: 1
  Ignored issues: 0
  Total issues: 11 [ 0 critical, 3 high, 1 medium, 7 low ]

-------------------------------------------------------

Tip

  New: Share your test results in the Snyk Web UI with the option --report

```

この修正によりCloud SQL databaseのバックアップが有効化され、クラッシュしたり攻撃された場合でもバックアップによりリストアすることができるようになりました。

もし、興味と時間があれば他の脆弱性についても修正を試してみてください。また、修正した結果をGitHubレポジトリに対してもコミット・プッシュして、GitHub連携の方でも修正されているか試してみてください。

* CLIでのスキャン結果をUIで見てみる

CLIでのスキャンは非常に簡単で便利ですが、解析結果をグラフィカルなUIで見たい場合も多いと思います。そんな時に便利なのが `--report`　オプションです。これを指定すると解析結果をSnyk UIでも見れるようになります。

```bash
$ snyk iac test terraform/big_data.tf --report
```

解析結果が表示され、また以下のような出力があります。

```
Report Complete

  Your test results are available at: https://snyk.io/org/masatomo.ito-qx0/projects
  under the name: snyk-japan/snyk-iac-workshop
```

Snyk UIを開いてみると、新たなプロジェクトが作成され、そこからUIで見ることができます。

<img width="1131" alt="image" src="https://user-images.githubusercontent.com/45160975/193730533-63168ce7-9af3-4ab0-83a3-b0757decf070.png">

* テストフォーマットを `--json` オプションを使用してJSONとして出力することもできます。これにより、課題参照へのリンクを含むより詳細な情報が提供され、また報告目的のために他のシステムやカスタムツールなどにデータをアップロードすることができます。以下は出力例です、長いので途中は省略しています。

```json 
$ snyk iac test terraform/big_data.tf --json
{
  "meta": {
    "isPrivate": true,
    "isLicensesEnabled": false,
    "ignoreSettings": {
      "adminOnly": false,
      "reasonRequired": false,
      "disregardFilesystemIgnores": false
    },
    "org": "masatomo.ito-qx0",
    "orgPublicId": "c555e9ed-54b0-4889-b2db-328c565ea954",
    "policy": ""
  },
  "filesystemPolicy": false,
  "vulnerabilities": [],
  "dependencyCount": 0,
  "licensesPolicy": null,
  "ignoreSettings": null,
  "targetFile": "terraform/big_data.tf",
  "projectName": "snyk-iac-workshop",
  "org": "masatomo.ito-qx0",
  "policy": "",
  "isPrivate": true,
  "targetFilePath": "/Users/masa/demo/workshops/snyk-iac-workshop/terraform/big_data.tf",
  "packageManager": "terraformconfig",
  "path": "terraform/big_data.tf",
  "projectType": "terraformconfig",
  "ok": false,
  "infrastructureAsCodeIssues": [
    {
      "id": "SNYK-CC-TF-242",
      "title": "Public IP assigned to SQL database instance",
      "severity": "high",
      "isIgnored": false,
      "subType": "Cloud SQL",
      "documentation": "https://snyk.io/security-rules/SNYK-CC-TF-242",
      "isGeneratedByCustomRule": false,
      "issue": "Public IP will be assigned to the SQL database",
      "impact": "Database can be accessed remotely over public Internet",
      "resolve": "Set `settings.ip_configuration.ipv4_enabled` attribute to `false`",
      "lineNumber": 9,
      "iacDescription": {
        "issue": "Public IP will be assigned to the SQL database",
        "impact": "Database can be accessed remotely over public Internet",
        "resolve": "Set `settings.ip_configuration.ipv4_enabled` attribute to `false`"
      },
      "publicId": "SNYK-CC-TF-242",
      "msg": "resource.google_sql_database_instance[master_instance].settings.ip_configuration.ipv4_enabled",
      "references": [
        "CIS Google Cloud Platform Foundation v1.1.0 - 6.6 Ensure that Cloud SQL database instances do not have public IPs",
        "https://cloud.google.com/sql/docs/mysql/configure-private-ip",
        "https://cloud.google.com/sql/docs/sqlserver/configure-ip"
      ],
      "path": [
        "resource",
        "google_sql_database_instance[master_instance]",
        "settings",
        "ip_configuration",
        "ipv4_enabled"
      ],
      "compliance": []
    },

    ---省略---  

  ]
}
```

* Snyk IaCはTerraformファイルだけでなく、Terraform planの出力に対しても解析をかけることができます。

Terraform planは実際のプロビジョニングをする前に立てる実行計画です。
よってTerraform planの出力にはプロビジョニング成功後の完全なる「望むべき形」が記載されています。
よって、`count`や`for_each`など静的には計算が難しいような記述も、Planの出力には正確な結果が記載されます。

また、ユーザー定義のTerraformファイルだけでなく、Terraform moduleのような外部から取り込まれるコードも含めて解析できます。

よって、より正確なスキャンを行うにはTerraformファイルではなく、Plan後の出力ファイルにたいして行うことを推奨しています。

より詳細についてはこちらを参照ください。
[Test your Terraform files with our CLI tool](https://docs.snyk.io/products/snyk-infrastructure-as-code/snyk-cli-for-infrastructure-as-code/test-your-terraform-files-with-the-cli-tool)

_**注：　Planを行うには実際のクラウドクレデンシャルなどが必要なため、このワークショップでは割愛しています。**_

## Step 5 - Snyk CLIでAWS CloudFormationをスキャンしてみる

Snyk IaCは、ソースコードレポジトリからCloudFormationファイルもスキャンすることができます。クラウド環境の安全性を高めるために、CloudFormationの設定ミスを本番環境に反映させる前に検出し、その修正方法についてアドバイスします。

* Snyk IaCは、"**CloudFormation**"ディレクトリ内のすべてのCloudFormationファイル（*.yml | *.yaml | *.json) をピックアップしスキャンします。

```bash
$ snyk iac test ./CloudFormation/

Snyk Infrastructure as Code

✔ Test completed.

Issues

Low Severity Issues: 15

  [Low] CloudWatch log group not encrypted with managed key
  Info:    Log group is not encrypted with customer managed key. Scope of use of
           the key cannot be controlled via KMS/IAM policies
  Rule:    https://snyk.io/security-rules/SNYK-CC-AWS-415
  Path:    [DocId: 0] > Resources[LogGroup] > Properties > KmsKeyId
  File:    fargate-service.yml
  Resolve: Set `Properties.KmsKeyId` attribute with customer managed key id

  [Low] ECR repository is not encrypted with customer managed key
  Info:    ECR repository is not encrypted with customer managed key. Scope of
           use of the key cannot be controlled via KMS/IAM policies
  Rule:    https://snyk.io/security-rules/SNYK-CC-AWS-418
  Path:    [DocId: 0] > Resources[EcrDockerRepository] > Properties >
           EncryptionConfiguration > KmsKey
  File:    fargate-service.yml
  Resolve: Set `Properties.EncryptionConfiguration.KmsKey` attribute to customer
           managed KMS key

  [Low] S3 bucket versioning disabled
  Info:    S3 bucket versioning is disabled. Changes or deletion of objects will
           not be reversible
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-124
  Path:    [DocId: 0] > Resources[CodePipelineArtifactBucket] > Properties >
           VersioningConfiguration > Status
  File:    fargate-service.yml
  Resolve: Set `Properties.VersioningConfiguration.Status` attribute to
           `Enabled`

  [Low] ECR Registry allows mutable tags
  Info:    The AWS ECR registry does not enforce immutable tags. Image tags can
           be modified post deployment
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-126
  Path:    [DocId: 0] > Resources[EcrDockerRepository] > Properties >
           ImageTagMutability
  File:    fargate-service.yml
  Resolve: Set `Properties.ImageTagMutability` attribute to `IMMUTABLE`

  [Low] ECR image scanning is disabled
  Info:    The ECR image scan for known vulnerabilities is disabled. The known
           vulnerabilities will not be automatically discovered
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-61
  Path:    [DocId: 0] > Resources[EcrDockerRepository] > Properties >
           ImageScanningConfiguration
  File:    fargate-service.yml
  Resolve: Set `Properties.ImageScanningConfiguration` attribute to `true`

  [Low] Rule allows open egress
  Info:    The security group rule allows open egress. Open egress can be used
           to exfiltrate data to unauthorized destinations, and enable access to
           potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-72
  Path:    Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[1]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] Rule allows open egress
  Info:    The security group rule allows open egress. Open egress can be used
           to exfiltrate data to unauthorized destinations, and enable access to
           potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-72
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[1]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] Rule allows open egress
  Info:    The security group rule allows open egress. Open egress can be used
           to exfiltrate data to unauthorized destinations, and enable access to
           potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-72
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[2]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] Rule allows open egress
  Info:    The security group rule allows open egress. Open egress can be used
           to exfiltrate data to unauthorized destinations, and enable access to
           potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-72
  Path:    Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[0]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] Rule allows open egress
  Info:    The security group rule allows open egress. Open egress can be used
           to exfiltrate data to unauthorized destinations, and enable access to
           potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-72
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[0]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] AWS Security Group allows open egress
  Info:    The inline security group rule allows open egress. Open egress can be
           used to exfiltrate data to unauthorized destinations, and enable
           access to potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-73
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[2]
           > CidrIp
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] AWS Security Group allows open egress
  Info:    The inline security group rule allows open egress. Open egress can be
           used to exfiltrate data to unauthorized destinations, and enable
           access to potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-73
  Path:    Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[1] >
           CidrIp
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] AWS Security Group allows open egress
  Info:    The inline security group rule allows open egress. Open egress can be
           used to exfiltrate data to unauthorized destinations, and enable
           access to potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-73
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[0]
           > CidrIp
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] AWS Security Group allows open egress
  Info:    The inline security group rule allows open egress. Open egress can be
           used to exfiltrate data to unauthorized destinations, and enable
           access to potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-73
  Path:    Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[0] >
           CidrIp
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

  [Low] AWS Security Group allows open egress
  Info:    The inline security group rule allows open egress. Open egress can be
           used to exfiltrate data to unauthorized destinations, and enable
           access to potentially malicious resources
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-73
  Path:    Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[1]
           > CidrIp
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupEgress.CidrIp` attribute to specific
           ranges e.g. `192.168.1.0/24`

Medium Severity Issues: 1

  [Medium] Security Group allows open ingress
  Info:    That inbound traffic is allowed to a resource from any source instead
           of a restricted range. That potentially everyone can access your
           resource
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-1
  Path:    Resources > ELBSecurityGroup > Properties > SecurityGroupIngress[0]
  File:    vpc.json
  Resolve: Set `Properties.SecurityGroupIngress.CidrIp` attribute with a more
           restrictive IP, for example `192.16.0.0/24`

High Severity Issues: 4

  [High] S3 block public ACLs control is disabled
  Info:    Bucket does not prevent creation of public ACLs. Anyone who can
           manage bucket's ACLs will be able to grant public access to the
           bucket
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-95
  Path:    [DocId: 0] > Resources[CodePipelineArtifactBucket] > Properties >
           PublicAccessBlockConfiguration > BlockPublicAcls
  File:    fargate-service.yml
  Resolve: Set `Properties.PublicAccessBlockConfiguration.BlockPublicAcls`
           attribute to `true`

  [High] S3 block public policy control is disabled
  Info:    Bucket does not prevent creation of public policies. Anyone who can
           manage bucket's policies will be able to grant public access to the
           bucket.
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-96
  Path:    [DocId: 0] > Resources[CodePipelineArtifactBucket] > Properties >
           PublicAccessBlockConfiguration > BlockPublicPolicy
  File:    fargate-service.yml
  Resolve: Set `Properties.PublicAccessBlockConfiguration.BlockPublicPolicy`
           attribute to `true`

  [High] S3 ignore public ACLs control is disabled
  Info:    Bucket will recognize public ACLs and allow public access. If public
           ACL is attached to the bucket, anyone will be able to read and/or
           write to the bucket.
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-97
  Path:    [DocId: 0] > Resources[CodePipelineArtifactBucket] > Properties >
           PublicAccessBlockConfiguration > IgnorePublicAcls
  File:    fargate-service.yml
  Resolve: Set `Properties.PublicAccessBlockConfiguration.IgnorePublicAcls`
           attribute to `true`

  [High] S3 restrict public bucket control is disabled
  Info:    Bucket will recognize public policies and allow public access. If
           public policy is attached to the bucket, anyone will be able to read
           and/or write to the bucket.
  Rule:    https://snyk.io/security-rules/SNYK-CC-TF-98
  Path:    [DocId: 0] > Resources[CodePipelineArtifactBucket] > Properties >
           PublicAccessBlockConfiguration > RestrictPublicBuckets
  File:    fargate-service.yml
  Resolve: Set `Properties.PublicAccessBlockConfiguration.RestrictPublicBuckets`
           attribute to `true`

-------------------------------------------------------

Test Summary

  Organization: masatomo.ito-qx0
  Project name: snyk-japan/snyk-iac-workshop

✔ Files without issues: 0
✗ Files with issues: 2
  Ignored issues: 0
  Total issues: 20 [ 0 critical, 4 high, 1 medium, 15 low ]

-------------------------------------------------------

Tip

  New: Share your test results in the Snyk Web UI with the option --report
```

時間があれば、ここで検出した設定ミスを修正して再度スキャンをかけてみてください。

* Snyk Iacは解析結果をJsonフォーマットでも出力できます。より詳細な情報や、見つかった設定ミスの参照へのリンクなどを表示します。他のレポート生成ツールなどとの連携に便利です。

```bash
$ snyk iac test ./CloudFormation --json
```

CloudFormationのスキャンについてより詳細な説明は以下のリンクより参照ください。
[Scan for AWS CloudFormation misconfigurations with Snyk IaC](https://snyk.io/blog/scan-aws-cloudformation-misconfigurations-snyk-iac/)

## Step 6 - Snyk CLIでKubernetesのマニフェストファイルをスキャンしてみる

Snyk IaCは、Kubernetesの設定ファイルもスキャンすることができます。Kubernetes環境のセキュリティを向上させるための情報やヒントなどを本番環境に適用される前に検出し、脆弱性に対する修正を提供します。

Snyk IaCは以下をスキャンできます。

1. Deployments, Pods and Services.
1. CronJobs, Jobs, StatefulSet, ReplicaSet, DaemonSet, and ReplicationController
1. Helm Charts

* それではKubernetsのYamlファイルをスキャンしてみましょう。

```bash
$ snyk iac test ./Kubernetes/

Snyk Infrastructure as Code

✔ Test completed.

Issues

Low Severity Issues: 4

  [Low] Container is running without memory limit
  Info:    Memory limit is not defined. Containers without memory limits are
           more likely to be terminated when the node runs out of memory
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-4
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > resources > limits > memory
  File:    employee-K8s.yaml
  Resolve: Set `resources.limits.memory` value

  [Low] Container is running without liveness probe
  Info:    Liveness probe is not defined. Kubernetes will not be able to detect
           if application is able to service requests, and will not restart
           unhealthy pods
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-41
  Path:    [DocId: 0] > spec > template > spec > containers[snyk-employee-api] >
           livenessProbe
  File:    employee-K8s.yaml
  Resolve: Add `livenessProbe` attribute

  [Low] Container has no CPU limit
  Info:    Container has no CPU limit. CPU limits can prevent containers from
           consuming valuable compute time for no benefit (e.g. inefficient
           code) that might lead to unnecessary costs. It is advisable to also
           configure CPU requests to ensure application stability.
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-5
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > resources > limits > cpu
  File:    employee-K8s.yaml
  Resolve: Add `resources.limits.cpu` field with required CPU limit value

  [Low] Container is running with writable root filesystem
  Info:    `readOnlyRootFilesystem` attribute is not set to `true`. Compromised
           process could abuse writable root filesystem to elevate privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-8
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext >
           readOnlyRootFilesystem
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.readOnlyRootFilesystem` to `true`

Medium Severity Issues: 4

  [Medium] Container is running without root user control
  Info:    Container is running without root user control. Container could be
           running with full administrative privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-10
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext > runAsNonRoot
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.runAsNonRoot` to `true`

  [Medium] Service does not restrict ingress sources
  Info:    Defining a Load balancer Service without setting the
           loadBalancerSourceRanges property will use the default value of
           0.0.0.0/0. This allows access to any traffic to the Node Security
           Group(s), potentially meaning everyone can access your service.
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-15
  Path:    [DocId: 1] > service > spec > loadBalancerSourceRanges
  File:    employee-K8s.yaml
  Resolve: Set `loadBalancerSourceRanges` attribute value to specific IP
           addresses

  [Medium] Container does not drop all default capabilities
  Info:    All default capabilities are not explicitly dropped. Containers are
           running with potentially unnecessary privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-6
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext > capabilities > drop
  File:    employee-K8s.yaml
  Resolve: Add `ALL` to `securityContext.capabilities.drop` list, and add only
           required capabilities in `securityContext.capabilities.add`

  [Medium] Container is running without privilege escalation control
  Info:    `allowPrivilegeEscalation` attribute is not set to `false`. Processes
           could elevate current privileges via known vectors, for example SUID
           binaries
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-9
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext >
           allowPrivilegeEscalation
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.allowPrivilegeEscalation` to `false`

-------------------------------------------------------

Test Summary

  Organization: masatomo.ito-qx0
  Project name: snyk-japan/snyk-iac-workshop

✔ Files without issues: 0
✗ Files with issues: 1
  Ignored issues: 0
  Total issues: 8 [ 0 critical, 0 high, 4 medium, 4 low ]

-------------------------------------------------------

Tip

  New: Share your test results in the Snyk Web UI with the option --report
```

* ここで一つの危険な設定を修正してみましょう。

```bash
  [Medium] Container is running without root user control
  Info:    Container is running without root user control. Container could be
           running with full administrative privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-10
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext > runAsNonRoot
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.runAsNonRoot` to `true`
```

この設定では、コンテナがRoot権限で実行される可能性を示唆しています。
"**./Kubernetes/employee-K8s.yaml**"の"**securityContext > runAsNonRoot**"を`true`に設定して修正してみましょう。

```yaml
    spec:
      securityContext:
        runAsNonRoot: true
```

再度スキャンして修正が適用されたか確認してみます。

  
```bash
$ snyk iac test ./Kubernetes/

Snyk Infrastructure as Code

✔ Test completed.

Issues

Low Severity Issues: 4

  [Low] Container is running without memory limit
  Info:    Memory limit is not defined. Containers without memory limits are
           more likely to be terminated when the node runs out of memory
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-4
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > resources > limits > memory
  File:    employee-K8s.yaml
  Resolve: Set `resources.limits.memory` value

  [Low] Container is running without liveness probe
  Info:    Liveness probe is not defined. Kubernetes will not be able to detect
           if application is able to service requests, and will not restart
           unhealthy pods
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-41
  Path:    [DocId: 0] > spec > template > spec > containers[snyk-employee-api] >
           livenessProbe
  File:    employee-K8s.yaml
  Resolve: Add `livenessProbe` attribute

  [Low] Container has no CPU limit
  Info:    Container has no CPU limit. CPU limits can prevent containers from
           consuming valuable compute time for no benefit (e.g. inefficient
           code) that might lead to unnecessary costs. It is advisable to also
           configure CPU requests to ensure application stability.
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-5
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > resources > limits > cpu
  File:    employee-K8s.yaml
  Resolve: Add `resources.limits.cpu` field with required CPU limit value

  [Low] Container is running with writable root filesystem
  Info:    `readOnlyRootFilesystem` attribute is not set to `true`. Compromised
           process could abuse writable root filesystem to elevate privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-8
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext >
           readOnlyRootFilesystem
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.readOnlyRootFilesystem` to `true`

Medium Severity Issues: 3

  [Medium] Service does not restrict ingress sources
  Info:    Defining a Load balancer Service without setting the
           loadBalancerSourceRanges property will use the default value of
           0.0.0.0/0. This allows access to any traffic to the Node Security
           Group(s), potentially meaning everyone can access your service.
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-15
  Path:    [DocId: 1] > service > spec > loadBalancerSourceRanges
  File:    employee-K8s.yaml
  Resolve: Set `loadBalancerSourceRanges` attribute value to specific IP
           addresses

  [Medium] Container does not drop all default capabilities
  Info:    All default capabilities are not explicitly dropped. Containers are
           running with potentially unnecessary privileges
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-6
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext > capabilities > drop
  File:    employee-K8s.yaml
  Resolve: Add `ALL` to `securityContext.capabilities.drop` list, and add only
           required capabilities in `securityContext.capabilities.add`

  [Medium] Container is running without privilege escalation control
  Info:    `allowPrivilegeEscalation` attribute is not set to `false`. Processes
           could elevate current privileges via known vectors, for example SUID
           binaries
  Rule:    https://snyk.io/security-rules/SNYK-CC-K8S-9
  Path:    [DocId: 0] > input > spec > template > spec >
           containers[snyk-employee-api] > securityContext >
           allowPrivilegeEscalation
  File:    employee-K8s.yaml
  Resolve: Set `securityContext.allowPrivilegeEscalation` to `false`

-------------------------------------------------------

Test Summary

  Organization: masatomo.ito-qx0
  Project name: snyk-japan/snyk-iac-workshop

✔ Files without issues: 0
✗ Files with issues: 1
  Ignored issues: 0
  Total issues: 7 [ 0 critical, 0 high, 3 medium, 4 low ]

-------------------------------------------------------

Tip

  New: Share your test results in the Snyk Web UI with the option --report
```


## Step 7 - Snyk IaCの検知するルールを確認してみよう

Snyk IaCは、Terraform、CloudFormation、Kubernetes、HelmのIaCを解析することができます。また、AWS、Azure、GCP、Kubernetesにわたる包括的なセキュリティルールのセットを適用します。各プロバイダーのベストプラクティスやCISなどのベンチマークなどを揃えています。

こちらのリンクより、それらのルールセットを確認ください。
[Snyk Infrastructure as Code](https://snyk.io/security-rules)

以上でSnyk IaCのワークショップは終わりになります。
ありがとうございました。

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
