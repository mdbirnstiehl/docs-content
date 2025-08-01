---
applies_to:
  deployment:
    self: ga
products:
  - id: kibana
---

# FIPS compliance for {{kib}} [fips-kibana]

To run {{kib}} in FIPS mode, you must have the appropriate [subscription](https://www.elastic.co/subscriptions).

::::{important}
The Node bundled with {{kib}} is not configured for FIPS 140-2. You must configure a FIPS 140-2 compliant OpenSSL3 provider. Consult the Node.js documentation to learn how to configure your environment.

::::


For {{kib}}, adherence to FIPS 140-2 is ensured by:

* Using FIPS approved / NIST recommended cryptographic algorithms.
* Delegating the implementation of these cryptographic algorithms to a NIST validated cryptographic module (available via Node.js configured with an OpenSSL3 provider).
* Allowing the configuration of {{kib}} in a FIPS 140-2 compliant manner, as documented below.

## Configuring {{kib}} for FIPS 140-2 [_configuring_kib_for_fips_140_2]

Apart from setting `xpack.security.fipsMode.enabled` to `true` in your {{kib}} config, a number of security related settings need to be reviewed and configured in order to run {{kib}} successfully in a FIPS 140-2 compliant Node.js environment.

### {{kib}} keystore [_kibana_keystore]

FIPS 140-2 (via NIST Special Publication 800-132) dictates that encryption keys should at least have an effective strength of 112 bits. As such, the {{kib}} keystore that stores the application’s secure settings needs to be password protected with a password that satisfies this requirement. This means that the password needs to be 14 bytes long which is equivalent to a 14 character ASCII encoded password, or a 7 character UTF-8 encoded password.

For more information on how to set this password, refer to the [keystore documentation](/deploy-manage/security/secure-settings.md#change-password).


### TLS keystore and keys [_tls_keystore_and_keys]

Keystores can be used in a number of General TLS settings in order to conveniently store key and trust material. PKCS#12 keystores cannot be used in a FIPS 140-2 compliant Node.js environment. Avoid using these types of keystores. Your FIPS 140-2 provider may provide a compliant keystore implementation that can be used, or you can use PEM encoded files. To use PEM encoded key material, you can use the relevant `\*.key` and `*.certificate` configuration options, and for trust material you can use `*.certificate_authorities`.

As an example, avoid PKCS#12 specific settings such as:

* `server.ssl.keystore.path`
* `server.ssl.truststore.path`
* `elasticsearch.ssl.keystore.path`
* `elasticsearch.ssl.truststore.path`