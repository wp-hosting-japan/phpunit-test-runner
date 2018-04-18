## ホストテストを開始する
あなたのサーバ環境でPHPUnitをテストしていただけませんか？  
[テスト結果](https://make.wordpress.org/hosting/test-results/)が届くのを楽しみにしています。

テストの進め方に関して質問がある場合や、途中でテストに失敗した場合は、[プロジェクトのリポジトリでissue探してみてください。](https://github.com/wordpress/phpunit-test-runner/issues)。and we’ll help diagnose/get the documentation updated. Alternatively, また、[Wordpress.org Slack](https://make.wordpress.org/chat/)の`#hosting-community`チャネルに参加して解決の糸口を見つけることもできます。

## セットアップ
セットアップには2つの主要なステップがあります。    

一つ目は、あなたのサーバ環境で[phpunit-test-runnerが実行できるように設定することです](https://github.com/wordpress/phpunit-test-runner)。詳しくは、[GithubのREADME](https://github.com/WordPress/phpunit-test-runner/blob/master/README.md)を読んでください。特に:

* テストはTravis CIまたは、サーバのcronジョブから実行可能です。
    * 最終的には全てのコミットに対してテストを実行させたいと思っていますが、現時点では、３〜6時間おきに実行するだけで十分です。
* テスト用に様々な環境変数があります。

次に、全てのテストに合格した場合、結果をWordPress.orgへ送ることができます。そのために必要なことは、

1. WordPress.orgのbotアカウントを作成します。あなたの会社名が"Wonderful Hosting"の場合、botアカウント名はおそらく`wonderfulbot`となるでしょう。テストが失敗した際、メールを送信するので、メールアドレスを設定してください。
2. phpunit-test-runner Githubリポジトリ上でWordPress.orgサイトにbotユーザーを"Test Reporter"として追加するようにissueを新規作成します。  
追加するにはユーザーに紐付いたメールアドレスが必要です。  
テスト結果表示にはカスタム登校タイプを使用しています。
3. botユーザーがWordPress.orgサイトに追加されると、テストランナーのアプリケーションパスワードを作成することができます。  
WordPress.orgサイトにログインし、Users > Your Profileに移動し、アプリケーションパスワードを生成します。  
次に、生成したアプリケーションパスワードを環境変数として設定します。  
`export WPT_REPORT_API_KEY='wonderfulbot:Osho NHgM xYSY UWF9 qNUn YdjV'`  
ユーザー名後のコロンの後にある文字を、botユーザーのアプリケーションパスワードに置き換えます。

全てが上手くいくと、テスト結果は`php report.php`を実行後、最後に表示されます。  

<img data-attachment-id="244" data-permalink="https://make.wordpress.org/hosting/test-results-getting-started/2017-08-24-at-2-25-pm/#main" data-orig-file="https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?fit=913%2C279&amp;ssl=1" data-orig-size="913,279" data-comments-opened="1" data-image-meta="{&quot;aperture&quot;:&quot;0&quot;,&quot;credit&quot;:&quot;&quot;,&quot;camera&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;created_timestamp&quot;:&quot;0&quot;,&quot;copyright&quot;:&quot;&quot;,&quot;focal_length&quot;:&quot;0&quot;,&quot;iso&quot;:&quot;0&quot;,&quot;shutter_speed&quot;:&quot;0&quot;,&quot;title&quot;:&quot;&quot;,&quot;orientation&quot;:&quot;0&quot;}" data-image-title="2017-08-24 at 2.25 PM" data-image-description="" data-medium-file="https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?fit=300%2C92&amp;ssl=1" data-large-file="https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?fit=776%2C237&amp;ssl=1" class="alignnone size-full wp-image-244" src="https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?resize=776%2C237&amp;ssl=1" alt="" width="647" height="198" srcset="https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?w=913&amp;ssl=1 913w, https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?resize=300%2C92&amp;ssl=1 300w, https://i0.wp.com/make.wordpress.org/hosting/files/2017/08/2017-08-24-at-2.25-PM.png?resize=768%2C235&amp;ssl=1 768w" sizes="(max-width: 776px) 100vw, 776px">      
<br>
<br>
最後の行は結果が正常にウェブサイトへアップロードされたことを示しています。