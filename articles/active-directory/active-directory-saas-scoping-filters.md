<properties
	pageTitle="スコープ フィルターを使用する属性ベースのアプリ プロビジョニング | Microsoft Azure"
	description="スコープ フィルターを使用して、自動ユーザー プロビジョニングをサポートするアプリ内のオブジェクトが、ビジネス要件を満たしていないのに実際にプロビジョニングされてしまうことを防ぐ方法について説明します。"
	services="active-directory"
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
	ms.date="02/09/2016"
	ms.author="markusvi"/>


# スコープ フィルターを使用する属性ベースのアプリ プロビジョニング

このセクションでは、スコープ フィルターを使用して属性ベースのルールを定義する方法について説明します。このルールで、アプリケーションに対してプロビジョニングするユーザーを指定します。





## 句とスコープ グループ


![スコープ フィルター][1]




スコープ フィルターは、1 つ以上の**スコープ グループ**で定義され、それぞれが 1 つ以上の**句**を保持します。特定のスコープ グループの句を表示するには、グループ名の左側の矢印をクリックして展開します。

**句**は、各ユーザーの属性を評価することによって、スコープ フィルターを通過できるユーザーを決定します。たとえば、ユーザーの「都道府県」属性が New York と等しいことを求める、つまり New York ユーザーのみをアプリケーションにプロビジョニングする句を設定することができます。

![スコープ グループ名][2]



各**スコープ グループ**は、上のスクリーンショットに示すように、1 つの必須の**句**ではじまります。この句は、ユーザーをスコープ フィルターによって評価する前に、まずアプリケーションに割り当てる必要があることを示しています。この句を削除または変更することはできません。

新しい句または新しいスコープ グループを追加するには、適切なボタンをクリックします。各スコープ グループに名前を付けるには、その**スコープ グループ名**プロパティを編集します。





## スコープ フィルターを評価する方法

プロビジョニング中は、ユーザーがアプリケーションにアクセスする資格があるかどうかを判断するために、すべての割り当てられているユーザーをスコープ フィルターに対してテストします。各句は、ユーザーがフィルターで除外されないために合格する必要のあるテストと考えることができます。

複数のスコープ グループを定義している場合は、各ユーザーがアプリケーションにアクセスするためには、ユーザーは少なくとも 1 つのスコープ グループで適格とされる必要があります。ただし、ユーザーが特定のスコープ グループで適格とされるためには、各スコープ グループ内のすべての句で適格とされる必要があります。

つまり、スコープ グループは OR でまとめられたもの、スコープ グループ内の句は AND でまとめられたものと考えることができます。たとえば、以下のスコープ フィルターがあるとします。


![スコープ グループ名][2]


このスコープ フィルターに従うと、ユーザーは、プロビジョニングされるために次の条件を満たす必要があります。

1. アプリケーションに割り当てられている

2. エンジニアリング部門で働いている

3. San Francisco または Canada で働いている


##関連記事

- [Article Index for Application Management in Azure Active Directory](active-directory-apps-index.md)
- [SaaS アプリへのユーザー プロビジョニングとプロビジョニング解除の自動化](active-directory-saas-app-provisioning.md)
- [ユーザーのプロビジョニング用の属性マッピングのカスタマイズ](active-directory-saas-customizing-attribute-mappings.md)
- [属性マッピングの式の書き方](active-directory-saas-writing-expressions-for-attribute-mappings.md)
- [アカウント プロビジョニング通知](active-directory-saas-account-provisioning-notifications.md)
- [SCIM を使用して、Azure Active Directory からアプリケーションへのユーザーとグループの自動プロビジョニングを有効にする](active-directory-scim-provisioning.md)
- [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](active-directory-saas-tutorial-list.md)

<!--Image references-->
[1]: ./media/active-directory-saas-scoping-filters/ic782811.png
[2]: ./media/active-directory-saas-scoping-filters/ic782812.png
[3]: ./active-directory-saas-scoping-filters/ic782813.png

<!---HONumber=AcomDC_0211_2016-->