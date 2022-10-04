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
  - [Step 5 Test using the Snyk CLI - AWS CloudFormation files](#step-5-test-using-the-snyk-cli---aws-cloudformation-files)
  - [Step 6 Test using the Snyk CLI - Kubernetes YAML files](#step-6-test-using-the-snyk-cli---kubernetes-yaml-files)
  - [Step 7 View Snyk IaC Rules](#step-7-view-snyk-iac-rules)

## 事前準備 

* GitHubアカウント（パブリックであること） - http://github.com
* git CLI - https://git-scm.com/downloads
* snyk CLI - https://support.snyk.io/hc/en-us/articles/360003812538-Install-the-Snyk-CLI
* Snykアカウント - http://app.snyk.io

# Workshop Steps

_注: この手順はMacを想定していますが、WindowsやLinuxでもスクリプトを一部修正することで対応できます。_

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

![alt tag](https://i.ibb.co/bPqqybM/snyk-starter-open-source-1.png)

## Step 3 脆弱性を発見するプロジェクトの追加

* このステップを行う前にIaCスキャンが有効になっているか確認ください。"**Settings → Infrastructure as code**"から確認および有効化ができます。

![alt tag](https://i.ibb.co/bRLppWW/snyk-iac-7.png)

GitHubとの連携ができたので、次はGitHubレポジトリをSnykのプロジェクトとしてインポートします。

* Snyk UI上でProjectsをクリックします
* "**Add Project**" → "**GitHub**"
* Forkした"**snyk-iac-workshop**"を選択します

![alt tag](https://i.ibb.co/pWJW1VK/snyk-iac-1.png)

__注: インポートとスキャンが成功すると以下のような画面が表示されます。_

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

_注: インストールしたSnyk CLIのバージョンが以下のバージョンと同じかそれ以上か確認してください。Snyk CLIのインストールおよびアップグレードはこちらになります。https://docs.snyk.io/snyk-cli/install-the-snyk-cli_

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

_注： このレポジトリではなく、皆様がForkしたGitレポジトリをCloneしてください。_

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

_注：　Planを行うには実際のクラウドクレデンシャルなどが必要なため、このワークショップでは割愛しています。_

## Step 5 Test using the Snyk CLI - AWS CloudFormation files

_Note: Please ensure you have the latest version of the Snyk CLI installed a version equal to or greater than the version below. https://docs.snyk.io/features/snyk-cli/install-the-snyk-cli_

```bash
$ snyk --version
1.675.0
```

Snyk tests and monitors CloudFormation files from source code repositories. It gives advice on how to better secure cloud environments by catching misconfigurations before they are pushed to production along with assistance on how best to fix them

* Run the following IaC scan of any files inside the folder "**CloudFormation**". This will pick up all the IaC config files which exist in this directory in our case we have two files

```bash
$ snyk iac test ./CloudFormation/

Testing fargate-service.yml...


Infrastructure as code issues:
  ✗ S3 block public policy control is disabled [High Severity] [SNYK-CC-TF-96] in S3
    introduced by Resources[CodePipelineArtifactBucket] > Properties > PublicAccessBlockConfiguration > BlockPublicPolicy

  ✗ S3 ignore public ACLs control is disabled [High Severity] [SNYK-CC-TF-97] in S3
    introduced by Resources[CodePipelineArtifactBucket] > Properties > PublicAccessBlockConfiguration > IgnorePublicAcls

  ✗ S3 block public ACLs control is disabled [High Severity] [SNYK-CC-TF-95] in S3
    introduced by Resources[CodePipelineArtifactBucket] > Properties > PublicAccessBlockConfiguration > BlockPublicAcls

  ✗ S3 restrict public bucket control is disabled [High Severity] [SNYK-CC-TF-98] in S3
    introduced by Resources[CodePipelineArtifactBucket] > Properties > PublicAccessBlockConfiguration > RestrictPublicBuckets

  ✗ ECR image scanning is disabled [Low Severity] [SNYK-CC-TF-61] in ECR
    introduced by Resources[EcrDockerRepository] > Properties > ImageScanningConfiguration

  ✗ S3 bucket versioning disabled [Low Severity] [SNYK-CC-TF-124] in S3
    introduced by Resources[CodePipelineArtifactBucket] > Properties > VersioningConfiguration > Status

  ✗ CloudWatch log group not encrypted with managed key [Low Severity] [SNYK-CC-AWS-415] in CloudWatch
    introduced by Resources[LogGroup] > Properties > KmsKeyId

  ✗ ECR repository is not encrypted with customer managed key [Low Severity] [SNYK-CC-AWS-418] in ECR
    introduced by Resources[EcrDockerRepository] > Properties > EncryptionConfiguration > KmsKey

  ✗ ECR Registry allows mutable tags [Low Severity] [SNYK-CC-TF-126] in ECR
    introduced by Resources[EcrDockerRepository] > Properties > ImageTagMutability


Organization:      pas.apicella-41p
Type:              CloudFormation
Target file:       fargate-service.yml
Project name:      CloudFormation
Open source:       no
Project path:      ./CloudFormation/

Tested fargate-service.yml for known issues, found 9 issues

-------------------------------------------------------

Testing vpc.json...


Infrastructure as code issues:
  ✗ Security Group allows open ingress [Medium Severity] [SNYK-CC-TF-1] in VPC
    introduced by Resources > ELBSecurityGroup > Properties > SecurityGroupIngress[0]

  ✗ AWS Security Group allows open egress [Low Severity] [SNYK-CC-TF-73] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[1] > CidrIp

  ✗ Rule allows open egress [Low Severity] [SNYK-CC-TF-72] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[1]

  ✗ AWS Security Group allows open egress [Low Severity] [SNYK-CC-TF-73] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[2] > CidrIp

  ✗ Rule allows open egress [Low Severity] [SNYK-CC-TF-72] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[2]

  ✗ AWS Security Group allows open egress [Low Severity] [SNYK-CC-TF-73] in VPC
    introduced by Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[1] > CidrIp

  ✗ Rule allows open egress [Low Severity] [SNYK-CC-TF-72] in VPC
    introduced by Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[0]

  ✗ AWS Security Group allows open egress [Low Severity] [SNYK-CC-TF-73] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[0] > CidrIp

  ✗ Rule allows open egress [Low Severity] [SNYK-CC-TF-72] in VPC
    introduced by Resources[BastionSecurityGroup] > Properties > SecurityGroupEgress[0]

  ✗ Rule allows open egress [Low Severity] [SNYK-CC-TF-72] in VPC
    introduced by Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[1]

  ✗ AWS Security Group allows open egress [Low Severity] [SNYK-CC-TF-73] in VPC
    introduced by Resources[DbSecurityGroup] > Properties > SecurityGroupEgress[0] > CidrIp


Organization:      pas.apicella-41p
Type:              CloudFormation
Target file:       vpc.json
Project name:      CloudFormation
Open source:       no
Project path:      ./CloudFormation/

Tested vpc.json for known issues, found 11 issues


Tested 2 projects, 2 contained issues.
```

Go ahead and fix others if you have time and optionally commit your changes back to the GitHub repo if you like.

* To output the test format as JSON issue a command as follows. This provides more detailed information including links to issue references as well as the ability to upload the data into other system for reporting purposes.

```bash
$ snyk iac test ./CloudFormation --json
```

For more information on AWS Cloud Formation scanning with Snyk see the following blog post
[Scan for AWS CloudFormation misconfigurations with Snyk IaC](https://snyk.io/blog/scan-aws-cloudformation-misconfigurations-snyk-iac/)

## Step 6 Test using the Snyk CLI - Kubernetes YAML files

Snyk tests and monitors Kubernetes configurations stored in your source code repositories and provides information, tips, and tricks to better secure a Kubernetes environment--catching misconfigurations before they are pushed to production as well as providing fixes for vulnerabilities

Snyk Infrastructure as Code for Kubernetes supports:

1. Deployments, Pods and Services.
1. CronJobs, Jobs, StatefulSet, ReplicaSet, DaemonSet, and ReplicationController
1. Helm Charts

* Let's scan our Kubernetes YAML file which is just a basic K8s deployment as shown below

```bash
$ snyk iac test ./Kubernetes/employee-K8s.yaml

Testing employee-K8s.yaml...


Infrastructure as code issues:
  ✗ Container is running without privilege escalation control [Medium Severity] [SNYK-CC-K8S-9] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > allowPrivilegeEscalation

  ✗ Container is running without root user control [Medium Severity] [SNYK-CC-K8S-10] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > runAsNonRoot

  ✗ Container does not drop all default capabilities [Medium Severity] [SNYK-CC-K8S-6] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > capabilities > drop

  ✗ Service does not restrict ingress sources [Medium Severity] [SNYK-CC-K8S-15] in Service
    introduced by service > spec > loadBalancerSourceRanges

  ✗ Container is running without liveness probe [Low Severity] [SNYK-CC-K8S-41] in Deployment
    introduced by spec > template > spec > containers[snyk-employee-api] > livenessProbe

  ✗ Container is running with writable root filesystem [Low Severity] [SNYK-CC-K8S-8] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > readOnlyRootFilesystem

  ✗ Container is running without AppArmor profile [Low Severity] [SNYK-CC-K8S-32] in Deployment
    introduced by metadata > annotations['container.apparmor.security.beta.kubernetes.io/snyk-employee-api']

  ✗ Container is running without memory limit [Low Severity] [SNYK-CC-K8S-4] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > resources > limits > memory

  ✗ Container is running without cpu limit [Low Severity] [SNYK-CC-K8S-5] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > resources > limits > cpu


Organization:      pas.apicella-41p
Type:              Kubernetes
Target file:       ./Kubernetes/employee-K8s.yaml
Project name:      Kubernetes
Open source:       no
Project path:      ./Kubernetes/employee-K8s.yaml

Tested employee-K8s.yaml for known issues, found 9 issues
```

* Let's go ahead and fix the following

```bash
  ✗ Container is running without root user control [Medium Severity] [SNYK-CC-K8S-10] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > runAsNonRoot
```

* Edit the file "**./Kubernetes/employee-K8s.yaml**" and add "**securityContext > runAsNonRoot**" as shown below.

```yaml
    spec:
      securityContext:
        runAsNonRoot: true
      containers:
        - name: snyk-employee-api
          image: pasapples/springbootemployee:jib
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
```

* Run a scan of "**./Kubernetes/employee-K8s.yaml**" again to verify you have fixed that issue
  
```bash
$ snyk iac test ./Kubernetes/employee-K8s.yaml

Testing employee-K8s.yaml...


Infrastructure as code issues:
  ✗ Container is running without privilege escalation control [Medium Severity] [SNYK-CC-K8S-9] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > allowPrivilegeEscalation

  ✗ Container does not drop all default capabilities [Medium Severity] [SNYK-CC-K8S-6] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > capabilities > drop

  ✗ Service does not restrict ingress sources [Medium Severity] [SNYK-CC-K8S-15] in Service
    introduced by service > spec > loadBalancerSourceRanges

  ✗ Container is running without liveness probe [Low Severity] [SNYK-CC-K8S-41] in Deployment
    introduced by spec > template > spec > containers[snyk-employee-api] > livenessProbe

  ✗ Container is running with writable root filesystem [Low Severity] [SNYK-CC-K8S-8] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > securityContext > readOnlyRootFilesystem

  ✗ Container is running without AppArmor profile [Low Severity] [SNYK-CC-K8S-32] in Deployment
    introduced by metadata > annotations['container.apparmor.security.beta.kubernetes.io/snyk-employee-api']

  ✗ Container is running without memory limit [Low Severity] [SNYK-CC-K8S-4] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > resources > limits > memory

  ✗ Container is running without cpu limit [Low Severity] [SNYK-CC-K8S-5] in Deployment
    introduced by input > spec > template > spec > containers[snyk-employee-api] > resources > limits > cpu


Organization:      pas.apicella-41p
Type:              Kubernetes
Target file:       ./Kubernetes/employee-K8s.yaml
Project name:      Kubernetes
Open source:       no
Project path:      ./Kubernetes/employee-K8s.yaml

Tested employee-K8s.yaml for known issues, found 8 issues
```

Go ahead and fix others if you have time and optionally commit your changes back to the GitHub repo if you like.

* To output the test format as JSON issue a command as follows. This provides more detailed information including links to issue references as well as the ability to upload the data into other system for reporting purposes.

```bash
$ snyk iac test ./Kubernetes/employee-K8s.yaml --json
```

## Step 7 View Snyk IaC Rules

Snyk IaC has a comprehensive set of security rules across AWS, Azure, GCP & Kubernetes with support for Terraform, CloudFormation, Kubernetes, and Helm configuration formats. The details of these issues, their impact, and how to fix them are all built-in to Snyk IaC, so developers get feedback directly in their own tools. For reference, we have also documented the security rules that we support for each provider below, along with relevant benchmarks and authoritative third-party references

Navigate to [Snyk Infrastructure as Code](https://snyk.io/security-rules)

Thanks for attending and completing this workshop

![alt tag](https://i.ibb.co/7tnp1B6/snyk-logo.png)

<hr />
Pas Apicella [pas at snyk.io] is an Solution Engineer at Snyk APJ
