---
タイトル：認証の基礎知識
---

# 認証の基礎知識

{:toc}

このセクションでは、認証や粉ミルクの基礎に集中するつもりです。具体的には、
我々は、使用して（Rubyのサーバーを作成しようとしています[シナトラ][Sinatra]）その実装
インクルード[ウェブの流れ][webflow]いくつかの異なる方法でアプリケーションの。

{{#tip}}

あなたはこのプロジェクトのための完全なソースコードをダウンロードすることができます[プラットフォームのサンプルのレポから](https://github.com/github/platform-samples/tree/master/api/).

{{/tip}}

## あなたのアプリを登録します

まず、あなたがする必要があります[アプリケーションを登録][new oauth app]。すべての
登録のOAuthアプリケーションは、一意のクライアントIDとクライアントシークレットを割り当てられています。
クライアントの秘密は、共有すべきではありません！これは、文字列をチェックすることが含ま
リポジトリへ。

あなたは除いて、しかし、あなたが好きなあらゆる情報を記入することができます
**認証コールバックURL**。これは、簡単に設定することに最も重要な部分であります
アプリケーションのアップ。これは、後にユーザーを返すコールバックURLです {{ site.data.variables.product.product_name }}
認証に成功します。

我々は、通常のシナトラサーバー、ローカルインスタンスの場所を実行しているので、
に設定されています`Http：// localhostを：4567`。のは、コールバックURLなどを記入してみましょう`Http：// localhostを：4567 /コールバック`.

## 受け入れユーザー認証

今、私たちの単純なサーバーを記入し始めましょう。ファイルを作成します。_server.rb_そしてそれにこれを貼り付けます。

``` ruby
インデックス、：：地元の人々=> '末端シナトラ' 必要 '残りのクライアント' 'jsonの' CLIENT_ID = ENV CLIENT_SECRET ['GH_BASIC_CLIENT_ID'] = ENV取得 '/' を実行ERBを必要とする必要が ['GH_BASIC_SECRET_ID'] {:client_id => CLIENT_ID}```

あなたのクライアントIDとクライアント秘密鍵から来ます[アプリケーションの設定
ページ][app settings]。あなたがすべき**決して、_今までに_**これらの値を保存します
{{ site.data.variables.product.product_name }} そのことについては、他の公共の場所を、-OR。私たちは、それらを保存するお勧めします
[環境変数][about env vars]--which我々はここでやったまさにです。

次に、中_views/index.erb_このコンテンツを貼り付けます。

``` erb
<html> <head> </head> <body> <p> まあ、こんにちは！現在のGitHub APIに話をするつもりです。準備はできましたか？ 「>開始するにはこちらをクリック！そのリンクが動作しない場合は、独自のクライアントIDを提供することを忘れないでください！ </p> </p> </a> </p> </a> </body> </html> <p> <p> <a href="https://github.com/login/oauth/authorize?scope=user:email&client_id=<%= client_id %> <a href="/v3/oauth/#web-application-flow">```

あなたはシナトラの仕組みに慣れていないなら（、私たちはお勧めします[シナトラのガイドを読みます][Sinatra guide].)

また、URLを使用していることに気づきます`範囲`定義するためのクエリパラメータ
[スコープ][oauth scopes]アプリケーションによって要求されました。私たちのアプリケーションのために、私たちはしています
要求`ユーザー：電子メール`プライベートのメールアドレスを読み取るためのスコープ。

にブラウザをナビゲート`Http：// localhostを：4567`。リンクをクリックした後、あなた
とって、このようになりますダイアログを提示する必要があります。 {{ site.data.variables.product.product_name }}
![GitHubののOAuthプロンプト](/assets/images/oauth_prompt.png)

あなた自身を信頼している場合、クリックしてください**アプリケーションを許可**。 WUH-OH！シナトラが吐き出します
`404`エラー。何ができます！

私たちがするコールバックURLを指定したときだけでなく、覚えています`折り返し電話`？我々は提供していませんでした
そのためのルート、彼らは承認した後、そのユーザーを削除するにはわかりません {{ site.data.variables.product.product_name }}
アプリ。それでは、その問題を修正しましょう！

### コールバックを提供

に_server.rb_、コールバックが何をすべきかを指定するためにルートを追加します。

``` ruby
'/コールバック' を取得＃は、一時的なGitHubのコードを取得しない... Session_code = request.env＃...とGitHubの結果に戻ってそれをPOST = RestClient.post（ 'https://github.com/login/oauth/access_token' 、：受け入れる=>：JSON）＃抽出トークン付与スコープaccess_tokenは= JSON.parse（結果）端 ['rack.request.query_hash'] ['access_token'] {:client_id => CLIENT_ID, :client_secret => CLIENT_SECRET, :code => session_code} ['code']```

成功したアプリの認証後、一時的に提供 {{ site.data.variables.product.product_name }}`コード`値。
まず、あなたがする必要があります`役職`引き換えに、背中に、このコード {{ site.data.variables.product.product_name }}`アクセストークン`.
私たちのGETとPOSTのHTTPリクエストを簡素化するために、私たちが使用しています[残りのクライアント][REST Client].
あなたはおそらく、RESTを通じてAPIにアクセスすることは決してないだろうことに注意してください。より深刻なのために
アプリケーションは、おそらく使用する必要があります[お好みの言語で書かれたライブラリー][libraries].

### 付与されたスコープを確認します

将来的には、ユーザーができるようになります[あなたが要求したスコープを編集][edit scopes post],
そして、あなたのアプリケーションは、もともとのために尋ねたよりも少ないアクセス権を付与される可能性があります。
だから、トークンを持つすべての要求を行う前に、あなたはそのスコープを確認してください
ユーザーがトークンのために付与されました。

付与されたスコープはからの応答の一部として返されます
トークンを交換します。

``` ruby
我々は、ユーザーが許可された場合は、「/コールバック」を実行＃... ##上記のコードサンプルを使用してaccess_tokenはを取得...＃チェックを取得：電子メールのスコープスコープ= JSON.parse（結果）.split（「」）has_user_email_scope = scopes.include？ 「ユーザー：メール」末端 ['scope']```

私たちのアプリケーションでは、使用しています`Scopes.include？`私たちが付与されたかどうかを確認します
インクルード`ユーザー：電子メール`認証されたユーザーの秘密を取得するために必要な範囲
メールアドレス。アプリケーションが他のスコープのために求めていた、我々は持っているでしょう
同様にそれらをチェック。

スコープ間の階層関係がありますので、また、あなたがすべき
あなたが必要なスコープの最低レベルを付与されたことを確認してください。例えば、
アプリケーションは、のために求めていた場合`ユーザー`スコープ、それだけで許可されている可能性があります
`ユーザー：電子メール`範囲。その場合、アプリケーションが付与されていません
何それはを求めたが、許可されたスコープは、まだ十分なされていると思います。

それは可能だからのみ要求を行う前に、スコープの確認は十分ではありません
ことは、ユーザーがあなたのチェックと実際の要求の間にスコープを変更します。
起こる場合は、あなたが成功すると予想API呼び出しがで失敗する可能性があります`404`
または`401`状態、または情報の異なるサブセットを返します。

あなたは優雅に、これらの状況、要求のためのすべてのAPIの応答を処理するために役立ちます
有効なトークンで作らも含まれてい[`X-OAuthの-スコープ`ヘッダ][oauth scopes].
このヘッダは、作るために使用されたトークンのスコープのリストが含まれています
要求。それに加えて、認証APIは、エンドポイントへの提供します
[有効性のためのトークンをチェックします][check token valid].
トークンのスコープの変化を検出するために、この情報を使用して、あなたのユーザーに通知
利用可能なアプリケーション機能の変更。

### 認証された要求を行います

最後に、このアクセストークンを使用して、あなたはとして認証要求を行うことができます
ログインしているユーザー：

``` ruby
＃フェッチユーザ情報auth_result = JSON.parse（RestClient.get（ 'https://api.github.com/user'}））、ユーザがそれを許可した場合、プライベートメールをフェッチ＃場合has_user_email_scope auth_result = JSON.parse（RestClient {:params => {:access_token => access_token} .get（ 'https://api.github.com/user/emails'}））エンドERB：基本的な、：地元=> auth_result ['private_emails'] {:params => {:access_token => access_token}```

我々は我々の結果でやりたいことができます。この場合、私たちはまっすぐにそれらをダンプします_basic.erb_:

``` erb
<p> こんにちは、<％=ログイン％>！ <％！のemail.nil場合は？ &&！email.empty？あなたの公開メールアドレスは<％=電子メール％>であるように％>に見えます。 <％それ以外％>あなたは公共の電子メールを持っていないように見えます。カッコいい。 </p> <p> <％は終わり％> <％定義されている場合は？あなたの秘密について、<％= private_emails.map .join（「」）％> <％それ以外％>また、あなたがしているビット秘密：private_emails％>あなたの許可を得て、我々はまた、あなたのプライベートのメールアドレスを掘ることができましたメールアドレス。 <％の終わり％> { |private_email_address| private_email_address["email"] } </p> <p> </p>```

## 「永続」の認証を実装します

我々はアプリにすべての単一のログを記録するようにユーザーに必要な場合、それはかなり悪いモデルになるだろう
彼らは、Webページにアクセスするのに必要な時間。例えば、直接ナビゲートしてみてください
`Http：// localhostを：4567/basic /コールバック`。あなたは、エラーを取得します。

我々は全体を回避することができればどのような
単にプロセスを「ここをクリック」、および_思い出します_限り、ユーザのがログインしています、
{{ site.data.variables.product.product_name }} 、彼らはこのアプリケーションにアクセスできるようにすべきですか？あなたの帽子を保持、
なぜなら_それは私たちがやろうとしているまさにです_.

上記の私たちの小さなサーバーはかなり単純です。いくつかのインテリジェントに食い​​込ませるために
認証は、我々は、トークンを格納するためのセッションを使用してに切り替えるつもりです。
これは、ユーザーに認証が透明になります。

私たちはセッション内でスコープを持続していることからも、我々はする必要があります
ユーザーは、我々はそれらを確認した後にスコープを更新し、または取り消したときにケースを扱います
トークン。これを行うには、我々は、使用します`レスキュー`最初のAPIをブロックしていることを確認
呼び出しは、トークンがまだ有効であることを確認した、成功しました。その後、我々はよ
チェック`X-OAuthの-スコープ`ユーザが失効していないことを確認する応答ヘッダ
インクルード`ユーザー：電子メール`範囲。

ファイルを作成します。_advanced_server.rb_、およびそれにこれらの行を貼り付けます。

``` ruby
「シナトラが」「JSON」＃が必要な「rest_client」を必要と必要！ EVER REAL APPにハードコードされた値を使用しないでください！ ENV && ENV＃CLIENT_ID = ENV＃CLIENT_SECRET = ENV＃エンドCLIENT_ID = ENV CLIENT_SECRET = ENV利用のラックであれば##以下のような代わりに、設定およびテスト環境変数を、::セッション::プール、：DEF認証さcookie_only => falseを？セッション終了はデフ認証します！ ERB：インデックス、：！地元の人々=>エンドGET '/' 場合は、認証されたのですか？認証する！トークンがあったため、救助=> Eの＃リクエストが成功しなかった他のaccess_tokenは=セッションスコープ=はauth_result = RestClient.get（JSON}：：=>を受け入れる 'https://api.github.com/user'、、）開始します私たちは＃セッションに保存されているトークンを無効にし、ユーザーがセッション= nilのリターンが認証再びOAuthのフローを開始できるように、＃インデックスページをレンダリング取り消さ！エンド＃要求は成功しましたので、auth_result.headers.include場合、我々は、現在のスコープのリストを確認しますか？ ：x_oauth_scopesスコープ= auth_result.headers .split（ ''）エンドauth_result = ['GITHUB_CLIENT_ID'] JSON.parse（auth_result）scopes.include場合は？ 'ユーザ：メール' auth_result ['GITHUB_CLIENT_SECRET'] = JSON.parse（RestClient.get（ 'https://api.github.com/user/emails'、：受け入れる=>：JSON}））エンドERB：高度、：地元=> auth_resultエンド・エンドのget '/コールバック' ['GITHUB_CLIENT_ID'] を実行session_code = request.env結果= RestClient.postセッション= JSON.parse（（ ['GITHUB_CLIENT_SECRET'] 'https://github.com/login/oauth/access_token'、：JSON：=>を受け入れます）結果）リダイレクト「/」末端 {:client_id => CLIENT_ID, :client_secret => CLIENT_SECRET, :code => session_code} [:access_token] [:access_token] [:access_token] ['code'] ['GH_BASIC_CLIENT_ID'] ['private_emails'] ['access_token'] {:params => {:access_token => access_token} [] {:client_id => CLIENT_ID} {:params => {:access_token => access_token} ['GH_BASIC_SECRET_ID'] [:access_token] ['rack.request.query_hash'] [:x_oauth_scopes]```

コードの多くはおなじみのはずです。例えば、我々はまだ使用しています`RestClient.get`
APIを呼び出すために、我々はまだ我々の結果を渡しているレンダリングされます {{ site.data.variables.product.product_name }}
ERBテンプレート（この時点では、それが呼ばれています`advanced.erb`).

また、我々が今持っています`認証されましたか？`ユーザが既にあるかどうかを確認する方法
認証されました。そうでない場合は、`認証する！`この方法は、実行する、と呼ばれています
OAuthが流れ、許可されたトークンとスコープとのセッションを更新します。

次に、中のファイルを作成します_景色_呼ばれます_advanced.erb_、そしてそれには、このマークアップを貼り付けます。

``` erb
<html> <head> </head> <body> <p> まあ、まあ、まあ、<％=は、ログイン％>！ <％！のemail.empty場合は？あなたの公開メールアドレスは<％=電子メール％>であるように％>に見えます。 <％それ以外％>あなたは公共の電子メールを持っていないように見えます。カッコいい。 <％は終わり％> <％定義されている場合は？あなたの秘密について、<％= private_emails.map </p> <p> .join（「」）％> <％それ以外％>また、あなたがしているビット秘密：private_emails％>あなたの許可を得て、我々はまた、あなたのプライベートのメールアドレスを掘ることができましたメールアドレス。 <％の終わり％> </p> </body> </html> { |private_email_address| private_email_address["email"] } <p> </p>```

コマンドラインから、コール`ルビーadvanced_server.rb`、あなたを開始します
ポート上のサーバー`4567`- 私たちは、単純なシナトラのアプリを持っていたときに我々が使用したのと同じポート。
あなたはに移動すると`Http：// localhostを：4567`、アプリコール`認証する！`
これはあなたがにリダイレクト`/折り返し電話`.`/折り返し電話`その後に私たちを送り返します`/`,
私たちが認証されてきたので、レンダリング_advanced.erb_.

我々は完全に、単に私たちのコールバックを変更することにより、この往復のルーティングを簡素化することができ
の中のURL {{ site.data.variables.product.product_name }}`/`。しかし、両方以来_server.rb_そして_advanced.rb_依存しています
同じコールバックURLは、我々はそれを動作させるためにwonkinessの少しを行うようになってきました。

また、我々は我々のデータにアクセスするには、このアプリケーションを承認したことがなかった場合には、 {{ site.data.variables.product.product_name }}
我々は以前のポップアップから同じ確認ダイアログを見て、私たちに警告しただろう。

ご希望の場合はで遊ぶことができます[さらに別のシナトラ、GitHubの認証の例][sinatra auth github test]
別のプロジェクトとして利用できます。

[webflow]: / V3 / OAuthの/＃Webアプリケーションフロー
[Sinatra]: http://www.sinatrarb.com/
[about env vars]: http://en.wikipedia.org/wiki/Environment_variable#Getting_and_setting_environment_variables
[Sinatra guide]: https://github.com/sinatra/sinatra-book/blob/master/book/Introduction.markdown#hello-world-application
[REST Client]: https://github.com/archiloque/rest-client
[libraries]: /ライブラリ/
[sinatra auth github test]: https://github.com/atmos/sinatra-auth-github-test
[oauth scopes]: / V3 /のOAuth /＃スコープ
[edit scopes post]: /変更/ 2013年10月4日 - OAuthの-変更-来て/
[check token valid]: / V3 / oauth_authorizations /＃チェック-承認
[platform samples]: https://github.com/github/platform-samples/tree/master/api/ruby/basics-of-authentication
[new oauth app]: https://github.com/settings/applications/new
[app settings]: https://github.com/settings/developers
