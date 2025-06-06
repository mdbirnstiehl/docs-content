ldap
:   Uses an external LDAP server to authenticate the users. This realm supports an authentication token in the form of username and password, and requires explicit configuration in order to be used. LDAP is not available on {{ech}} deployments. For more information, refer to [LDAP user authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/ldap.md).

active_directory
:   Uses an external Active Directory Server to authenticate the users. With this realm, users are authenticated by usernames and passwords. Active Directory is not available on {{ech}} deployments. For more information, refer to [Active Directory user authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/active-directory.md).

pki
:   Authenticates users using Public Key Infrastructure (PKI). This realm works in conjunction with SSL/TLS and identifies the users through the Distinguished Name (DN) of the client’s X.509 certificates. PKI is not available on {{ech}} deployments. For more information, refer to [PKI user authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/pki.md).

saml
:   Facilitates authentication using the SAML 2.0 Web SSO protocol. This realm is designed to support authentication through {{kib}} and is not intended for use in the REST API. For more information, refer to [SAML authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/saml.md).

kerberos
:   Authenticates a user using Kerberos authentication. Users are authenticated on the basis of Kerberos tickets. For more information, refer to [Kerberos authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/kerberos.md).

oidc
:   Facilitates authentication using OpenID Connect. It enables {{es}} to serve as an OpenID Connect Relying Party (RP) and provide single sign-on (SSO) support in {{kib}}. For more information, refer to [Configuring single sign-on to the {{stack}} using OpenID Connect](/deploy-manage/users-roles/cluster-or-deployment-auth/openid-connect.md).

jwt
:   Facilitates using JWT identity tokens as authentication bearer tokens. Compatible tokens are OpenID Connect ID Tokens, or custom JWTs containing the same claims. For more information, refer to [JWT authentication](/deploy-manage/users-roles/cluster-or-deployment-auth/jwt.md).