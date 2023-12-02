---
title: "【Laravel】VerifyCsrfTokenの中身を調べてみた"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Laravel]
published: true
---
Laravel内部でCSRFトークンをどのように使っているのかが気になったので、VerifyCsrfTokenの中身を調べてみました。
# 結論
- VerifyCsrfToken.php内のhandleメソッド内でtokenの判定が行われている
- postの場合はtokenがマッチするかを見ている
- その他の場合にはcookieにトークンをセットしている
# 環境
Laravel 10
# VerifyCsrfTokenとは
csrfトークンを用いてXSRF対策を行うミドルウェアです。
Kernel.phpの中を見るとwebの時のミドルウェアとして登録されています。
```php:Kernel.php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class, // これです
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```
# VerifyCsrfTokenの中身
## App内のVerifyCsrfToken
先ほどのVerifyCsrfTokenのパスをよく見るとAppの中になっています。
その中身が以下です。
```php:/App/Http/Middleware/VerifyCsrfToken.php
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array<int, string>
     */
    protected $except = [
        //
    ];
}
```
`$except`という配列がありますが、この配列に入っているパスはcsrfトークンによる検証をスキップします。
このクラスが継承している、パッケージのMiddlewareが今回見ていくクラスです。
## パッケージ内のVerifyCsrfToken
ここからは`Illuminate/Foundation/Http/Middleware/VerifyCsrfToken.php`の中を見ていきます。
### handleメソッド

```php
public function handle($request, Closure $next)
{
    if (
        $this->isReading($request) ||
        $this->runningUnitTests() ||
        $this->inExceptArray($request) ||
        $this->tokensMatch($request)
    ) {
        return tap($next($request), function ($response) use ($request) {
            if ($this->shouldAddXsrfTokenCookie()) {
                $this->addCookieToResponse($request, $response);
            }
        });
    }

    throw new TokenMismatchException('CSRF token mismatch.');
}
```
handleメソッドでは4つの条件のどれかに当てはまれば`tap`関数をreturnします。
- isReading
  - メソッドがhead,get,optionであるか
- runningUnitTests
  - ユニットテストを実行しているか
- inExceptArray 
  - 前述の`$expect`配列に入っている
- **tokenMatch**
  - tokenが一致するか

tap関数が受け取るクロージャでは新しいtokenをcookieにセットしています
次にtokenが一致するかを検証している`tokenMatch`メソッドについてみていきます
### tokenMatchメソッド
```php
protected function tokensMatch($request)
{
    $token = $this->getTokenFromRequest($request);

    return is_string($request->session()->token()) &&
           is_string($token) &&
           hash_equals($request->session()->token(), $token);
}
```
`getTokenFromRequest`でtokenを取得し、[ hash_equals関数 ](https://www.php.net/manual/ja/function.hash-equals.php)でsessionに保存したtokenとrequestで送られてきたtokenが一致するかを検証しています。
tokenが一致せずにエラーになる場合は、`$token`の中身と`$request->session()->token()`の中身がどうなっているかを確認するといいと思います。
次に`getTokenFromRequest`でのtoken取得方法をみていきます。
#### getTokenFromRequestメソッド
```php
protected function getTokenFromRequest($request)
{
    $token = $request->input('_token') ?: $request->header('X-CSRF-TOKEN');

    if (! $token && $header = $request->header('X-XSRF-TOKEN')) {
        try {
            $token = CookieValuePrefix::remove($this->encrypter->decrypt($header, static::serialized()));
        } catch (DecryptException) {
            $token = '';
        }
    }

    return $token;
}
```
ここではrequestに含まれる`_token`というパラメータか、ヘッダに含まれる`X-CSRF-TOKEN`の内容をtokenとして使用しています。
それでもtokenが取れなければ`X-XSRF-TOKEN`をヘッダから取得して復号化し、tokenとしてセットしています。