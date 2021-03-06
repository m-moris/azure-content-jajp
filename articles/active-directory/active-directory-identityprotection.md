<properties
	pageTitle="Azure Active Directory Identity Protection | Microsoft Azure"
	description="Azure AD Identity Protection を使用して、侵害された ID またはデバイスを攻撃者が悪用する能力を制限する方法、および以前に疑われた、または侵害を確認された ID またはデバイスを保護する方法について説明します。"
	services="active-directory"
	keywords="Azure Active Directory Identity Protection, Cloud App Discovery, アプリケーションの管理, セキュリティ, リスク, リスク レベル, 脆弱性, セキュリティ ポリシー"
	documentationCenter=""
	authors="markusvi"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/02/2016"
	ms.author="markvi"/>

#Azure Active Directory Identity Protection 

Azure Active Directory Identity Protection は、リスク イベントや組織の ID に影響する潜在的な脆弱性に関する統合ビューを提供するセキュリティ サービスです。Microsoft は 10 年以上にわたってクラウド ベースの ID を保護してきました。Azure AD Identity Protection は、同じ保護システムを企業のお客様が利用できるようにするものです。Identity Protection は、既存の Azure AD 異常検出機能 (Azure AD の異常アクティビティ レポートで利用可能) を利用し、リアルタイムで異常を検出できる新しいリスク イベントの種類が導入されています。

> [AZURE.NOTE] 現在、Identity Protection のプレビューは、米国内でプロビジョニングされている Azure AD テナントでのみ利用できます。
 

ほとんどのセキュリティ侵害は、攻撃者がユーザーの ID を盗むことにより環境にアクセスできるようになると発生します。攻撃者は、ますます巧妙にサード パーティの侵害を利用し、高度なフィッシング攻撃を使用するようになっています。低い特権のユーザー アカウントであってもアクセス権を得た攻撃者は、比較的簡単に横方向への移動によって重要な企業リソースにアクセスしてしまいます。したがって、すべての ID を保護し、ID が侵害された場合は侵害された ID が悪用されるのを事前に防止することが不可欠です。

侵害された ID を検出するのは簡単な作業はありません。Identity Protection がそれを助けてくれます。Identity Protection はアダプティブ機械学習アルゴリズムとヒューリスティックを使用して、ID が侵害されたことを示している可能性のある異常とリスク イベントを検出します。
 
このデータを使用して、Identity Protection はレポートとアラートを生成し、ユーザーがこれらのリスク イベントを調査して、適切な修復または軽減のアクションを実行できるようにします。
 
ただし、Azure Active Directory Identity Protection は単なる監視とレポート作成のツールではありません。リスク イベントを基にして、Identity Protection は各ユーザーのユーザー リスク レベルを計算し、ユーザーがリスク ベースのポリシーを構成して組織の ID を自動的に保護するできるようにします。これらのリスクに基づくポリシーと、Azure Active Directory および EMS によって提供される他の条件付きアクセス コントロールにより、パスワードのリセットや多要素認証の適用などのアダプティブ修復アクションを自動的にブロックまたは提供できます。

####Identity Protection の機能 

**リスク イベントとリスクの高いアカウントの検出:**

- 機械学習とヒューリスティック規則を使用して、6 種類のリスク イベントを検出します 

- ユーザーのリスク レベルを計算します

- 脆弱性を目立たせることにより全体的なセキュリティ対策を向上させるためのカスタム推奨事項を提供します

<br>

**リスク イベントの調査:**

- リスク イベントの通知を送信します

- 関連情報とコンテキスト情報を使用してリスク イベントを調査します

- 調査を追跡するための基本的なワークフローを提供します

- パスワード リセットなどの修復アクションへの簡単なアクセスを提供します

<br>

**リスクに基づく条件付きアクセス ポリシー:**

- サインインのブロックまたは多要素認証チャレンジの要求によりリスクの高いサインインを軽減するポリシー

- リスクの高いユーザー アカウントをブロックまたはセキュリティ保護するためのポリシー

- 多要素認証用のユーザーを登録するようユーザーに要求するポリシー


## 検出とリスク

### リスク イベント

リスク イベントは Identity Protection によって疑いありのフラグが設定されたイベントであり、ID が侵害されている可能性を示します。リスク イベントの詳細な一覧については、「[Types of Risk Events (リスク イベントの種類)](#types-of-risk-events)」を参照してください。これらのリスク イベントの一部は、Microsoft Azure 管理ポータルの Azure AD 異常アクティビティ レポートで利用できたものです。次の表では、さまざまなリスク イベントの種類と、対応する **Azure AD 異常なアクティビティ** レポートの一覧を示します。Microsoft はこの分野への投資を続けることにより、既存のリスク イベントの検出精度を継続的に向上させ、必要に応じて新しいリスク イベントの種類を追加する予定です。



| Identity Protection のリスク イベントの種類 | 対応する Azure AD 異常アクティビティ レポート |
| :-- | :-- |
| 漏洩した資格情報 | 資格情報が漏洩したユーザー |
| 特殊な場所へのあり得ない移動 |	不規則なサインイン アクティビティ |
| 感染しているデバイスからのサインイン | 感染している可能性があるデバイスからのサインイン |
| 匿名の IP アドレスからのサインイン | 不明なソースからのサインイン |
| 不審なアクティビティのある IP アドレスからのサインイン |	不審なアクティビティのある IP アドレスからのサインイン |
| 未知の場所からのサインイン | - | 
| ロックアウト イベント (パブリック プレビューには含まれません) | - |

以下の Azure AD 異常アクティビティ レポートは Azure AD Identity Protection のリスク イベントには含まれず、したがって Identity Protection では利用できません。これらのレポートは Microsoft Azure 管理ポータルではまだ使用できますが、Identity Protection のリスク イベントによって置き換えられるため、将来的には廃止されます。

- 複数のエラー後のサインイン
- 複数の地域からのサインイン

### リスク レベル

リスク イベントのリスク レベルは、リスク イベントの重大度 (高、中、低) を示します。リスク レベルは、Identity Protection のユーザーが組織に対するリスクの軽減に必要なアクションの優先順位を決定するときの参考になります。リスク イベントの重大度は、一般に含まれるノイズの量との組み合わせで、ID 侵害の予測因子としての信号の強度を表します。

- **高**: 確実性が高く、重大度が高いリスク イベント。これらのイベントはユーザーの ID が侵害されたことの強力なインジケーターであり、影響を受けたすべてのユーザー アカウントをすぐに修復する必要があります。

- **中**: 重大度が高く確実性が低いリスク イベント、またはその逆。これらのイベントはリスクが高い可能性があり、影響を受けたすべてのユーザー アカウントを修復する必要があります。

- **底**: 確実性が低く、重大度が低いリスク イベント。このイベントだけでは早急な対応は必要ありませんが、他のリスク イベントと組み合わせた場合、ID が侵害された強い指標となることがあります。


![リスク レベル](./media/active-directory-identityprotection/01.png "リスク レベル")

 

リスク イベントは、**リアルタイム**で識別されるか、またはリスク イベントが既に発生した後の後処理 (オフライン) で識別されます。現在、Identity Protection のほとんどのリスク イベントはオフラインで計算され、2 ～ 4 時間以内に Identity Protection に表示されます。リアルタイムのリスク イベントは、評価はリアルタイムで行われますが、Identity Protection コンソールに表示されるまでには 5 ～ 10 分かかります。

現在、一部のレガシー クライアントではリアルタイムのリスク イベントの検出と防止はサポートされていません。その結果、これらのクライアントからのサインインをリアルタイムで検出または防止することはできません。

### リスク イベントの種類

このセクションでは、使用できるリスク イベントの種類の概要について説明します

#### 漏洩した資格情報

漏洩した資格情報が悪質な Web で公開されていることを、Microsoft のセキュリティ調査員が発見します。通常、このような資格情報はプレーン テキストで発見されます。発見された資格情報は、Azure AD の資格情報と照合されて、一致した場合は、Identity Protection で "漏洩した資格情報" として報告されます。

漏洩した資格情報のリスク イベントは、攻撃者がユーザー名とパスワードを利用できる明らかな兆候となるので、"高" 重大度のリスク イベントに分類されます。

#### 特殊な場所へのあり得ない移動

このリスク イベントの種類では、地理的に離れた 2 つの場所で行われたサインインを識別します。少なくとも 1 つの場所は、ユーザーの過去の行動から考えて、ユーザーが普通いそうにない場所でもあります。さらに、2 つのサインインの間の時間が、最初の場所から 2 回目の場所にユーザーが移動するのに要する時間より短く、別のユーザーが同じ資格情報を使用していることを示唆します。

この機械学習アルゴリズムは、組織内の他のユーザーによって普通に使用される VPN や場所など、不可能な移動状況の原因になる明らかな "*誤検知*" を無視します。システムには 14 日間の初期学習期間があり、その間に新しいユーザーのサインイン行動が学習されます。

あり得ない移動は通常、ハッカーがサインインに成功したことのよいインジケーターとなります。ただし、ユーザーが新しい手段を使用して移動している場合、または組織内の他のユーザーが通常使用しない VPN を使用している場合、誤検知が発生する可能性があります。誤検知のもう 1 つの原因は、クライアント IP として誤ってサーバー IP を渡すアプリケーションです。その場合、そのアプリケーションのバックエンドがホストされているデータセンターからサインインが行われたように見えます (多くの場合、それは Microsoft のデータセンターであり、Microsoft 所有の IP アドレスからサインインが行われたように見えます)。このような誤検知のため、このリスク イベントのリスク レベルは "**中**" です。

####感染しているデバイスからのサインイン

このリスク イベントの種類は、ボット サーバーと頻繁に通信していることがわかっている、マルウェアに感染したデバイスからのサインインを示します。これは、ボット サーバーと接触していた IP アドレスに対してユーザーのデバイスの IP アドレスを関連付けることによって決定されます。

このリスク イベントは、ユーザーのデバイスではなく IP アドレスを識別します。1 つの IP アドレスを複数のデバイスが使用していて、その一部だけがボット ネットワークによって制御されている場合、他のデバイスからサインインすると不要なイベントがトリガーされるため、このリスク イベントは "**低**" に分類されています。

ユーザーに連絡して、ユーザーのすべてのデバイスをスキャンして安全を確認することをお勧めします。また、ユーザーの個人デバイスが感染している可能性、または前に説明したようにユーザーと同じ IP アドレスから他のユーザーがウイルスに感染したデバイスを使用していた可能性もあります。感染したデバイスは、ウイルス対策ソフトウェアによってまだ識別されていないマルウェアに感染していることが多く、デバイスが感染する原因になるユーザーの悪い習慣を示していることもあります。

マルウェア感染に対処する方法の詳細については、[マルウェア対策センター](http://go.microsoft.com/fwlink/?linkid=335773&clcid=0x409)を参照してください。


#### 匿名の IP アドレスからのサインイン

このリスク イベントの種類は、匿名プロキシ IP アドレスとして識別されている IP アドレスからのサインインにユーザーが成功したことを示します。このようなプロキシは、自分のデバイスの IP アドレスを隠したいユーザーによって使用され、悪意のある目的で使用される場合があります。

直ちにユーザーに連絡し、匿名 IP アドレスを使用していたかどうかを確認することをお勧めします。匿名 IP 自体はアカウント侵害を強く示しているわけではないので、このリスク イベントの種類のリスク レベルは "**中**" です。

#### 不審なアクティビティのある IP アドレスからのサインイン

このリスク イベントの種類は、短期間に複数のユーザー アカウントで多数のサインイン試行失敗が検出された IP アドレスを示します。これは攻撃者によって使用された IP アドレスのトラフィック パターンと一致し、アカウントが既に侵害されたこと、または侵害されようとしていることを示す強いインジケーターです。これは、組織内の他のユーザーがよく使用する IP アドレスなど、明確な "*誤検知*" を無視する機械学習アルゴリズムです。システムには 14 日間の初期学習期間があり、その間に新しいユーザーおよび新しいテナントのサインイン行動が学習されます。

ユーザーに問い合わせて、疑わしい IP アドレスから実際にサインインしたかどうかを確認することをお勧めします。同じ IP アドレスを使用する複数のデバイスの一部だけが疑わしいアクティビティの原因になっている可能性があるため、このイベント種類のリスク レベルは "**中**" です。


#### 未知の場所からのサインイン

このリスク イベントの種類はリアルタイム サインイン評価メカニズムであり、過去のサインインの場所 (IP、緯度/経度、ASN) を考慮して、新規/未知の場所を決定します。システムは、ユーザーが過去に使用した場所に関する情報を保持し、これらを "既知の" 場所と考えます。既知の場所のリストにまだ含まれない場所からサインインが行われると、リスク イベントがトリガーされます。システムには 14 日間の初期学習期間があり、この間はどの新しい場所にも未知の場所としてのフラグは設定されません。また、システムは、既知のデバイスからのサインイン、および既知の場所と地理的に近い場所からのサインインも無視します。<br>未知の場所は、攻撃者が盗まれた ID を使用しようとしていることを強く示している場合があります。ユーザーが移動しているとき、新しいデバイスを試した場合、新しい VPN を使用したときなど、誤検知が発生することがあります。このような誤検知のため、このリスク イベントの種類のリスク レベルは "**中**" です。

### 脆弱性

脆弱性は、攻撃者によって悪用される可能性のある環境内の弱点です。これらの脆弱性に対処して組織のセキュリティ対策を強化し、攻撃者による脆弱性の悪用を防ぐことをお勧めします。

#### 多要素認証の登録が構成されていない 

この脆弱性は、組織内の Azure Multi-Factor Authentication のデプロイの制御に役立ちます。

Azure Multi-Factor Authentication は、ユーザー認証に対して第 2 のセキュリティ層を提供します。シンプルなサインイン プロセスを好むユーザーのニーズに応えながら、データやアプリケーションへのアクセスを効果的に保護することが可能です。電話やテキスト メッセージ、モバイル アプリによる通知のほか、確認コードやサード パーティの OATH トークンなど、一連の簡単な照合方法を通じて確実な認証を行うことができます。

ユーザーのサインインに対して多要素認証を要求することをお勧めします。多要素認証は、Identity Protection で使用可能なリスクに基づく条件付きアクセス ポリシーにおいて重要な役割を果たします。

詳細については、「[Azure Multi-Factor Authentication とは](../multi-factor-authentication/multi-factor-authentication.md)」を参照してください。


#### 管理されていないクラウド アプリ

この脆弱性は、組織内にある管理されていないクラウド アプリの識別に役立ちます。
 
現代の企業では、IT 部門が、組織のユーザーが作業のために使用しているクラウド アプリケーションを部分的にしか認識できていないことがよくあります。管理者が企業データへの不正アクセスを心配している理由は容易に理解できます。データの漏洩やその他のセキュリティ リスクが発生するおそれがあるからです。

組織に Cloud App Discovery をデプロイして管理されていないクラウド アプリケーションを検出し、Azure Active Directory を使用してこれらのアプリケーションを管理することをお勧めします。

詳細については、「[管理されていないクラウド アプリケーションを Cloud App Discovery で検出する](active-directory-cloudappdiscovery-whatis.md)」を参照してください。



####Privileged Identity Management からのセキュリティ アラート

この脆弱性は、組織内の特権 ID に関するアラートを検出して解決するのに役立ちます。

ユーザーが特権操作を実行できるよう、組織はユーザーに Azure AD、Azure や Office 365 のリソース、他の SaaS アプリへの一時的または永続的な特権アクセスを付与することが必要になる場合があります。このような特権ユーザーにより組織の攻撃対象領域が増えます。この脆弱性は、不要な特権アクセスを持つユーザーを特定し、それらがもたらすリスクを軽減または除去するための適切なアクションを実施するのに役立ちます。

Azure AD Privileged Identity Management を使用して、特権 ID と、Azure AD および他の Microsoft オンライン サービス (Office 365 や Microsoft Intune など) のリソースへの特権 ID のアクセスを管理、制御、監視することをお勧めします。

詳細については、「[Azure AD Privileged Identity Management](active-directory-privileged-identity-management-configure.md)」を参照してください。

## 調査
Identity Protection を使用するときは、通常、Identity Protection ダッシュボードから開始します。

<br><br> ![Remediation](./media/active-directory-identityprotection/29.png "Remediation") <br>

ダッシュボードからは次のものにアクセスできます。
 
- **リスクのフラグ付きユーザー**、**リスク イベント**、**脆弱性**などのレポート
- **セキュリティ ポリシー**、**通知**、**多要素認証の登録**の構成などの設定
 

通常、これが調査の起点になります。調査は、修復または軽減の手順が必要であるかどうかを決定し、ID が侵害を受けているかどうか、およびどのように侵害されたかを理解し、侵害された ID がどのように使用されたかを理解するために、アクティビティ、ログ、およびリスク イベントに関連するその他の関連情報を確認するプロセスです。


### 通知

Azure AD Identity Protection では、ユーザーのリスクとリスク イベントの管理に役立つ 2 種類の自動通知電子メールが送信されます。

- ユーザー侵害アラート電子メール

- 週間ダイジェスト電子メール

#### ユーザー侵害アラート電子メール

ユーザー侵害電子メール アラートは、Azure AD Identity Protection がアカウントの侵害を識別すると生成されます。この電子メールには、Identity Protection コンソールのリスクのフラグ付きユーザー レポートへのリンクが含まれます。侵害ユーザーの通知はすぐに調査することをお勧めします。


#### 週間ダイジェスト電子メール

週間ダイジェスト電子メールには、新しいリスク イベントの概要が含まれます。<br> 次の情報が含まれます。
- リスクのあるユーザー
- 報告されたリスク イベント
- 検出された脆弱性
- Identity Protection の関連するレポートへのリンク


<br> ![Remediation](./media/active-directory-identityprotection/400.png "Remediation") <br>

既定では、侵害ユーザー アラートはすべての Azure Active Directory 管理者に送信されます。次の方法で受信者をカスタマイズできます。

- アラートの下部にある管理受信者のリンクをクリックします

- [Identity Protection の設定] の下にある [通知] をクリックすします
 
<br> ![ユーザーのリスク](./media/active-directory-identityprotection/62.png "ユーザーのリスク") <br>
 



## ユーザーのリスク レベルとは

ユーザーのリスク レベルは、ユーザーの ID が侵害された可能性を示す値 (高、中、低) です。ユーザーの ID に関連付けられているユーザー リスク イベントに基づいて計算されます。

リスク イベントの状態は、**アクティブ**または**クローズ**のいずれかです。ユーザー リスクの計算に使用されるのは、**アクティブ**なリスク イベントだけです。

ユーザー リスク レベルは、次の入力を使用して計算されます。

- ユーザーに影響を与えるアクティブなリスク イベント
- それらのイベントのリスク レベル 
- すべての修復アクションが実施されたかどうか 

<br> ![ユーザーのリスク](./media/active-directory-identityprotection/86.png "ユーザーのリスク") <br>



ユーザー リスク レベルを使用して、リスクの高いユーザーのサインインをブロックする、またはパスワードの安全な変更を強制する、条件付きアクセス ポリシーを作成できます。


## リスク イベントの手動クローズ

ほとんどの場合、セキュリティ保護されたパスワードのリセットなどの修復アクションを実行すると、リスク イベントは自動的にクローズします。ただし、それが不可能な場合もあります。<br>**アクティブ**なリスク イベントはユーザー リスク計算に使用されるため、リスク イベントを手動でクローズしてリスク レベルを下げることが必要になる場合があります。<br>調査の過程で、リスク イベントの状態を変更するいずれかの操作を実行できます。

<br> ![アクション](./media/active-directory-identityprotection/34.png "アクション") <br>

- **解決** - リスク イベントを調査した後、Identity Protection の外部で適切な修復アクションを行い、リスク イベントがクローズしたことが確実な場合、イベントを解決済みとしてマークします。解決されたイベントはリスク イベントの状態をクローズに設定し、そのリスク イベントはユーザー リスクに対して考慮されなくなります。

- **誤検知としてマーク** - 場合によっては、リスク イベントを調査した結果、誤って高リスクのフラグが設定されたことがわかる場合があります。リスク イベントを誤検知としてマークすることにより、このようなことの発生を減らすことができます。これは、今後機械学習アルゴリズムが同様のイベントの分類を向上させるのに役立ちます。誤検知イベントの状態は**クローズ**になり、ユーザー リスクに対して考慮されなくなります。

- **無視** - どのような修復アクションも行わずに、リスク イベントをアクティブ リストから削除する場合は、リスク イベントを無視に指定できます。イベントはクローズ状態になります。無視されたイベントは、ユーザー リスクに対して考慮されません。このオプションは、特殊な状況下でのみ使用する必要があります。

- **再アクティブ化** - **解決**、**誤検知**、または**無視**を選択することによって手動でクローズされたリスク イベントを再アクティブ化し、イベントの状態を**アクティブ**に戻すことができます。再アクティブ化されたリスク イベントは、ユーザー リスク レベルの計算に対して考慮されます。修復 (セキュリティ保護されたパスワードのリセットなど) によってクローズされたリスク イベントを再アクティブ化することはできません。



## ユーザー リスク イベントの修復

修復とは、以前に侵害の疑いがある、または侵害を検知した ID またはデバイスをセキュリティで保護するアクションです。修復アクションでは、ID またはデバイスを安全な状態に復元し、ID またはデバイスに関連付けられた以前のリスク イベントを解決します。

次の方法でユーザー リスク イベントを修復できます。

- セキュリティ保護されたパスワードのリセットを実行し、ユーザー リスク イベントを手動で修復します 

- ユーザーのリスク セキュリティ ポリシーを構成し、ユーザー リスク イベントを自動的に緩和または修復します

- 感染したデバイスを再イメージングします。


### セキュリティ保護されたパスワードのリセット

セキュリティ保護されたパスワードのリセットは、多くのリスク イベントに対する効果的な修復手段であり、実行すると、そのリスク イベントは自動的にクローズして、ユーザー リスク レベルが再計算されます。Identity Protection コンソールを使用して、リスクの高いユーザーのパスワード リセットを開始できます。

コンソールでパスワードをリセットするには 2 つの方法があります。

**パスワードのリセット** - **[ユーザーにパスワードのリセットを求める]** を選択すると、ユーザーが多要素認証に登録している場合、ユーザーは自分で回復できます。ユーザーは、次にサインインするとき、多要素認証のチャレンジを正しく解決することを要求され、その後、パスワードの変更を強制されます。ユーザー アカウントがまだ多要素認証に登録されていない場合、このオプションは使用できません。

**一時パスワード** - **[一時パスワードの生成]** を選択すると、すぐに既存のパスワードが無効化され、ユーザーに対して新しい一時パスワードが作成されます。ユーザーの連絡用メール アドレスまたはユーザーのマネージャーに、新しい一時パスワードを送信します。パスワードは一時的なので、ユーザーはサインイン時にパスワードの変更を求められます。

<br> ![ポリシー](./media/active-directory-identityprotection/71.png "ポリシー") <br>



## ユーザーのリスク セキュリティ ポリシー

ユーザーのリスク セキュリティ ポリシーは、特定のユーザーに対するリスク レベルを評価し、定義されている条件とルールに基づいて修復および軽減のアクションを適用する条件付きアクセス ポリシーです。<br><br> ![ユーザーのリスク ポリシー](./media/active-directory-identityprotection/500.png "ユーザーのリスク ポリシー") <br>

Azure AD Identity Protection では、リスクのフラグ付きユーザーの軽減策と修復を管理するために、次のことが可能です。

- ポリシーを適用するユーザーとグループを設定します<br><br> ![ユーザーのリスク ポリシー](./media/active-directory-identityprotection/501.png "ユーザーのリスク ポリシー") <br>

- パスワードの変更をトリガーするユーザー リスク レベルのしきい値を (低、中、高) を設定します<br><br> ![ユーザーのリスク ポリシー](./media/active-directory-identityprotection/502.png "ユーザーのリスク ポリシー") <br>

- ユーザーのブロックをトリガーするユーザー リスク レベルのしきい値を (低、中、高) を設定します<br><br> ![ユーザーのリスク ポリシー](./media/active-directory-identityprotection/503.png "ユーザーのリスク ポリシー") <br>

- 変更を実施する前に、変更の影響を確認および評価します<br><br> ![ユーザーのリスク ポリシー](./media/active-directory-identityprotection/504.png "ユーザーのリスク ポリシー") <br>


**高**しきい値を選択すると、ポリシーがトリガーされる回数が減り、ユーザーへの影響が最小限になります。ただし、**低**および**中**レベルのリスクのフラグ付きユーザーはポリシーから除外されるため、以前に疑いのあった、または侵害されたことが知られていた、ID またはデバイスをセキュリティで保護することはできません。

ポリシーを設定するときは次のようにします。

- 多数の誤検知を生成する可能性があるユーザー (開発者、セキュリティ アナリスト) を除外します。

- ポリシーを有効にするのが実際的でないロケールのユーザーを除外します (ヘルプデスクにアクセスできないユーザーなど)。

- ポリシーの初期展開中、またはエンド ユーザーに表示されるチャレンジを最小限に抑える必要がある場合は、**高**しきい値を使用します。

- 組織のセキュリティを強化する必要がある場合は、**低**しきい値を使用します。**低**しきい値を選択すると、追加のユーザー サインイン チャレンジが導入されますが、セキュリティは強化されます。

ほとんどの組織に対する既定の設定として、**中**しきい値の規則を構成し、使いやすさとセキュリティのバランスを取ることをお勧めします。



### ユーザー リスク イベントの緩和
管理者は、ユーザー リスク セキュリティ ポリシーを設定し、リスク レベルに応じてサインイン時にユーザーをブロックできます。

サインインをブロックすると、以下のことが行われます。
 
- 影響を受けるユーザーに対して新しいユーザー リスク イベントが生成されなくなります

- 管理者は、ユーザーの ID に影響を与えるリスク イベントを修復して、セキュリティで保護された状態に復元できます





### ユーザー リスクの修復と軽減のフロー  

以降のセクションでは、ユーザー リスクの修復と軽減のフローに関するユーザー エクスペリエンスの概要を示します。

#### 侵害されたアカウントの復旧フロー

ユーザーのリスク セキュリティ ポリシーが構成されている場合、ポリシーで指定されているユーザー リスク レベルを満たす (したがって侵害されているものと考えられる) ユーザーは、ユーザー侵害復旧フローを通過してからでないと、サインインできません。

ユーザー侵害復旧フローには、3 つのステップがあります。

1. 不審なアクティビティまたは漏洩した資格情報のためにアカウントのセキュリティにリスクがあることが、ユーザーに通知されます。

<br> ![Remediation](./media/active-directory-identityprotection/101.png "Remediation") <br>

2.	ユーザーは、セキュリティ チャレンジを解くことによって自分の ID を証明するように要求されます。ユーザーが多要素認証に登録されている場合、ユーザーは侵害状態から自力で復旧できます。ユーザーは、セキュリティ コードを電話番号に折り返させる必要があります。 

<br> ![Remediation](./media/active-directory-identityprotection/110.png "Remediation") <br>


3.	最後に、誰かがアカウントにアクセスした可能性があるため、ユーザーはパスワードの変更を強制されます。このエクスペリエンスのスクリーンショットは次のとおりです。
 
<br> ![Remediation](./media/active-directory-identityprotection/111.png "Remediation") <br>







 
#### パスワードのリセット
ユーザーがサインインからブロックされている場合、管理者はユーザー用に一時的なパスワードを生成できます。ユーザーは、次にサインインするときに、パスワードを変更する必要があります。パスワードの変更により、ほとんどの種類のユーザー リスク イベントは修復されてクローズします。

<br> ![Remediation](./media/active-directory-identityprotection/160.png "Remediation") <br>


### ユーザー リスク レベルの軽減

### 侵害されたアカウント: ブロック 

ユーザーのリスク セキュリティ ポリシーによってブロックされたユーザーをブロック解除するには、ユーザーが管理者またはヘルプ デスクに連絡する必要があります。この場合、多要素認証を解決することによる自己復旧は利用できません。

<br> ![Remediation](./media/active-directory-identityprotection/104.png "Remediation") <br>


## サインイン リスク イベントの軽減 
軽減とは、ID またはデバイスを安全な状態に復元せずに、侵害された ID またはデバイスを悪用されないように、攻撃者の能力を制限するアクションです。軽減策では、ID またはデバイスと関連付けられた以前のサインイン リスク イベントを解決することはありません。

Azure AD Identity Protection の条件付きアクセスを使用して、サイン イン リスク イベントを自動的に軽減できます。これらのポリシーを使用するときは、リスクの高いサインインをブロックする、またはユーザーに多要素認証の実行を要求する、ユーザーまたはサインインのリスク レベルを検討します。これらのアクションにより、攻撃者が盗んだ ID 利用して損害を与えるのを防ぎ、ID をセキュリティで保護する時間を稼ぐことができます。


## サインインのリスク セキュリティ ポリシー

サインインのリスク ポリシーは、事前定義された条件と規則に基づいて、特定のサインインのリスクを評価し、対応策に適用する条件付きアクセス ポリシーです。<br><br>![サインインのリスク ポリシー](./media/active-directory-identityprotection/700.png "サインインのリスク ポリシー")<br>

Azure AD Identity Protection では、リスクの高いサインインの軽減策を管理するために、次のことが可能です。

- ポリシーを適用するユーザーとグループを設定します<br><br> ![サインインのリスク ポリシー](./media/active-directory-identityprotection/701.png "サインインのリスク ポリシー") <br>

- 影響を受けたサインインに対して多要素認証のチャレンジをトリガーするサインイン リスク レベルのしきい値 (低、中、高) を設定します<br><br> ![サインインのリスク ポリシー](./media/active-directory-identityprotection/702.png "サインインのリスク ポリシー") <br>

- 影響を受けたサインインをブロックするサインイン リスク レベルのしきい値 (低、中、高) を設定します<br><br> 。![サインインのリスク ポリシー](./media/active-directory-identityprotection/703.png "サインインのリスク ポリシー") <br>

- 変更を実施する前に、変更の影響を確認および評価します<br><br> ![サインインのリスク ポリシー](./media/active-directory-identityprotection/704.png "サインインのリスク ポリシー") <br>

 
**高**しきい値を選択すると、ポリシーがトリガーされる回数が減り、ユーザーへの影響が最小限になります。<br> ただし、**低**および**中**レベルのリスクのフラグ付きサインインはポリシーから除外されるため、攻撃者が侵害された ID を悪用するのをブロックすることはできません。

ポリシーを設定するときは次のようにします。

- 多要素認証を使用しない/できないユーザーを除外します

- ポリシーを有効にするのが実際的でないロケールのユーザーを除外します (ヘルプデスクにアクセスできないユーザーなど)。

- 多数の誤検知を生成する可能性があるユーザー (開発者、セキュリティ アナリスト) を除外します。

- ポリシーの初期展開中、またはエンド ユーザーに表示されるチャレンジを最小限に抑える必要がある場合は、**高**しきい値を使用します。

- 組織のセキュリティを強化する必要がある場合は、**低**しきい値を使用します。**低**しきい値を選択すると、追加のユーザー サインイン チャレンジが導入されますが、セキュリティは強化されます。

ほとんどの組織に対する既定の設定として、**中**しきい値の規則を構成し、使いやすさとセキュリティのバランスを取ることをお勧めします。

 
サインイン リスク ポリシーは次のように使用されます。

- 最新の認証を使用するすべてのブラウザー トラフィックとサインインに適用されます。
- ADFS などのフェデレーション IDP で WS-Trust エンドポイントを無効にすることによって従来のセキュリティ プロトコルを使用するアプリケーションには適用されません。

Identity Protection コンソールの **[リスク イベント]** ページには、以下に該当するすべてのイベントが一覧表示されます。

- そのポリシーが適用されたイベント
- アクティビティを確認して、アクションが適切だったかどうかを判断できるイベント 




## 多要素認証登録ポリシー

多要素認証は、ユーザー ID の保護を強化するために使用されます。<br> 多要素認証への登録は、アカウント侵害からの保護と回復を行うように組織を準備するときの重要なステップです。<br> ![MFA 登録](./media/active-directory-identityprotection/600.png "MFA 登録") <br>

Azure AD Identity Protection を使用すると、多要素認証の登録の展開を管理するために、以下のことが可能です。

- ポリシーを適用するユーザーとグループを設定します<br><br> ![MFA 登録](./media/active-directory-identityprotection/601.png "MFA 登録") <br>

- 登録のスキップを許可する時間を定義します <br><br> ![MFA 登録](./media/active-directory-identityprotection/602.png "MFA 登録") <br>

- 影響を受けたユーザーの現在の登録状態を表示します <br><br> ![MFA 登録](./media/active-directory-identityprotection/603.png "MFA 登録") <br>


##サインイン リスク軽減のフロー 

#### リスクの高いサインインの復旧フロー

管理者がサインイン リスクのポリシーを構成してある場合、影響を受けたユーザーはサインインを試みると通知を受け取ります。

高リスク サインインのフローには 2 つのステップがあります。

1. ユーザーは、新しい場所、デバイス、アプリからのサインインなど、サインインに関して異常が検出されたことの通知を受け取ります。<br> ![Remediation](./media/active-directory-identityprotection/120.png "Remediation") <br>

2. ユーザーは、セキュリティ チャレンジを解くことによって自分の ID を証明するように要求されます。多要素認証に登録しているユーザーは、電話番号にセキュリティ コードを折り返す必要があります。これは単にリスクの高いサインインであり、侵害されたアカウントではないので、このフローではユーザーがパスワードを変更する必要はありません。 <br> ![Remediation](./media/active-directory-identityprotection/121.png "Remediation") <br>
 
#### ブロックされたリスクの高いサインイン
管理者は、サインイン リスク ポリシーを設定し、リスク レベルに応じてサインイン時にユーザーをブロックすることもできます。ブロックを解除するには、エンドユーザーは管理者またはヘルプ デスクに連絡する必要があります。または、既知の場所またはデバイスからサインインを試みることもできます。この場合、多要素認証を解決することによる自己復旧は利用できません。

<br> ![Remediation](./media/active-directory-identityprotection/130.png "Remediation") <br>
 
#### 多要素認証の登録

侵害されたアカウントの復旧フローでも、リスクの高いサインインのフローでも、最善のユーザー エクスペリエンスは、ユーザーが自分で復旧できる場合です。多要素認証に登録されているユーザーは、セキュリティのチャレンジを解くために使用できる電話番号がアカウントに既に関連付けられています。アカウントの侵害から復旧するために、ヘルプ デスクまたは管理者の介入は必要ありません。したがって、ユーザーを多要素認証に登録させることを強くお勧めします。

管理者は次のことをできます。

- 追加のセキュリティ検証用にアカウントを設定するようユーザーに要求するポリシーを設定します。 
- 登録前に猶予期間をユーザーに与える場合、最大で 30 日間、多要素認証の登録のスキップを許可します。

 
 <br> ![Remediation](./media/active-directory-identityprotection/140.png "Remediation") <br> <br> ![Remediation](./media/active-directory-identityprotection/141.png "Remediation") <br> <br> ![Remediation](./media/active-directory-identityprotection/142.png "Remediation") <br>

 

#### リスクの高いサインインの間の多要素認証の登録

セキュリティのチャレンジを準備して通過できるようにユーザーが対要素認証に登録することが重要です。ユーザーが多要素認証に登録していない場合、ポリシーで多要素認証への登録が要求されていると、リスクの高いサインインの間にユーザーに登録が求められる可能性があります。つまり、正しいユーザーの代わりに、攻撃者に対して電話番号の追加の要求が行われます。<br>このような状況を避けるため、ユーザーは可能な限り早く多要素認証に登録し、侵害されたときには既に電話番号がアカウントに関連付けられているようにする必要があります。または、管理者は、多要素認証に登録していない侵害されたユーザーを完全にブロックすることもできます。

 <br> ![Remediation](./media/active-directory-identityprotection/150.png "Remediation") <br> <br> ![Remediation](./media/active-directory-identityprotection/151.png "Remediation") <br>






 
## プレイブック

### リスク イベントのシミュレーション

このセクションでは、次のリスク イベントの種類のシミュレーションの概要について説明します。

- 匿名の IP アドレスからのサインイン (簡単)

- 未知の場所からのサインイン (普通)

- 特殊な場所へのあり得ない移動 (困難)

他のリスク イベントは、安全な方法でシミュレートすることはできません。


#### 匿名の IP アドレスからのサインイン

このリスク イベントの種類は、匿名プロキシ IP アドレスとして識別されている IP アドレスからのサインインにユーザーが成功したことを示します。このようなプロキシは、自分のデバイスの IP アドレスを隠したいユーザーによって使用され、悪意のある目的で使用される場合があります。

匿名 IP からのサインインをシミュレートするには、次の手順を実行します。

1.	[Tor Browser](https://www.torproject.org/projects/torbrowser.html.en) をダウンロードします
2.	Tor Browser を使用して、https://myapps.microsoft.com に移動します   
3.	匿名 IP アドレスからのサインイン レポートに表示するアカウントの資格情報を入力します

以上で終了です。 5 分以内にサインインが Identity Protection に表示されます。


####未知の場所からのサインイン
未知の場所のリスクはリアルタイム サインイン評価メカニズムであり、過去のサインインの場所 (IP、緯度/経度、ASN) を考慮して、新規/未知の場所を決定します。システムは、ユーザーの以前の IP、緯度/経度、および ASN を保存し、それを "既知の" 場所と見なします。サインインの場所が既存の場所のいずれとも一致しない場合、そのサインインの場所は未知と見なされます。システムには 14 日間の初期学習期間があり、この間はどの新しい場所にも未知の場所としてのフラグは設定されません。また、システムは、既知のデバイスからのサインイン、および既存の場所と地理的に近い場所からのサインインも無視します。

未知の場所をシミュレートするには、それ以前にアカウントがサインインしていない場所とデバイスからサインインする必要があります。手順は次のとおりです。

1.	少なくとも 14 日間のサインイン履歴のあるアカウントを選択します。 

2.	次のどちらかの操作を行います。
	
    a.VPN を使用している状態で、[https://myapps.microsoft.com](https://myapps.microsoft.com) に移動し、リスク イベントをシミュレートするアカウントの資格情報を入力します

    b.別の場所にいる知り合いに、アカウントの資格情報を使用してサインインするよう頼みます (推奨されません)

以上で終了です。 5 分以内にサインインが Identity Protection に表示されます。
 
#### 特殊な場所へのあり得ない移動
アルゴリズムでは、機械学習を使用して、既知のデバイスからのあり得ない移動や、ディレクトリ内の他のユーザーによって使用される VPN からのサインインなどの誤検知が除去されるため、あり得ない移動の状態をシミュレートすることは困難です。さらに、ユーザーには、リスク イベントの生成を開始する前に、3 ～ 14 日間のサインイン履歴が必要です。

1.	標準的なブラウザーを使用して、[https://myapps.microsoft.com](https://myapps.microsoft.com) に移動します  

2.	あり得ない移動のリスク イベントを生成するアカウントの資格情報を入力します

3.	ここで、ユーザー エージェントを変更します。ユーザー エージェントの変更は、Internet Explorer の開発者ツールから、あるいは Firefox または Chrome のユーザー エージェント切り替えアドオンを使用して、行うことができます。

4.	次に、IP アドレスを変更します。IP アドレスの変更は、VPN または Tor アドオンを使用して、または別のデータセンターの Azure で新しいマシンを起動して、行うことができます。

5.	前と同じ資格情報を使用し、前のサインインから数分以内に、[https://myapps.microsoft.com](https://myapps.microsoft.com) にサインインします

2 ～ 4 時間以内にサインインが Identity Protection に表示されます。ただし、複雑な機械学習モデルが含まれるため、取得されない可能性があります。複数の Azure AD アカウントでこの手順を繰り返すのがよい方法です。


#### 脆弱性のシミュレーション 
脆弱性は、悪意のあるユーザーによって悪用される可能性のある Azure AD 環境内の弱点です。現在、Azure AD Identity Protection では、Azure AD の他の機能を利用する 3 種類の脆弱性が明らかになっています。これらの脆弱性は、以下の機能がセットアップされると Identity Protection に自動的に表示されます。

-	Azure AD [Multi-Factor Authentication](../multi-factor-authentication/multi-factor-authentication.md)
-	Azure AD [Cloud App Discovery](active-directory-cloudappdiscovery-whatis.md)
-	Azure AD [Privileged Identity Management](active-directory-privileged-identity-management-configure.md) 



###リスクに基づく条件付きアクセス ポリシー
####ユーザーの侵害リスク

**ユーザーの侵害リスクをテストするには、次の手順を実行します**。

1.	テナントのグローバル管理者の資格情報を使用して [https://portal.azure.com](https://portal.azure.com) にサインインします。

2.	**Identity Protection** に移動します。

3.	Azure AD Identity Protection のメイン ブレードで、**[設定]** をクリックします。

4.	**[ポータルの設定]** ブレードの **[セキュリティ規則]** で、**[ユーザーの侵害リスク]** をクリックします。

5.	**[サインインのリスク]** ブレードで、**[ルールの有効化]** をオフにして、**[保存]** をクリックします。

6.	特定のユーザー アカウントで、未知の場所または匿名 IP のリスク イベントをシミュレートします。これにより、そのユーザーのユーザー リスク レベルが**中**に上がります。

7.	数分待った後、ユーザーのユーザー レベルが**中**であることを確認します。

8.	**[ポータルの設定]** ブレードに移動します。

9.	**[ユーザーの侵害リスク]** ブレードで、**[ルールの有効化]** の **[オン]** を選択します。

10.	次のいずれかのオプションを選択します。

    a.ブロックするには、**[リスクが指定された設定値以上の場合、サインインをブロックする]** で **[中]** を選択します。

    b.セキュリティで保護されたパスワードの変更を強制するには、**[リスクが指定された設定値以上の場合、多要素認証が必要]** で **[中]** を選択します。

13.	**[保存]** をクリックします。

14. リスク レベルを上げたユーザーを使用してサインインすることにより、リスクに基づく条件付きアクセスをテストできます。ユーザーのリスクが中の場合、設定したポリシーに応じて、サインインがブロックされるか、またはパスワードの変更を強制されます。 <br><br> ![プレイブック](./media/active-directory-identityprotection/201.png "プレイブック") <br>

 
####サインイン リスク

 


サインイン リスクをテストするには、次の手順を実行します。

1.	テナントのグローバル管理者の資格情報を使用して [https://portal.azure.com](https://portal.azure.com) にサインインします。

2.	**Identity Protection** に移動します。

3.	**Azure AD Identity Protection** のメイン ブレードで、**[設定]** をクリックします。

4.	**[ポータルの設定]** ブレードの **[セキュリティ規則]** で、**[サインインのリスク]** をクリックします。

5.	[**サインインのリスク**] ブレードで、**[ルールの有効化]** の **[オン]** を選択します。

7.	次のいずれかのオプションを選択します。

    a.ブロックするには、**[リスクが指定された設定値以上の場合、サインインをブロックする]** で **[中]** を選択します。

    b.セキュリティで保護されたパスワードの変更を強制するには、**[リスクが指定された設定値以上の場合、多要素認証が必要]** で **[中]** を選択します。

8.	ブロックするには、[リスクが指定された設定値以上の場合、サインインをブロックする] で [中] を選択します。

9.	多要素認証を強制するには、[リスクが指定された設定値以上の場合、多要素認証が必要] で [中] を選択します。

10.	**[Save]** をクリックします。

11.	未知の場所または匿名 IP のリスク イベントをシミュレートすることにより、リスクに基づく条件付きアクセスをテストできます。どちらも中リスク イベントと見なされます。

<br> ![プレイブック](./media/active-directory-identityprotection/200.png "プレイブック") <br>


## 関連項目

 - [Azure AD and Identity Show: Identity Protection Preview (Azure AD および Identity ショー: Identity Protection プレビュー)](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)
 - [Identity Protection の用語集](active-directory-identityprotection-glossary.md)

<!-----HONumber=AcomDC_0302_2016-->