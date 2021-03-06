
<properties
	pageTitle="SaaS アプリケーションへのアクセスをグループで管理する| Microsoft Azure"
	description="Azure Active Directory と連携する SaaS アプリケーションへのアクセス権を、Azure Active Directory Premium または Basic でグループを使用して割り当てる方法について説明します。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/09/2016"
	ms.author="curtand"/>


# SaaS アプリケーションへのアクセスをグループで管理する

Azure Active Directory (Azure AD) Premium では、Azure AD と連携する SaaS アプリケーションへのアクセス権を、グループを使って割り当てることができます。たとえばマーケティング部門を対象に 5 つの SaaS アプリケーションを使用するためのアクセス権を割り当てる場合、同部門のユーザーから成るグループを作成したうえで、マーケティングで必要となる 5 つの SaaS アプリケーションにそのグループを割り当てます。このようにマーケティング部門のメンバーシップを一元的に管理することで時間を節約することが可能です。この場合アプリケーションに対するユーザーの割り当ては、そのユーザーがマーケティング グループのメンバーとして追加されたときに行われます。同様に、マーケティング グループからユーザーが削除されると、アプリケーションからもその割り当てが削除されます。

この機能は、Azure AD アプリケーション ギャラリー内から追加できる多数のアプリケーションで利用することができます。

**SaaS アプリケーションに対してグループのアクセス権を割り当てるには**

1. 任意のブラウザーを開いて、Azure ポータルに移動します。Azure ポータルの左側のナビゲーション バーで Active Directory 拡張機能を選択します。**[ディレクトリ]** タブで、Saas アプリケーションに対するグループのアクセス権を割り当てるディレクトリをクリックします。


2. 目的のディレクトリの **[アプリケーション]** タブをクリックします。アプリケーション ギャラリーから追加したアプリケーションをクリックし、**[ユーザーとグループ]** タブをクリックします。

3. **[ユーザーとグループ]** タブで、アクセス権を割り当てるグループの名前を **[指定値で始まる]** フィールドに入力し、右上のチェック マークをオンにします。入力するのは、グループ名の先頭部分だけでかまいません。その後グループをクリックして強調表示し、**[アクセス権の割り当て]** ボタンをクリックして、確認のメッセージが表示されたら **[はい]** をクリックします。


4. アプリケーションに直接割り当てられているユーザーとグループのメンバーシップを通じて割り当てられているユーザーを表示することもできます。その場合は、[表示] ドロップ ダウンを **[グループ]** から **[すべてのユーザー]** に変更してください。ディレクトリ内のユーザーが一覧表示され、アプリケーションに割り当てられているかどうかがユーザーごとに表示されます。ユーザーがアプリケーションに直接割り当てられているのか ([割り当ての種類] = [直接])、グループのメンバーシップを通じて割り当てられているのか ([割り当ての種類] = [継承済み]) も、このリストで確認できます。


> [AZURE.NOTE]
[ユーザーとグループ] タブが表示されるのは、Azure AD Premium または Azure AD Basic を有効にした場合のみです。

##関連記事

次の記事は、Azure Active Directory に関する追加情報を示します。

* [Azure Active Directory グループによるリソースのアクセス管理](active-directory-manage-groups.md)

* [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)

* [Azure Active Directory とは](active-directory-whatis.md)

* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0211_2016-->