# HTTPリダイレクト

- [リダイレクトの作成](#creating-redirects)
- [名前付きルートへのリダイレクト](#redirecting-named-routes)
- [コントローラアクションへのリダイレクト](#redirecting-controller-actions)
- [一時保持セッションデータを伴うリダイレクト](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## リダイレクトの作成

リダイレクトレスポンスは、`Illuminate\Http\RedirectResponse`クラスのインスタンスで、ユーザーを他のURLへリダイレクトするために必要なしっかりとしたヘッダを含んでいる必要があります。`RedirectResponse`インスタンスを生成する方法はいくつかあります。一番簡単なのは、グローバルな`redirect`ヘルパを使用する方法です。

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

無効なフォームが送信された場合など、まれにユーザーを直前のページへリダイレクトする必要が起きます。グローバルな`back`ヘルパ関数を使用することで行なえます。この機能は[セッション](/docs/{{version}}/session)を利用しているため、`back`関数を呼び出すルートが`web`ミドルウェアグループを使用しているか、全セッションミドルウェアを確実に適用してください。

```php
Route::post('/user/profile', function () {
    // リクエスのバリデート処理…

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
## 名前付きルートへのリダイレクト

`redirect`ヘルパを引数なしで呼び出すと、`Illuminate\Routing\Redirector`インスタンスが返されるため、`Redirector`インスタンスの全メソッドを呼び出せます。たとえば、名前付きルートへの`RedirectResponse`を生成する場合は、`route`メソッドを使用できます。

```php
return redirect()->route('login');
```

ルートにパラメータが必要な場合は、`route`メソッドの第２引数として渡してください。

```php
// profile/{id}ルートへのリダイレクト

return redirect()->route('profile', ['id' => 1]);
```

使い勝手を良くするため, Laravelは`to_route`グローバル関数も提供しています。

```php
return to_route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### Eloquentモデルを使用したパラメータの埋め込み

あるEloquentモデルの"ID"パラメータを含むルートへリダイレクトする場合は、そのモデル自身を渡してください。IDは自動的に取り出されます。

```php
// profile/{id}ルートへのリダイレクト

return redirect()->route('profile', [$user]);
```

ルートパラメータへ含める値をカスタマイズしたい場合は、Eloquentモデルの`getRouteKey`メソッドをオーバーライドしてください。

```php
/**
 * モデルのルートキー値の取得
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
## コントローラアクションへのリダイレクト

[コントローラアクション](/docs/{{version}}/controllers)へのリダイレクトを生成することもできます。そのためには、`action`メソッドへコントローラとアクションの名前を渡してください。

```php
use App\Http\Controllers\HomeController;

return redirect()->action([HomeController::class, 'index']);
```

コントローラルートがパラメータを必要としている場合、`action`メソッドの第２引数として渡してください。

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-with-flashed-session-data"></a>
## 一時保持セッションデータを伴うリダイレクト

新しいURLへリダイレクトし、[セッションへデータを保持する](/docs/{{version}}/session#flash-data)のは、通常同時に行います。あるアクションが正しく実行された後に、セッションへ成功メッセージを一時保持データとして保存するのが典型的なケースでしょう。便利なように、`RedirectResponse`インスタンスを生成し、データをセッションへ一時保持データとして保存するには、記述的なチェーン一つで行なえます。

```php
Route::post('/user/profile', function () {
    // ユーザープロフィールの更新処理…

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

ユーザーを新しい場所にリダイレクトする前に、`RedirectResponse`インスタンスが提供する`withInput`メソッドを使用して、現在のリクエストの入力データをセッションへ一時保存できます。入力をセッションへ一時保存すると、次のリクエスト中に簡単に[取得](/docs/{{version}}/requests#retrieveing-old-input)できます。

```php
return back()->withInput();
```

ユーザーをリダイレクトしたあと、[セッション](/docs/{{version}}/session)の一時保持データとして保存したメッセージを表示できます。例として[Blade記法](/docs/{{version}}/blade)を使ってみます。

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
