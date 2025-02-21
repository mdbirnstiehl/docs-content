---
mapped_pages:
  - https://www.elastic.co/guide/en/elasticsearch/reference/current/realms.html
---

# Authentication realms [realms]

The {{stack-security-features}} authenticate users by using realms and one or more [token-based authentication services](token-based-authentication-services.md).

A *realm* is used to resolve and authenticate users based on authentication tokens. The {{security-features}} provide the following built-in realms:

*native*
:   An internal realm where users are stored in a dedicated {{es}} index. This realm supports an authentication token in the form of username and password, and is available by default when no realms are explicitly configured. The users are managed via the [user management APIs](https://www.elastic.co/docs/api/doc/elasticsearch/group/endpoint-security). See [Native user authentication](native.md).

*ldap*
:   A realm that uses an external LDAP server to authenticate the users. This realm supports an authentication token in the form of username and password, and requires explicit configuration in order to be used. See [LDAP user authentication](ldap.md).

*active_directory*
:   A realm that uses an external Active Directory Server to authenticate the users. With this realm, users are authenticated by usernames and passwords. See [Active Directory user authentication](active-directory.md).

*pki*
:   A realm that authenticates users using Public Key Infrastructure (PKI). This realm works in conjunction with SSL/TLS and identifies the users through the Distinguished Name (DN) of the client’s X.509 certificates. See [PKI user authentication](pki.md).

*file*
:   An internal realm where users are defined in files stored on each node in the {{es}} cluster. This realm supports an authentication token in the form of username and password and is always available. See [File-based user authentication](file-based.md).

*saml*
:   A realm that facilitates authentication using the SAML 2.0 Web SSO protocol. This realm is designed to support authentication through {{kib}} and is not intended for use in the REST API. See [SAML authentication](saml.md).

*kerberos*
:   A realm that authenticates a user using Kerberos authentication. Users are authenticated on the basis of Kerberos tickets. See [Kerberos authentication](kerberos.md).

*oidc*
:   A realm that facilitates authentication using OpenID Connect. It enables {{es}} to serve as an OpenID Connect Relying Party (RP) and provide single sign-on (SSO) support in {{kib}}. See [Configuring single sign-on to the {{stack}} using OpenID Connect](openid-connect.md).

*jwt*
:   A realm that facilitates using JWT identity tokens as authentication bearer tokens. Compatible tokens are OpenID Connect ID Tokens, or custom JWTs containing the same claims. See [JWT authentication](jwt.md).

The {{security-features}} also support custom realms. If you need to integrate with another authentication system, you can build a custom realm plugin. For more information, see [Integrating with other authentication systems](custom.md).

## Internal and external realms [_internal_and_external_realms]

Realm types can roughly be classified in two categories:

Internal
:   Realms that are internal to Elasticsearch and don’t require any communication with external parties. They are fully managed by the {{stack}} {{security-features}}. There can only be a maximum of one configured realm per internal realm type. The {{security-features}} provide two internal realm types: `native` and `file`.

External
:   Realms that require interaction with parties/components external to {{es}}, typically, with enterprise grade identity management systems. Unlike internal realms, there can be as many external realms as one would like - each with its own unique name and configuration. The {{security-features}} provide the following external realm types: `ldap`, `active_directory`, `saml`, `kerberos`, and `pki`.


