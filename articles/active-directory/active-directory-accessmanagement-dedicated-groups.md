<properties
	pageTitle="Azure Active Directory の専用グループ | Microsoft Azure"
	description="Azure Active Directory の専用グループの機能と作成方法の概要について説明します。"
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""
	/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/09/2016"
	ms.author="curtand"/>

# Azure Active Directory の専用グループ

Azure Active Directory では、専用グループが自動的に作成され、そのグループ メンバーシップも自動的に作成されます。専用グループのメンバーは、Azure ポータル、Windows PowerShell コマンドレット、プログラムのいずれの方法でも追加したり削除したりすることはできません。専用グループを有効にするには、Azure ポータルの [構成] タブで **[専用グループを有効にする] を [はい]** に設定します。

[専用グループの有効化] を **[はい]** に設定した後、**["すべてのユーザー" グループの有効化]** を **[はい]** に設定することで、"すべてのユーザー" 専用グループをディレクトリで自動的に作成できるようになります。その後、**["すべてのユーザー" グループの表示名]** で、この専用グループの名前を編集できます。

All Users 専用グループは、ディレクトリ内のすべてのユーザーに同じアクセス許可を割り当てる場合に便利です。たとえば、ディレクトリ内のすべてのユーザーに SaaS アプリケーションへのアクセス権を与えるには、All Users 専用グループに対してこのアプリケーションへのアクセス権を割り当てます。

専用 "All Users" グループには、ディレクトリ内のすべてのユーザーが含まれており、これには、ゲストおよび外部ユーザーが含まれていることに注意してください。外部ユーザーを除外するグループが必要な場合、次のような動的ルールでグループを作成することによって、これを実現できます。

(user.userPrincipalName notContains"#EXT #@")

すべてのゲストを除外するグループの場合は、次のようなルールを使用してください。

(user.userType-ne"Guest")

この記事では、Azure Active Directory でグループのメンバーを管理するルールを作成する方法の詳細について説明します。

* [単純なルールを作成してグループの動的メンバーシップを構成する](active-directory-accessmanagement-simplerulegroup.md)


次の記事は、Azure Active Directory に関する追加情報を示します。

* [Azure Active Directory グループによるリソースのアクセス管理](active-directory-manage-groups.md)
* [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)
* [Azure Active Directory とは](active-directory-whatis.md)
* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0224_2016-->