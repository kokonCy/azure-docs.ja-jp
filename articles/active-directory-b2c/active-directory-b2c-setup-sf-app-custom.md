---
title: "Azure Active Directory B2C: カスタム ポリシーを使用した Salesforce SAML プロバイダーの追加 | Microsoft Docs"
description: "Azure Active Directory B2C のカスタム ポリシーに関するトピック"
services: active-directory-b2c
documentationcenter: 
author: parakhj
manager: krassk
editor: parakhj
ms.assetid: d7f4143f-cd7c-4939-91a8-231a4104dc2c
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 06/11/2017
ms.author: parakhj
ms.translationtype: Human Translation
ms.sourcegitcommit: 857267f46f6a2d545fc402ebf3a12f21c62ecd21
ms.openlocfilehash: dec042efe30b80d68b3f200a4aad4fd5d24cd158
ms.contentlocale: ja-jp
ms.lasthandoff: 06/28/2017


---
# <a name="azure-active-directory-b2c-log-in-using-salesforce-accounts-via-saml"></a>Azure Active Directory B2C: SAML を利用した、Salesforce アカウントでのログイン

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

この記事では、[カスタム ポリシー](active-directory-b2c-overview-custom.md)を使用して特定の Salesforce 組織のユーザーのログインを有効にする方法について説明します。

## <a name="prerequisites"></a>前提条件

### <a name="azure-ad-b2c-setup"></a>Azure AD B2C のセットアップ

[カスタム ポリシーを使い始める](active-directory-b2c-get-started-custom.md)にあたっての手順がすべて完了していることを確認します。

次のトピックがあります。

1. Azure AD B2C テナントの作成。
1. Azure AD B2C アプリケーションの作成。
1. 2 つのポリシー エンジン アプリケーションの登録。
1. キーのセットアップ。
1. スターター パックのセットアップ。

### <a name="salesforce-setup"></a>Salesforce のセットアップ

このチュートリアルでは、既に以下が完了していることを前提としています。

1. Salesforce アカウントへのサインアップ。 [無料の Developer Edition](https://developer.salesforce.com/signup) にサインアップできます。
1. Salesforce 組織用の ["私のドメイン" のセットアップ](https://help.salesforce.com/articleView?id=domain_name_setup.htm&language=en_US&type=0)。

## <a name="configure-salesforce-to-allow-users-to-federate"></a>ユーザーがフェデレーションを実行できるように Salesforce を構成する

Azure AD B2C が Salesforce と通信できるようにするには、Salesforce のメタデータ URL を取得する必要があります。

### <a name="enable-salesforce-as-an-identity-provider"></a>Salesforce を ID プロバイダーとして有効にする

>[!NOTE]
> このチュートリアルでは、[Salesforce Lighting Experience](https://developer.salesforce.com/page/Lightning_Experience_FAQ) を使用していることを前提としています。

1. [Salesforce にログインします。](https://login.salesforce.com/) 
1. 左側のメニューで、**[Settings]\(設定\)** の下の **[Identity]\(ID\)** を展開し、**[Identity Provider]\(ID プロバイダー\)** をクリックします。
1. **[Enable Identity Provider]\(ID プロバイダーを有効にする\)** をクリックします。
1. Azure AD B2C と通信するときに Salesforce で使用する**証明書を選択**し、**[Save]\(保存\)** をクリックします。 既定の証明書を使用できます。

### <a name="create-a-connected-app-in-salesforce"></a>Salesforce での接続されたアプリの作成

1. **[Identity Provider]\(ID プロバイダー\)** ページで、**[Service Providers]\(サービス プロバイダー\)** セクションまでスクロールします。
1. **[Service Providers are now created via Connected Apps]\(接続されたアプリでサービス プロバイダーが作成されました。ここをクリックしてください。\)** をクリックします。
1. 接続されたアプリの必須の**基本情報**を提供します。
1. **[Web App Settings]\(Web アプリの設定\)** セクションで、**[Enable SAML]\(SAML を有効にする\)** をオンにします。
1. **[Entity ID]\(エンティティ ID\)** フィールドに次の URL を入力します。必ず `tenantName` を置き換えてください。
      ```
      https://login.microsoftonline.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase
      ```
1. **[ACS URL]** フィールドに次の URL を入力します。必ず `tenantName` を置き換えてください。
      ```
      https://login.microsoftonline.com/te/tenantName.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
      ```
1. それ以外の設定はすべて既定のままにします。
1. 一番下までスクロールし、**[Save]\(保存\)** ボタンをクリックします。

### <a name="get-the-metadata-url"></a>メタデータ URL の取得

1. 接続されたアプリの概要ページで **[Manage]\(管理\)** をクリックします。
1. **メタデータ検出エンドポイント**をコピーし、後で使用できるように保存します。

### <a name="enable-salesforce-users-to-federate"></a>Salesforce ユーザーがフェデレーションを実行できるようにする

1. 接続されたアプリの [Manage]\(管理\) ページで、**[Profiles]\(プロファイル\)** セクションまでスクロールします。
1. **[Manage Profiles]\(プロファイルを管理する\)** をクリックします。
1. Azure AD B2C とフェデレーションできるようにするプロファイル (またはユーザーのグループ) を選択します。 システム管理者は、Salesforce アカウントを使用してフェデレーションを行うことができるように、**[System Administrator]/(システム管理者/)** のチェック ボックスをオンにする必要があります。

## <a name="generate-a-signing-certificate-for-azure-ad-b2c"></a>Azure AD B2C の署名証明書の生成

Salesforce に送信された要求は、Azure AD B2C で署名する必要があります。 署名証明書を生成するには、PowerShell を開き、次のコマンドを実行します。

**先頭の 2 行のテナント名およびパスワードを必ず更新してください。**

```PowerShell
$tenantName = "<YOUR TENANT NAME>.onmicrosoft.com"
$pwdText = "<YOUR PASSWORD HERE>"

$Cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -DnsName "SamlIdp.$tenantName" -Subject "B2C SAML Signing Cert" -HashAlgorithm SHA256 -KeySpec Signature -KeyLength 2048

$pwd = ConvertTo-SecureString -String $pwdText -Force -AsPlainText

Export-PfxCertificate -Cert $Cert -FilePath .\B2CSigningCert.pfx -Password $pwd
```

## <a name="add-the-saml-signing-certificate-to-azure-ad-b2c"></a>Azure AD B2C への SAML 署名証明書の追加

Azure AD B2C テナントに署名証明書をアップロードする必要があります。 これを行うには、次の手順を実行します。

1. Azure AD B2C テナントに移動し、**[B2C Settings]\(B2C 設定\)、[Identity Experience Framework]、[Policy Keys]\(ポリシー キー\)** の順に開きます。
1. **[+ 追加]** をクリックします。
    * **[オプション]、[アップロード]** の順に選択します。
    * **名前** (例: `SAMLSigningCert`) を入力します。 プレフィックス B2C_1A_ がキーの名前に自動的に追加されます。
    * **ファイルのアップロード コントロール**を使用して証明書を選択する
    * PowerShell スクリプトで設定した証明書のパスワードを入力します。
1. ページの下部にある **[Create]**」を参照してください。
1. キー (例: `B2C_1A_SAMLSigningCert`) を作成したことを確認します。 フル ネーム (B2C_1A_ を含む) を書き留めておいてください。後でポリシーの中でこの名前を引用します。

## <a name="create-the-salesforce-saml-claims-provider-in-your-base-policy"></a>基本ポリシーでの Salesforce SAML 要求プロバイダーの作成

ユーザーが Salesforce を使用してログインできるようにするには、Salesforce を要求プロバイダーとして定義する必要があります。 つまり、Azure AD B2C が通信するエンドポイントを指定する必要があります。 エンドポイントは、特定のユーザーが認証されていることを確認するために Azure AD B2C で使用する一連の "*要求*" を "*提供*" します。 これを行うには、ポリシーの拡張ファイルで Salesforce の `<ClaimsProvider>` を追加します。

1. 作業ディレクトリ (TrustFrameworkExtensions.xml) から拡張ファイルを開きます。
1. セクション `<ClaimsProviders>` を探します。 存在しない場合は、ルート ノードの下に追加します。
1. 次のように新しい `<ClaimsProvider>` を追加します。

    ```XML
    <ClaimsProvider>
      <Domain>salesforce</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="salesforce">
          <DisplayName>Salesforce</DisplayName>
          <Description>Login with your Salesforce account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="RequestsSigned">false</Item>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="WantsSignedAssertions">false</Item>
            <Item Key="PartnerEntity">https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="socialIdpUserId" PartnerClaimType="userId"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="username"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="externalIdp"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="SAMLIdp" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

`<ClaimsProvider>` ノードの下で、以下の操作を実行します。

* `<Domain>` の値を、他の ID プロバイダーと区別するために使用できる一意の値に更新します。
* `<DisplayName>` の値を、要求プロバイダーを表すわかりやすい名前に更新します。 この値は現在使用されていません。

### <a name="update-the-technical-profile"></a>技術プロファイルの更新

Salesforce から SAML トークンを取得するためには、Azure AD B2C が Azure AD との通信に使用するプロトコルを定義する必要があります。 これは、`<ClaimsProvider>` の `<TechnicalProfile>` 要素内で行います。

1. `<TechnicalProfile>` ノードの ID を更新します。 この ID は、ポリシーの他の部分からこの技術プロファイルを参照するために使用します。
1. `<DisplayName>` の値を更新します。 この値は、ログイン画面のログイン ボタンに表示されます。
1. `<Description>` の値を更新します。
1. Salesforce では SAML 2.0 プロトコルが使用されるため、`<Protocol>` が "SAML2" であることを確認してください。

特定の Salesforce アカウントの構成設定を反映するように、上の XML の `<Metadata>` セクションを更新する必要があります。 XML で、次のようにメタデータ値を更新します。

1. `<Item Key="PartnerEntity">` の値を、先ほどコピーした Salesforce メタデータ URL で更新します。 形式は以下のとおりです。`https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp/connectedapp.xml`

1. 上記の XML の `<CryptographicKeys>` セクションで、両方の `StorageReferenceId` の値を署名証明書のキーの名前 (例: B2C_1A_SAMLSigningCert) に更新します。

### <a name="upload-the-extension-file-for-verification"></a>拡張ファイルのアップロードによる確認

ここまでで、Azure AD B2C が Salesforce と通信する方法を認識するようにポリシーを設定しました。 ポリシーの拡張ファイルをアップロードして、現時点で問題がないことを確認してみます。 そのためには、次の手順を実行します。

1. Azure AD B2C テナントの **[All Policies]**(すべてのポリシー) ブレードに移動します。
1. **[Overwrite the policy if it exists]**(ポリシーが存在する場合は上書き) チェック ボックスをオンにします。
1. 拡張ファイル (TrustFrameworkExtensions.xml) をアップロードし、検証に失敗しないことを確認します。

## <a name="register-the-salesforce-saml-claims-provider-to-a-user-journey"></a>ユーザー体験への Salesforce SAML 要求プロバイダーの登録

ここで、Salesforce SAML の ID プロバイダーをいずれかのユーザー体験に追加する必要があります。 この時点では、ID プロバイダーはセットアップされていますが、サインアップ/サインイン画面で使用することはできません。 これを行うには、既存のテンプレート ユーザー ジャーニーの複製を作成してから変更し、Azure AD の ID プロバイダーも含めるようにします。

1. ポリシーの基本ファイルを開きます (例: TrustFrameworkBase.xml)。
1. `<UserJourneys>` 要素を見つけて、Id ="SignUpOrSignIn" を含む `<UserJourney>` 全体をコピーします。
1. 拡張ファイル (例: TrustFrameworkExtensions.xml) を開き、`<UserJourneys>` 要素を見つけます。 要素が存在しない場合は追加します。
1. コピーした `<UserJourney>` 全体を `<UserJourneys>` 要素の子として貼り付けます。
1. 新しい `<UserJourney>` (つまり SignUpOrSignUsingContoso) の ID の名前を変更します。

### <a name="display-the-button"></a>"ボタン" の表示

`<ClaimsProviderSelection>` 要素は、サインアップ/サインイン画面の ID プロバイダーのボタンに類似しています。 Salesforce の `<ClaimsProviderSelection>` 要素を追加することで、ユーザーがページにアクセスしたときに新しいボタンが表示されます。 これを行うには、次の手順を実行します。

1. 作成した `<UserJourney>` で `Order="1"` になっている `<OrchestrationStep>` を見つけます。
1. 以下を追加します。

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

1. `TargetClaimsExchangeId` を適切な値に設定します。 他の場合と同じ規則に従うことをお勧めします - *\[ClaimProviderName\]Exchange*。

### <a name="link-the-button-to-an-action"></a>"ボタン" のアクションへのリンク

"ボタン" が所定の位置に配置されたので、ボタンをアクションにリンクする必要があります。 この場合のアクションでは、Azure AD B2C が Salesforce と通信して SAML トークンを受信します。 これを行うには、Salesforce SAML 要求プロバイダーの技術プロファイルをリンクします。

1. `<UserJourney>` ノードで `Order="2"` になっている `<OrchestrationStep>` を見つけます。
1. 以下を追加します。

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="ContosoProfile" />
    ```

1. `Id` を上の `TargetClaimsExchangeId` の値と同じ値に更新します。
1. `TechnicalProfileReferenceId` を、前に作成した技術プロファイルの `Id` に更新します (例: ContosoProfile)。

### <a name="upload-the-updated-extension-file"></a>更新済み拡張ファイルのアップロード

拡張ファイルの変更が終了しました。 このファイルを保存してアップロードし、すべての検証が正常に行われることを確認します。

### <a name="update-the-rp-file"></a>RP ファイルの更新

ここで、作成したユーザー ジャーニーを開始する RP ファイルを更新する必要があります。

1. 作業ディレクトリに SignUpOrSignIn.xml のコピーを作成してその名前を変更します (例: SignUpOrSignInWithAAD.xml)。
1. 新しいファイルを開き、`<TrustFrameworkPolicy>` の `PolicyId` 属性を一意の値で更新します。 これがポリシーの名前になります (例: SignUpOrSignInWithAAD)。
1. `<DefaultUserJourney>` の `ReferenceId` 属性を変更して、作成した新しいユーザー ジャーニーの ID と一致するようにします (例: SignUpOrSignUsingContoso)。
1. 変更内容を保存し、ファイルをアップロードします。

## <a name="testing-and-troubleshooting"></a>テストおよびトラブルシューティング

アップロードしたカスタム ポリシーのブレードを開いて [Run now] \(今すぐ実行) をクリックし、カスタム ポリシーをテストします。 エラーが発生した場合は、[トラブルシューティング](active-directory-b2c-troubleshoot-custom.md)の方法を参照してください。

## <a name="next-steps"></a>次のステップ

[AADB2CPreview@microsoft.com](mailto:AADB2CPreview@microsoft.com) にフィードバックを提供します。
