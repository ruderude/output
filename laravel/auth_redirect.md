# Laravelでログイン後、直前のページに遷移したい

ロールごととか１箇所に指定するのはやったことありますが、動的に変化させるにはどうしたらいいかわからず調べたことをまとめます。

ログイン後のページ遷移をデフォルトのHOMEや設定したページだけでなく、例えばカート機能でいざ決済する時にログインを促したりした場合、ログイン後は「そのページに戻ってきたい場合」などありますよね。

※Laravel7で動作確認  
※ほとんどこの記事のままです
[【Laravel】ログインした後、固定ページに飛ばさずにログインボタンを押した画面に戻りたい。](https://qiita.com/nekyo/items/f179875a8bcfba671785)

## デフォルトはHome画面に遷移するようになっています

`$redirectTo`にリダイレクト先が入っています。
`RouteServiceProvider::HOME`は定数で`Providers\RouteServiceProvider.php`にあります。

`LoginController`

```php
class LoginController extends Controller
{
    /*
    |--------------------------------------------------------------------------
    | Login Controller
    |--------------------------------------------------------------------------
    |
    | This controller handles authenticating users for the application and
    | redirecting them to your home screen. The controller uses a trait
    | to conveniently provide its functionality to your applications.
    |
    */

    use AuthenticatesUsers;

    /**
     * Where to redirect users after login.
     *
     * @var string
     */
    protected $redirectTo = RouteServiceProvider::HOME;

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }
}
```

<br>

とりあえず定数で保持されていて、通常ログイン後のリダイレクトを変更したりするのはここを変更すればOKです。

<br>

`RouteServiceProvider.php`

```php
    /**
     * The path to the "home" route for your application.
     *
     * @var string
     */
    public const HOME = '/home'; // <=ここ
```

<br>

### トレイトのAuthenticatesUsers.phpを見てみる

ログインできた後は `redirect()->intended($this->redirectPath());` を呼んでいます。
これは、認証フィルターでキャッチされる前にアクセスしたURLへリダイレクトさせる処理です。

`AuthenticatesUsers.php`

```php
    /**
     * Send the response after the user was authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendLoginResponse(Request $request)
    {
        $request->session()->regenerate();

        $this->clearLoginAttempts($request);

        if ($response = $this->authenticated($request, $this->guard()->user())) {
            return $response;
        }

        return $request->wantsJson()
                    ? new Response('', 204)
                    : redirect()->intended($this->redirectPath());
    }
```

### instended解析

instended は「意図された」という意味で機能は以下。

- セッションの 'url.instended' に値が設定されていたらそこにリダイレクトする。
- 値が設定されていなかったら '/' にリダイレクトする。

### 今回やりたいこと

認証フィルターはキャッチ前に 'url.instended' に参照元URLを設定しています。
ログイン画面が直接呼ばれた場合は、その値が設定されないため '/' 画面に固定で遷移してしまうわけです。
であれば、ログイン画面に遷移した際に 'url.instended' を設定してやれば認証フィルターでキャッチされたと同様の流れになるはず！

### 実装

ログインフォームを表示する showLoginForm メソッドを LoginController.phpにオーバーライドします。
このタイミングで直前のページを 'url.instended' セッションに設定します。
これで認証フィルターと同じ流れになり、ログイン画面直前にいたページに遷移されるはずです。

`LoginController.php`

 ```php
    /**
     * Show the application's login form.
     *
     * @return \Illuminate\Http\Response
     */
    public function showLoginForm()
    {
        // ここから
        if (array_key_exists('HTTP_REFERER', $_SERVER)) {
            $path = parse_url($_SERVER['HTTP_REFERER']); // URLを分解
            if (array_key_exists('host', $path)) {
                if ($path['host'] == $_SERVER['HTTP_HOST']) { // ホスト部分が自ホストと同じ
                    session(['url.intended' => $_SERVER['HTTP_REFERER']]);
                }
            }
        }
        // ここまで追加
        return view('auth.login');
    }

 ```

<br>

## レジスターも同じようにしたい

新規登録画面でも同じ仕様にしたいですが、ログインと同じやり方では上手くいきませんでした。
そこで、フォームに `url()->previous()` を仕込み、その値をバックエンド側でレダイレクトする作戦にしました。

`register.blade.php`

```php
・・・省略・・・

<form method="POST" action="{{ route('register') }}">
    @csrf

    @if( count( $errors->all() ) < 1)
        <input type="hidden" name="previous_url" value="{{ url()->previous() }}">
    @else
        <input type="hidden" name="previous_url" value="{{ old('previous_url') }}">
    @endif

    ・・・省略・・・

```

↑
バリデーションに引っかかった時用にoldヘルパー関数でセッション値を仕込みます。
これで直前のページをバックエンドへ送ったあと、`RegistersUsers.php`トレイトの`registeredメソッド`をレジスターコントローラーでオーバーライドします。
_
`LoginController.php`

```php
・・・省略・・・

    /**
     * The user has been registered.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return mixed
     */
    protected function registered(Request $request)
    {
        return redirect($request->previous_url);
    }

```

これで新規登録後も直前の画面に遷移することができました！

## まとめ

Laravel様、セッション様、便利な機能ありがとうございますm(_ _)m
