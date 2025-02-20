# OpenID Connect authentication [oidc-realm]

The OpenID Connect realm enables {{es}} to serve as an OpenID Connect Relying Party (RP) and provides single sign-on (SSO) support in {{kib}}.

It is specifically designed to support authentication via an interactive web browser, so it does not operate as a standard authentication realm. Instead, there are {{kib}} and {{es}} {{security-features}} that work together to enable interactive OpenID Connect sessions.

This means that the OpenID Connect realm is not suitable for use by standard REST clients. If you configure an OpenID Connect realm for use in {{kib}}, you should also configure another realm, such as the [native realm](../../../deploy-manage/users-roles/cluster-or-deployment-auth/native.md) in your authentication chain.

In order to simplify the process of configuring OpenID Connect authentication within the {{stack}}, there is a step-by-step guide: [Configuring single sign-on to the {{stack}} using OpenID Connect](../../../deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md).

