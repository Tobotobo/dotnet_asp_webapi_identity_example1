# dotnet_asp_webapi_identity_example1

## 概要
以下の記事を試したサンプルプログラム

Identity を使用して SPA の Web API バックエンドをセキュリティで保護する方法  
https://learn.microsoft.com/ja-jp/aspnet/core/security/authentication/identity-api-authorization?view=aspnetcore-8.0  

## 詳細

### プロジェクト作成

```
dotnet new webapi --use-controllers 
```

```
dotnet run
```
http://localhost:5147/weatherforecast
http://localhost:5147/swagger

```
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

ApplicationDbContext.cs を作成
```cs
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<IdentityUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) :
        base(options)
    { }
}
```

Program.cs に以下を追加
```cs
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
builder.Services.AddAuthorization();

builder.Services.AddDbContext<ApplicationDbContext>(
    options => options.UseInMemoryDatabase("AppDb"));

builder.Services.AddIdentityApiEndpoints<IdentityUser>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
app.MapPost("/logout", async (SignInManager<IdentityUser> signInManager,
    [FromBody] object empty) =>
    {
        if (empty != null)
        {
            await signInManager.SignOutAsync();
            return Results.Ok();
        }
        return Results.Unauthorized();
    })
    .RequireAuthorization();
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
app.MapIdentityApi<IdentityUser>();
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
app.MapSwagger().RequireAuthorization();
```

Controllers/WeatherForecastController.cs に以下を追加
```cs
using Microsoft.AspNetCore.Authorization;
〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜〜
[Authorize]
```

### 実行

#### アクセスできないことの確認
* http://localhost:5147/WeatherForecast にアクセスすると `HTTP ERROR 401` になること
* もしくは以下を実行して `HTTP/1.1 401 Unauthorized` になること  
  ```
  curl -v -X 'GET' 'http://localhost:5147/WeatherForecast' -H 'accept: text/plain'
  ```

#### ユーザー登録
* http://localhost:5147/swagger にアクセス
* POST /register をクリック
* Try it out をクリック
* Request body に以下を入力  
  ```json
  {
    "email": "test@example.com",
    "password": "Test123!"
  }
  ```
* Execute をクリック

もしくは以下を実行
```
curl -v -X 'POST' 'http://localhost:5147/register' -H 'accept: */*' -H 'Content-Type: application/json' -d '{ "email": "test@example.com", "password": "Test123!" }'
```

#### ログイン
* http://localhost:5147/swagger にアクセス
* POST /login をクリック
* Try it out をクリック
* Parameters の useCookies で true を選択
* Request body に以下を入力  
  ```json
  {
    "email": "test@example.com",
    "password": "Test123!"
  }
  ```  
  ※twoFactorXXXX は削除
* Execute をクリック

もしくは以下を実行
```
curl -c cookies.txt -v -X 'POST' 'http://localhost:5147/login?useCookies=true' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{ "email": "test@example.com", "password": "Test123!" }'
```

#### アクセスできることの確認
* http://localhost:5147/WeatherForecast にアクセスするとデータが表示されること
* もしくは以下を実行して `HTTP/1.1 200 OK` 且つデータが表示されること  
  ```
  curl -b cookies.txt -v -X 'GET' 'http://localhost:5147/WeatherForecast' -H 'accept: text/plain'
  ```

#### ログアウト
* http://localhost:5147/swagger にアクセス
* POST /logout をクリック
* Try it out をクリック
* Parameters の useCookies で true を選択
* Request body に以下を入力  
  ```json
  {}
  ```  
  ※"string" は削除
* Execute をクリック

もしくは以下を実行　※TODO: これを実行しても引き続きアクセスできてしまう。なんで？
```
curl -b cookies.txt -v -X 'POST' 'http://localhost:5147/logout' -H 'accept: */*' -H 'Content-Type: application/json' -d '{}'
```