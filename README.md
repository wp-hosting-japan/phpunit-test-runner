# phpunit-test-runner

<!--
Thanks for running the WordPress PHPUnit test suite on your infrastructure. We appreciate you helping to ensure WordPress’ compatibility for your users.
-->

WordPress の PHPUnit テストをみなさんのインフラで実行していただいてありがとうございます！ 私たちは、ユーザーのために WordPress との互換性を維持することを手助けしてくださることに感謝しています。

<!--
If you haven't already, [please first read through the "Getting Started" documentation](https://make.wordpress.org/hosting/test-results-getting-started/).
-->

まだ、実行していない場合は、[まずはじめにこちらのドキュメントをお読みください](https://make.wordpress.org/hosting/test-results-getting-started/)。（[日本語ドキュメンテーション](https://github.com/wp-hosting-japan/phpunit-test-runner/blob/master/docs-ja.md)）

<!--
The test suite runner is designed to be used without any file modification. Configuration happens with a series of environment variables (see [.env.default](.env.default) for an annotated overview). Use the [repository wiki](../../wiki) to document implementation details, to avoid README conflicts with the upstream.
-->

テストランナーは、ファイルの編集等を必要とすること無く利用できるように設計されています。初期設定はいくつかの環境変数で構成されています。（[.env.default](.env.default) をご覧ください。）

<!--
At a high level, the test suite runner:
-->

詳しく説明すると:

<!--
1. Prepares the test environment for the test suite.
2. Runs the PHPUnit tests in the test environment.
3. Reports the PHPUnit test results to WordPress.org
4. Cleans up the test suite environment.
-->

1. テストスイート用の環境を準備する。
2. テスト環境で PHPUnit を実行する。
3. WordPress.org にテスト結果を報告する。
4. テスト環境を初期化する。

<!--
## Configuring
-->

## 設定

<!--
The test suite runner can be used in one of two ways:
-->

テストスイート用ランナーは以下の2つの方法のうちのいずれかで実行することができます。

<!--
1. With Travis (or Circle or some other CI service) as the controller that connects to the remote test environment.
2. With the runner cloned to and run directly within the test environment.
-->

1. Travis CI （または Circle CI などの CI サービス）をコントローラーとして利用して、リモートサーバー上のテストを実行する。
2. テストランナーをテスト環境内にクローンして直接実行する。

<!--
The test runner is configured through environment variables, documented in [`.env.default`](.env.default). It shouldn't need any code modifications; in fact, please refrain from editing the scripts entirely, as it will make it easier to stay up to date.
-->

テストランナーは[`.env.default`](.env.default)に記述されている環境変数によって、設定を変更することが出来ます。コードを変更する必要はありません。最新の状態に保つのを簡単にするために、スクリプトを編集することは控えて下さい。

<!--
With a direct Git clone, you can:
-->

直接 Git クローンして、次のように設定出来ます。

<!--
    # Copy the default .env file.
    cp .env.default .env
    # Edit the .env file to define your variables.
    vim .env
    # Load your variables into scope.
    source .env
-->

    # デフォルトの .env ファイルをコピー.
    cp .env.default .env
    # .env ファイルを編集して、変数を設定。
    vim .env
    # 設定した変数を現在のスコープにロード。
    source .env

<!--
In a CI service, you can set these environment variables through the service's web console. Importantly, the `WPT_SSH_CONNECT` environment variable determines whether the test suite is run locally or against a remote environment.
-->

CI サービスでは、WEB コンソールでこれらの環境変数を設定出来ます。重要： 環境変数 `WPT_SSH_CONNECT` は、テストスイートがローカルで実行されるのかリモートで実行されるのかを決定します。

<!--
Concurrently run tests in the same environment by appending build ids to the test directory and table prefix:
-->

ビルドIDとテーブル接頭辞を追加すると、同じ環境でテストを実行:

<!--
    export WPT_TEST_DIR=wp-test-runner-$TRAVIS_BUILD_NUMBER
    export WPT_TABLE_PREFIX=wptests_$TRAVIS_BUILD_NUMBER\_
-->

    export WPT_TEST_DIR=wp-test-runner-$TRAVIS_BUILD_NUMBER
    export WPT_TABLE_PREFIX=wptests_$TRAVIS_BUILD_NUMBER\_

<!--
Connect to a remote environment over SSH by having the CI job provision the SSH key:
-->

CIジョブでSSH鍵を用意して、SSH経由でリモート環境に接続:

<!--
    # 1. Create a SSH key pair for the controller to use
    ssh-keygen -t rsa -b 4096 -C "travis@travis-ci.org"
    # 2. base64 encode the private key for use with the environment variable
    cat ~/.ssh/id_rsa | base64 --wrap=0
    # 3. Append id_rsa.pub to authorized_keys so the CI service can SSH in
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
-->

    # 1. CIサービスが使うための、SSHキーペアを作成
    ssh-keygen -t rsa -b 4096 -C "travis@travis-ci.org"
    # 2. 秘密鍵を環境変数として使うために、base64でエンコード。
    cat ~/.ssh/id_rsa | base64 --wrap=0
    # 3. CI サービスが SSH で接続できるように、id_rsa.pub を authorized_keys に追加
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

<!--
Use a more complex SSH connection process by creating a SSH alias:
-->

SSH エイリアスを利用して、複雑なよりSSH接続を行う:

<!--
    # 1. Add the following to ~/.ssh/config to create a 'wpt' alias
    Host wpt
      Hostname 123.45.67.89
      User wpt
      Port 1234
    # 2. Use 'wpt' wherever you might normally use a SSH connection string
    ssh wpt
-->

    # 1.  ~/.ssh/config に以下を追加して、'wpt' エイリアスを 作成
    Host wpt
      Hostname 123.45.67.89
      User wpt
      Port 1234
    # 2. SSH接続をする場合は、'wpt' を使用して下さい。
    ssh wpt


<!--
## Running
-->

## 実行

<!--
The test suite runner is run in four steps.
-->

テストスイートは4つのステップで実行されます。

<!--
### 0. Requirements
-->

### 0. 要件

<!--
Both the prep and test environments must meet some basic requirements.
-->

テストを準備する環境とテスト環境はどちらも基本的な要件を満たしていなければなりません。

<!--
Prep environment:
-->

準備環境:

<!--
* PHP 5.6 or greater (to run scripts).
* Utilities: `git`, `rsync`, `wget`, `unzip`.
-->

* PHP 5.6 以上 (スクリプト実行のため).
* ユーティリティー: `git`, `rsync`, `wget`, `unzip`.

<!--
Test environment:
-->

テスト環境:

<!--
* PHP 5.6 or greater with Phar support enabled (for PHPUnit).
* MySQL or MariaDB with access to a writable database.
* Writable filesystem for the entire test directory (see [#40910](https://core.trac.wordpress.org/ticket/40910) for details on why).
-->

* PHP 5.6 以上、 Phar のサポート (PHPUnitのため).
* 書き込み可能なデータベースにアクセスできる MySQL または MariaDB。
* テストディレクトリ全体の書き込み可能なファイルシステム (詳細については [#40910](https://core.trac.wordpress.org/ticket/40910) を参照して下さい).


### 1. Prepare

The [`prepare.php`](prepare.php) step:

1. Extracts the base64-encoded SSH private key, if necessary.
2. Clones the master branch of the WordPress develop git repo into the preparation directory.
3. Downloads `phpunit.phar` to the preparation directory.
4. Generates `wp-tests-config.php` and puts it into the preparation directory.
5. Delivers the prepared test directory to the test environment.

### 2. Test

The [`test.php`](test.php) step:

1. Calls `php phpunit.phar` to produce `tests/phpunit/build/logs/junit.xml`.

<!--
### 3. Report
-->
### 3. 報告

<!--
The [`report.php`](report.php) step:
-->
[`report.php`](report.php) の手順:

<!--
1. Processes PHPUnit XML log into a JSON blob.
2. Sends the JSON to WordPress.org.
-->
1. PHPUnit の XML を処理して、JSON の blob にする。
2. JSON を WordPress.org に送信する。

<!--
### 4. Cleanup
-->
### 4. 初期化

<!--
The [`cleanup.php`](cleanup.php) step:
-->
[`cleanup.php`](cleanup.php) の手順:

<!--
1. Resets the database.
2. Deletes all files delivered to the test environment.
-->
1. データベースをリセットする。
2. テスト環境に送信したすべてのファイルを削除する。

<!--
## Contributing
-->
## 貢献者

tk

<!--
## License
-->
## ライセンス

<!--
See [LICENSE](LICENSE) for project license.
-->
プロジェクトのライセンスは [LICENSE](LICENSE) (英文)を参照してください。
