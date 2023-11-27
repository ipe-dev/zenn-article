---
title: "【Laravel】VerifyCsrfTokenの中身を調べてみた"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Laravel]
published: false
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