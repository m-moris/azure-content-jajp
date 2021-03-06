<properties
	pageTitle="Azure Active Directory B2C プレビュー: Graph API を使用する | Microsoft Azure"
	description="アプリケーション ID を使用して B2C テナント用の Graph API を呼び出してプロセスを自動化する方法。"
	services="active-directory-b2c"
	documentationCenter=".net"
	authors="dstrockis"
	manager="msmbaldwin"
	editor=""/>

<tags
	ms.service="active-directory-b2c"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="01/21/2016"
	ms.author="dastrock"/>

# Azure AD B2C プレビュー: Graph API の使用


<!-- TODO [AZURE.INCLUDE [active-directory-b2c-devquickstarts-graph-switcher](../../includes/active-directory-b2c-devquickstarts-graph-switcher.md)]-->

Azure Active Directory (Azure AD) B2C テナントは非常に大規模になる傾向があります。これは、多くの一般的なテナント管理タスクをプログラムで実行する必要があることを意味します。主な例にはユーザーの管理があります。たとえば、既存のユーザー ストアを B2C テナントに移行することがあります。その場合、自分のページでユーザー登録をホストし、バックグラウンドで Azure AD のユーザー アカウントを作成することがあります。この種のタスクでは、ユーザー アカウントの作成、読み取り、更新、削除を実行する機能が必要です。Azure AD Graph API を使用してこれらの操作を実行できます。

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-preview-note.md)]

B2C テナントでは、主に 2 つのモードで Graph API と通信します。

- 対話型の一度だけ実行されるタスクでは、タスクの実行時に B2C テナントの管理者アカウントとして実行する必要があります。このモードでは、Graph API を呼び出す前に管理者が資格情報でサインインする必要があります。
- 自動化された継続的なタスクでは、必要な権限を付与した何らかの種類のサービス アカウントを使用し、管理タスクを実行する必要があります。Azure AD では、アプリケーションを登録して、Azure AD に認証することでそれを実行できます。その際、[OAuth 2.0 クライアント資格情報付与](../active-directory/active-directory-authentication-scenarios.md#daemon-or-server-application-to-web-api)を利用する**アプリケーション ID** を使用します。この場合、アプリケーションはユーザーとしてではなくアプリケーション自体として Graph API を呼び出します。

この記事では、自動化された事例を実行する方法を示します。実演として、ユーザーの CRUD (create, read, update, and delete/作成、読み取り、更新と削除) 操作を実行する .NET 4.5 `B2CGraphClient` を構築します。クライアントにはさまざまなメソッドを呼び出すことができる Windows コマンド ライン インターフェイス (CLI) を与えます。ただし、コードは、非対話型の自動化された方法で動作するように記述されます。

## Azure AD B2C テナントを取得する

アプリケーションまたはユーザーを作成する前に、あるいは Azure AD と対話する前に、Azure AD B2C テナントとそのテナントのグローバル管理者アカウントを用意する必要があります。テナントをまだ用意していない場合、[Azure AD B2C を始めてください](active-directory-b2c-get-started.md)。

## サービス アプリケーションをテナントに登録する

B2C テナントが用意できたら、Azure AD PowerShell コマンドレットを使用してサービス アプリケーションを作成する必要があります。最初に、[Microsoft Online Services サインイン アシスタント](http://go.microsoft.com/fwlink/?LinkID=286152) をダウンロードしてインストールします。次に、[64-bit Azure Active Directory Module for Windows PowerShell](http://go.microsoft.com/fwlink/p/?linkid=236297) をダウンロードしてインストールします。

> [AZURE.NOTE]
B2C テナントで Graph API を使用するには、PowerShell を使用して専用のアプリケーションを登録する必要があります。この記事の指示に従い、それを行います。Azure ポータルに登録済みの既存の B2C アプリケーションを再利用することはできません。これは Azure AD B2C プレビューの制限であり、近い将来、この制限は削除されます。その時点でこの記事は更新される予定です。

PowerShell モジュールをインストールした後、PowerShell を開き、B2C テナントに接続します。`Get-Credential` を実行すると、ユーザー名とパスワードが求められます。B2C テナント管理者アカウントのユーザー名とパスワードを入力します。

```
> $msolcred = Get-Credential
> Connect-MsolService -credential $msolcred
```

アプリケーションを作成する前に、新しい**クライアント シークレット**を生成する必要があります。アプリケーションはこのクライアント シークレットを使用し、Azure AD に対する認証を行ってアクセス トークンを取得します。PowerShell で有効なシークレットを生成できます。

```
> $bytes = New-Object Byte[] 32
> $rand = [System.Security.Cryptography.RandomNumberGenerator]::Create()
> $rand.GetBytes($bytes)
> $rand.Dispose()
> $newClientSecret = [System.Convert]::ToBase64String($bytes)
> $newClientSecret
```

最後のコマンドによって新しいクライアント シークレットが出力されます。これを安全な場所にコピーします。この情報は後で必要になります。これで、新しいクライアント シークレットをアプリの資格情報として指定して、アプリケーションを作成できます。

```
> New-MsolServicePrincipal -DisplayName "My New B2C Graph API App" -Type password -Value $newClientSecret

DisplayName           : My New B2C Graph API App
ServicePrincipalNames : {dd02c40f-1325-46c2-a118-4659db8a55d5}
ObjectId              : e2bde258-6691-410b-879c-b1f88d9ef664
AppPrincipalId        : dd02c40f-1325-46c2-a118-4659db8a55d5
TrustedForDelegation  : False
AccountEnabled        : True
Addresses             : {}
KeyType               : Password
KeyId                 : a261e39d-953e-4d6a-8d70-1f915e054ef9
StartDate             : 9/2/2015 1:33:09 AM
EndDate               : 9/2/2016 1:33:09 AM
Usage                 : Verify
```

アプリケーションが作成できたら、上記のようなアプリケーションのプロパティが出力されます。`ObjectId` と `AppPrincipalId` の両方が必要になります。これらの値もコピーしてください。

B2C テナント内にアプリケーションを作成したら、ユーザーの CRUD 操作を実行するために必要なアクセス許可をそのアプリケーションに割り当てる必要があります。アプリケーションにディレクトリ リーダー (ユーザーを読み取るため)、ディレクトリ ライター (ユーザーを作成し、更新するため)、ユーザー アカウント管理者 (ユーザーを削除するため) という 3 つのロールを割り当てます。これらのロールには既知の識別子があります。`-RoleMemberObjectId` パラメーターを前述の `ObjectId` に置き換え、次のコマンドを実行できます。すべてのディレクトリ ロールの一覧を表示するには、`Get-MsolRole` を実行してみてください。

```
> Add-MsolRoleMember -RoleObjectId 88d8e3e3-8f55-4a1e-953a-9b9898b8876b -RoleMemberObjectId <Your-ObjectId> -RoleMemberType servicePrincipal
> Add-MsolRoleMember -RoleObjectId 9360feb5-f418-4baa-8175-e2a00bac4301 -RoleMemberObjectId <Your-ObjectId> -RoleMemberType servicePrincipal
> Add-MsolRoleMember -RoleObjectId fe930be7-5e62-47db-91af-98c3a49a38b1 -RoleMemberObjectId <Your-ObjectId> -RoleMemberType servicePrincipal
```

これで B2C テナントに対してユーザーの作成、読み取り、更新、削除を実行する権限を持つアプリケーションが用意されました。

## サンプル コードをダウンロード、構成、ビルドする

まず、サンプル コードをダウンロードして実行します。次に、それを詳しく観察します。[サンプル コードは .zip ファイルとしてダウンロード](https://github.com/AzureADQuickStarts/B2C-GraphAPI-DotNet/archive/master.zip)できます。選択したディレクトリで複製することもできます。

```
git clone https://github.com/AzureADQuickStarts/B2C-GraphAPI-DotNet.git
```

`B2CGraphClient\B2CGraphClient.sln` Visual Studio ソリューションを Visual Studio で開きます。`B2CGraphClient` プロジェクトで、`App.config` ファイルを開きます。アプリの 3 つの設定を独自の値に置き換えます。

```
<appSettings>
    <add key="b2c:Tenant" value="contosob2c.onmicrosoft.com" />
    <add key="b2c:ClientId" value="{The AppPrincipalId from above}" />
    <add key="b2c:ClientSecret" value="{The client secret you generated above}" />
</appSettings>
```

[AZURE.INCLUDE [active-directory-b2c-devquickstarts-tenant-name](../../includes/active-directory-b2c-devquickstarts-tenant-name.md)]

次に、`B2CGraphClient` ソリューションを右クリックしてサンプルをリビルドします。リビルドできると、`B2CGraphClient\bin\Debug` に `B2C.exe` 実行可能ファイルが表示されます。

## Graph API を使用してユーザーの CRUD 操作をビルドする

B2CGraphClient を使用するには、`cmd` Windows コマンド プロンプトを開き、`Debug` ディレクトリにディレクトリを変更します。`B2C Help` コマンドを実行します。

```
> cd B2CGraphClient\bin\Debug
> B2C Help
```

各コマンドの短い説明が表示されます。これらのコマンドのいずれかを呼び出すたびに、`B2CGraphClient` は Azure AD Graph API に要求を行います。

### アクセス トークンを取得する

Graph API に要求するには、認証のためのアクセス トークンが必要になります。`B2CGraphClient` では、オープン ソースの Active Directory Authentication Library (ADAL) を利用し、アクセス トークンを取得します。ADAL は、単純な API を提供し、アクセス トークンのキャッシュなどのいくつかの重要な詳細を処理することで、トークンの取得を容易にします。ADAL を使用してトークンを取得する必要はありません。HTTP 要求を作成してトークンを取得することもできます。

> [AZURE.NOTE]
	このコード サンプルでは、一般に利用できる ADAL のバージョンである ADAL v2 を使用しています。Azure AD B2C で使用するためのプレビュー バージョンである ADAL v4 は使用していません。Azure AD B2C プレビューでは、Graph API と通信を行うには ADAL v2 を使用する必要があります。今後、ADAL v4 による Graph API へのアクセスが提供される予定です。完成した Azure AD B2C ソリューションで 2 つの ADAL バージョンを使用する必要はありません。

`B2CGraphClient` を実行すると、`B2CGraphClient` クラスのインスタンスが作成されます。このクラスのコンストラクターは、ADAL の認証のスキャフォールディングを設定します。

```C#
public B2CGraphClient(string clientId, string clientSecret, string tenant)
{
	// The client_id, client_secret, and tenant are provided in Program.cs, which pulls the values from App.config
	this.clientId = clientId;
	this.clientSecret = clientSecret;
	this.tenant = tenant;

	// The AuthenticationContext is ADAL's primary class, in which you indicate the tenant to use.
	this.authContext = new AuthenticationContext("https://login.microsoftonline.com/" + tenant);

	// The ClientCredential is where you pass in your client_id and client_secret, which are
	// provided to Azure AD in order to receive an access_token by using the app's identity.
	this.credential = new ClientCredential(clientId, clientSecret);
}
```

例として、`B2C Get-User` コマンドを使用します。`B2C Get-User` がそれ以外の入力なしで呼び出されると、CLI は `B2CGraphClient.GetAllUsers(...)` メソッドを呼び出します。このメソッドは、HTTP GET 要求を Graph API に送信する `B2CGraphClient.SendGraphGetRequest(...)` を呼び出します。`B2CGraphClient.SendGraphGetRequest(...)` は GET 要求を送信する前に、ADAL を使用してアクセス トークンを取得します。

```C#
public async Task<string> SendGraphGetRequest(string api, string query)
{
	// First, use ADAL to acquire a token by using the app's identity (the credential)
	// The first parameter is the resource we want an access_token for; in this case, the Graph API.
	AuthenticationResult result = authContext.AcquireToken("https://graph.windows.net", credential);

	...

```

ADAL の `AuthenticationContext.AcquireToken(...)` メソッドを呼び出すことで、Graph API 用のアクセス トークンを取得できます。ADAL はアプリケーションの ID を表す `access_token` を返します。

### ユーザーを読み取る

Graph API からユーザーの一覧または特定のユーザーを取得するには、HTTP `GET` 要求を `/users` エンドポイントに送信します。テナントのすべてのユーザーを要求した場合、次のようになります。

```
GET https://graph.windows.net/contosob2c.onmicrosoft.com/users?api-version=beta
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
```

この要求を確認するには、次のコマンドを実行します。

 ```
 > B2C Get-User
 ```

注意すべき重要な点が 2 つあります。

- ADAL 経由で取得されたアクセス トークンは、`Bearer` スキームを使用して `Authorization` ヘッダーに追加されます。
- B2C テナントに対しては、クエリ パラメーター `api-version=beta` を使用する必要があります。


> [AZURE.NOTE]
	Azure AD Graph API のベータ版の機能はプレビュー段階です。ベータ版の詳細については、[Graph API チームのこのブログ投稿](http://blogs.msdn.com/b/aadgraphteam/archive/2015/04/10/graph-api-versioning-and-the-new-beta-version.aspx)をご覧ください。

これらの詳細は、どちらも `B2CGraphClient.SendGraphGetRequest(...)` メソッド内で処理されます。

```C#
public async Task<string> SendGraphGetRequest(string api, string query)
{
	...

	// For B2C user management, be sure to use the beta Graph API version.
	HttpClient http = new HttpClient();
	string url = "https://graph.windows.net/" + tenant + api + "?" + "api-version=beta";
	if (!string.IsNullOrEmpty(query))
	{
		url += "&" + query;
	}

	// Append the access token for the Graph API to the Authorization header of the request by using the Bearer scheme.
	HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Get, url);
	request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);
	HttpResponseMessage response = await http.SendAsync(request);

	...
```

### コンシューマー ユーザー アカウントを作成する

B2C テナントにユーザー アカウントを作成するときは、HTTP `POST` 要求を `/users` エンドポイントに送信できます。

```
POST https://graph.windows.net/contosob2c.onmicrosoft.com/users?api-version=beta
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
Content-Type: application/json
Content-Length: 338

{
	// All of these properties are required to create consumer users.

	"accountEnabled": true,
	"alternativeSignInNamesInfo": [             // controls which identifier the user uses to sign in to the account
		{
			"type": "emailAddress",             // can be 'emailAddress' or 'userName'
			"value": "joeconsumer@gmail.com"
		}
	],
	"creationType": "NameCoexistence",          // always set to 'NameCoexistence'
	"displayName": "Joe Consumer",				// a value that can be used for displaying to the end user
	"mailNickname": "joec",						// an email alias for the user
	"passwordProfile": {
		"password": "P@ssword!",
		"forceChangePasswordNextLogin": false   // always set to false
	},
	"passwordPolicies": "DisablePasswordExpiration"
}
```

この要求のプロパティはすべて、コンシューマー ユーザーの作成に必要です。説明のために、`//` コメントが追加されています。実際の要求には追加しないでください。

要求を確認するには、次のコマンドのいずれかを実行します。

```
> B2C Create-User ..\..\..\usertemplate-email.json
> B2C Create-User ..\..\..\usertemplate-username.json
```

`Create-User` コマンドは、入力パラメーターとして .json ファイルを受け取ります。これには JSON 表記のユーザー オブジェクトが含まれます。サンプル コードには 2 つのサンプル .json ファイル、`usertemplate-email.json` と `usertemplate-username.json` があります。ニーズに合わせてこれらのファイルを変更できます。上記の必須フィールドに加えて、利用できるいくつかの任意フィールドがこれらのファイルに含まれています。任意フィールドの詳細については、「[Azure AD Graph API entity reference (Azure AD Graph API のエンティティ リファレンス)](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#UserEntity)」をご覧ください。

POST 要求が `B2CGraphClient.SendGraphPostRequest(...)` にどのように構成されているかを確認できます。

- アクセス トークンを要求の `Authorization` ヘッダーに添付します。
- `api-version=beta` を設定します。
- 要求の本文に JSON ユーザー オブジェクトを追加します。

### コンシューマー ユーザー アカウントを更新する

ユーザー オブジェクトの更新プロセスはユーザー オブジェクトの作成プロセスと同じです。ただし、HTTP `PATCH` メソッドが使用されます。

```
PATCH https://graph.windows.net/contosob2c.onmicrosoft.com/users/<user-object-id>?api-version=beta
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
Content-Type: application/json
Content-Length: 37

{
	"displayName": "Joe Consumer",				// this request updates only the user's displayName
}
```

JSON ファイルを新しいデータで更新することでユーザーを更新してください。それから `B2CGraphClient` を利用し、これらのコマンドのいずれかを実行できます。

```
> B2C Update-User <user-object-id> ..\..\..\usertemplate-email.json
> B2C Update-User <user-object-id> ..\..\..\usertemplate-username.json
```

`B2CGraphClient.SendGraphPatchRequest(...)` メソッドを調べて、この要求がどのように送信されるかの詳細を確認してください。

### ユーザーを削除する

ユーザーの削除プロセスは簡単です。HTTP `DELETE` メソッドを使用し、正しいオブジェクト ID で URL を構築します。

```
DELETE https://graph.windows.net/contosob2c.onmicrosoft.com/users/<user-object-id>?api-version=beta
Authorization: Bearer eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJod...
```

例を確認するには、このコマンドを実行し、コンソールに出力される DELETE 要求を観察します。

```
> B2C Delete-User <object-id-of-user>
```

`B2CGraphClient.SendGraphDeleteRequest(...)` メソッドを調べて、この要求がどのように送信されるかの詳細を確認してください。

ユーザー管理に加え、Azure AD Graph API では他にもさまざまなアクションを実行できます。「[Azure AD Graph API reference (Azure AD Graph API リファレンス)](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/api-catalog)」には、各アクションの詳細が要求の例とともに記載されています。

## カスタム属性を使用する

ほとんどのコンシューマー アプリケーションは、何らかの種類のカスタム ユーザー プロファイル情報を格納する必要があります。その方法の 1 つとして、B2C テナントにカスタム属性を定義できます。定義後、ユーザー オブジェクトの他のプロパティと同様にその属性を処理できます。属性を更新したり、属性を削除したり、属性に基づいてクエリを実行したり、サインイン トークンの中で要求として属性を送信したりできます。

B2C テナント内でカスタム属性を定義するには、[B2C プレビューのカスタム属性に関するリファレンス](active-directory-b2c-reference-custom-attr.md)を参照してください。

B2C テナント内に定義されたカスタム属性は、`B2CGraphClient` を使用して表示できます。

```
> B2C Get-B2C-Application
> B2C Get-Extension-Attribute <object-id-in-the-output-of-the-above-command>
```

これらの関数の出力によって、次のような各カスタム属性の詳細が明らかになります。

```JSON
{
      "odata.type": "Microsoft.DirectoryServices.ExtensionProperty",
      "objectType": "ExtensionProperty",
      "objectId": "cec6391b-204d-42fe-8f7c-89c2b1964fca",
      "deletionTimestamp": null,
      "appDisplayName": "",
      "name": "extension_55dc0861f9a44eb999e0a8a872204adb_Jersey_Number",
      "dataType": "Integer",
      "isSyncedFromOnPremises": false,
      "targetObjects": [
        "User"
      ]
}
```

ユーザー オブジェクトのプロパティとして、`extension_55dc0861f9a44eb999e0a8a872204adb_Jersey_Number` などの完全名を使用できます。新しいプロパティとその値で .json ファイルを更新し、次を実行します。

```
> B2C Update-User <object-id-of-user> <path-to-json-file>
```

`B2CGraphClient` の使用により、B2C テナント ユーザーをプログラミングで管理できるサービス アプリケーションが与えられます。`B2CGraphClient` ではそれ自体のアプリケーション ID を利用し、Azure AD Graph API に認証します。また、クライアント シークレットを使用してトークンが取得されます。アプリケーションにこの機能を組み込むときは、B2C アプリに関するいくつかの重要な点に注意してください。

- テナントの適切なアクセス許可をアプリケーションに付与する必要があります。
- 現時点では、ADAL v2 を使用してアクセス トークンを取得する必要があります。(ライブラリを使用せず、プロトコル メッセージを直接送信することもできます。)
- Graph API を呼び出すとき、[`api-version=beta`](http://blogs.msdn.com/b/aadgraphteam/archive/2015/04/10/graph-api-versioning-and-the-new-beta-version.aspx) を使用します。
- コンシューマー ユーザーを作成し、更新するとき、上述のようにいくつかのプロパティが必要になります。

> [AZURE.IMPORTANT] B2C アプリで Azure AD Graph API を使用するとき、Azure AD B2C の基礎にあるディレクトリ サービスのレプリケーション特性を考慮する必要があります。(詳細については、[この記事](http://blogs.technet.com/b/ad/archive/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-geo-distributed-cloud-directory.aspx)をご覧ください。) コンシューマーが**サインアップ** ポリシーを使用して B2C にサインアップした直後にアプリ内で Azure AD Graph API を使用してユーザー オブジェクトを読み取ろうとしてもできない場合があります。その際は、レプリケーション プロセスが完了するまで数秒待つ必要があります。一般公開時には、Azure AD Graph API とディレクトリ サービスによって提供される "書き込みと読み取りの整合性保証" について、より具体的なガイダンスを公開する予定です。

B2C テナントで Graph API を利用して実行するアクションに関するご質問やご要望がございましたら、この記事にコメントを投稿するか、GitHub コード サンプル リポジトリで問題を提出してください。

<!---HONumber=AcomDC_0302_2016-->