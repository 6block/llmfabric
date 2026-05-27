# Single Sign-On (SSO) Authentication

{{ brand.name }} supports Single Sign-On (SSO) authentication methods such as OIDC and SAML. This allows users to log in using their existing credentials from an external identity provider.

## OIDC

Any authentication provider that supports OIDC can be configured. The `email`, `name` and `picture` claims are used if available. The allowed redirect URI should include `<server-url>/auth/oidc/callback`.

If your OIDC provider uses a certificate issued by a private or corporate CA, see [Additional Trusted CAs](../installation/installation.md#additional-trusted-cas) for how to mount CA certificates into the {{ brand.name }} container.

The following CLI flags are available for OIDC configuration:

| <div style="width:180px">Flag</div>                   | Description                                                                                                                                                          |
|-------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--oidc-issuer`                                       | OIDC issuer URL. OIDC discovery under `<issuer>/.well-known/openid-configuration` will be used to discover the OIDC configuration.                                   |
| `--oidc-client-id`                                    | OIDC client ID.                                                                                                                                                      |
| `--oidc-client-secret`                                | OIDC client secret.                                                                                                                                                  |
| `--oidc-redirect-uri`                                 | The redirect URI configured in your OIDC application. This must be set to `<server-url>/auth/oidc/callback`.                                                         |
| `--external-auth-name` (Optional)                     | Mapping of OIDC user information to username, e.g., `preferred_username`. By default, the `email` claim is used if available.                                        |
| `--external-auth-full-name` (Optional)                | Mapping of OIDC user information to user's full name. Multiple elements can be combined, e.g., `name` or `firstName+lastName`. By default, the `name` claim is used. |
| `--external-auth-avatar-url` (Optional)               | Mapping of OIDC user information to user's avatar URL. By default, the `picture` claim is used if available.                                                         |
| `--external-auth-default-inactive` (Optional)         | Prevents new SSO users from being activated by default.                                                                                                              |
| `--external-auth-post-logout-redirect-key` (Optional) | Generic parameter name for post-logout redirection across different IdPs (e.g., Auth0 `returnTo`). Applied to both OIDC and SAML.                                    |

You can also set these options via environment variables instead of CLI flags:

```bash
{{ brand.env_prefix }}_OIDC_ISSUER="your-oidc-issuer-url"
{{ brand.env_prefix }}_OIDC_CLIENT_ID="your-client-id"
{{ brand.env_prefix }}_OIDC_CLIENT_SECRET="your-client-secret"
{{ brand.env_prefix }}_OIDC_REDIRECT_URI="{your-server-url}/auth/oidc/callback"
# Optional
{{ brand.env_prefix }}_EXTERNAL_AUTH_NAME="email"
{{ brand.env_prefix }}_EXTERNAL_AUTH_FULL_NAME="name"
{{ brand.env_prefix }}_EXTERNAL_AUTH_AVATAR_URL="picture"
{{ brand.env_prefix }}_EXTERNAL_AUTH_DEFAULT_INACTIVE="true"
{{ brand.env_prefix }}_EXTERNAL_AUTH_POST_LOGOUT_REDIRECT_KEY="returnTo"  # e.g., for Auth0
```

### Example: Integrate with Auth0 OIDC

To configure {{ brand.name }} with Auth0 as the OIDC provider:

1. Go to [auth0](https://auth0.com) and create a new application with type `Regular Web Applications`.

![create-oidc-app](../assets/sso/create-oidc-app.png)

2. Get the `Domain`, `Client ID`, and `Client Secret` from the application settings.

![auth0-app](../assets/sso/auth0-app.png)

3. Add `<your-server-url>/auth/oidc/callback` in the Allowed Callback URLs. Adapt the URL to match your server's URL.

4. In Allowed Logout URLs, add `<your-server-url>/` (or your desired post-logout URL).

![auth0-callback](../assets/sso/auth0-callback.png)

Then, run {{ brand.name }} with relevant OIDC configuration. The following example uses Docker with CUDA:

```bash
sudo docker run -d --name {{ brand.executable_name }} \
    --restart=unless-stopped \
    --privileged \
    --network=host \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
    --volume /path/to/custom-root-ca.crt:/usr/local/share/ca-certificates/custom-root-ca.crt:ro \
    --runtime nvidia \
    -e {{ brand.env_prefix }}_OIDC_ISSUER="https://<your-auth0-domain>" \
    -e {{ brand.env_prefix }}_OIDC_CLIENT_ID="<your-client-id>" \
    -e {{ brand.env_prefix }}_OIDC_CLIENT_SECRET="<your-client-secret>" \
    -e {{ brand.env_prefix }}_OIDC_REDIRECT_URI="<your-server-url>/auth/oidc/callback" \
    {{ brand.docker_image }}
```

!!! note

    The custom CA certificate mount is only required when your OIDC provider is signed by a private CA. Public OIDC providers such as Auth0 typically do not require it.

## SAML

{{ brand.name }} supports SAML authentication for Single Sign-On (SSO). This allows users to log in using their existing credentials from an external identity provider that supports SAML.

The following CLI flags are available for SAML configuration:

| <div style="width:180px">Flag</div>           | Description                                                                                                                                                                                                                                                    |
|-----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `--saml-idp-server-url`                       | SAML Identity Provider server URL.                                                                                                                                                                                                                             |
| `--saml-idp-entity-id`                        | SAML Identity Provider entity ID.                                                                                                                                                                                                                              |
| `--saml-idp-x509-cert`                        | SAML Identity Provider X.509 certificate.                                                                                                                                                                                                                      |
| `--saml-sp-entity-id`                         | SAML Service Provider entity ID.                                                                                                                                                                                                                               |
| `--saml-sp-acs-url`                           | SAML Service Provider Assertion Consumer Service URL. It should be set to `<{{ brand.executable_name }}-server-url>/auth/saml/callback`.                                                                                                                                          |
| `--saml-sp-x509-cert`                         | SAML Service Provider X.509 certificate.                                                                                                                                                                                                                       |
| `--saml-sp-private-key`                       | SAML Service Provider private key.                                                                                                                                                                                                                             |
| `--saml-idp-logout-url` (Optional)            | SAML Identity Provider Single Logout endpoint URL.                                                                                                                                                                                                             |
| `--saml-sp-slo-url` (Optional)                | SAML Service Provider Single Logout Service callback URL (e.g., `<server-url>/auth/saml/logout/callback`).                                                                                                                                                     |
| `--saml-sp-attribute-prefix` (Optional)       | SAML Service Provider attribute prefix, which is used for fetching the attributes that are specified by --external-auth-\*. e.g., 'http://schemas.auth0.com/'.                                                                                                 |
| `--saml-security` (Optional)                  | SAML security settings in JSON format.                                                                                                                                                                                                                         |
| `--external-auth-name` (Optional)             | Mapping of SAML user information to username. You must configure the full attribute name like 'http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress' or simplify with 'emailaddress' by '--saml-sp-attribute-prefix'.                            |
| `--external-auth-full-name` (Optional)        | Mapping of SAML user information to user's full name. Multiple elements can be combined. You must configure the full attribute name like 'http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name' or simplify with 'name' by '--saml-sp-attribute-prefix'. |
| `--external-auth-avatar-url` (Optional)       | Mapping of SAML user information to user's avatar URL. You must configure the full attribute name like 'http://schemas.auth0.com/picture' or simplify with 'picture' by '--saml-sp-attribute-prefix'.                                                          |
| `--external-auth-default-inactive` (Optional) | Prevents new SSO users from being activated by default.                                                                                                                                                                                                        |

You can also set these options via environment variables instead of CLI flags:

```bash
{{ brand.env_prefix }}_SAML_IDP_SERVER_URL="https://idp.example.com"
{{ brand.env_prefix }}_SAML_IDP_ENTITY_ID="your-idp-entity-id"
{{ brand.env_prefix }}_SAML_IDP_X509_CERT="your-idp-x509-cert"
{{ brand.env_prefix }}_SAML_SP_ENTITY_ID="your-sp-entity-id"
{{ brand.env_prefix }}_SAML_SP_ACS_URL="{your-server-url}/auth/saml/callback"
{{ brand.env_prefix }}_SAML_SP_X509_CERT="your-sp-x509-cert"
{{ brand.env_prefix }}_SAML_SP_PRIVATE_KEY="your-sp-private-key"
# Optional
{{ brand.env_prefix }}_SAML_SP_ATTRIBUTE_PREFIX="http://schemas.auth0.com/"
{{ brand.env_prefix }}_SAML_SECURITY="{}"
{{ brand.env_prefix }}_EXTERNAL_AUTH_NAME="emailaddress"
{{ brand.env_prefix }}_EXTERNAL_AUTH_FULL_NAME="name"
{{ brand.env_prefix }}_EXTERNAL_AUTH_AVATAR_URL="picture"
{{ brand.env_prefix }}_EXTERNAL_AUTH_DEFAULT_INACTIVE="true"
{{ brand.env_prefix }}_SAML_IDP_LOGOUT_URL="https://idp.example.com/saml/slo"  # if IdP supports SLO
{{ brand.env_prefix }}_SAML_SP_SLO_URL="{your-server-url}/auth/saml/logout/callback"
{{ brand.env_prefix }}_EXTERNAL_AUTH_POST_LOGOUT_REDIRECT_KEY="returnTo"  # optional, adds a post-logout parameter for compatible IdPs
```

### Example: Integrate with Auth0 SAML

To configure {{ brand.name }} with Auth0 as the SAML provider:

1. Go to [auth0](https://auth0.com) and create a new application with type `Regular Web Applications`.

![create-saml-app](../assets/sso/create-saml-app.png)

2. Get the `Domain` from the application settings and add `<your-server-url>/auth/saml/callback` in the Allowed Callback URLs. Adapt the URL to match your server's URL.

![auth0-saml-callback](../assets/sso/auth0-saml-callback.png)

3. In **Advanced Settings → Certificates**, copy the IdP `X.509 Certificate`.

![auth0-saml-cert](../assets/sso/auth0-saml-cert.png)

4. In **Endpoints** tab, find the `SAML Protocol URL`, which is your IdP server URL.

![auth0-saml-url](../assets/sso/auth0-saml-url.png)

5. Generate SP certificate and private key:

```bash
openssl req -x509 -newkey rsa:2048 -keyout myservice.key -out myservice.cert -days 365 -nodes -subj "/CN=myservice.example.com"
```

!!! note

    myservice.cert and myservice.key will be used for the SP configuration.

6. Run {{ brand.name }} with relevant SAML configuration. The following example uses Docker with CUDA:

```bash
SP_CERT="$(cat myservice.cert)"
SP_PRIVATE_KEY="$(cat myservice.key)"
SP_ATTRIBUTE_PREFIX="http://schemas.auth0.com/"

sudo docker run -d --name {{ brand.executable_name }} \
    --restart=unless-stopped \
    --privileged \
    --network=host \
    --volume /var/run/docker.sock:/var/run/docker.sock \
    --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
    --runtime nvidia \
    -e {{ brand.env_prefix }}_SAML_IDP_SERVER_URL="<auth0-saml-protocol-url>" \
    -e {{ brand.env_prefix }}_SAML_IDP_ENTITY_ID="urn:<auth0-domain>" \
    -e {{ brand.env_prefix }}_SAML_IDP_X509_CERT="<auth0-x509-cert>" \
    -e {{ brand.env_prefix }}_SAML_SP_ENTITY_ID="urn:{{ brand.executable_name }}:sp" \
    -e {{ brand.env_prefix }}_SAML_SP_ACS_URL="<your-{{ brand.executable_name }}-server-url>/auth/saml/callback" \
    -e {{ brand.env_prefix }}_SAML_SP_X509_CERT="$SP_CERT" \
    -e {{ brand.env_prefix }}_SAML_SP_PRIVATE_KEY="$SP_PRIVATE_KEY" \
    -e {{ brand.env_prefix }}_SAML_SP_ATTRIBUTE_PREFIX="$SP_ATTRIBUTE_PREFIX" \
    -e {{ brand.env_prefix }}_SAML_IDP_LOGOUT_URL="<idp-slo-url-if-available>" \
    -e {{ brand.env_prefix }}_SAML_SP_SLO_URL="<your-{{ brand.executable_name }}-server-url>/auth/saml/logout/callback" \
    -e {{ brand.env_prefix }}_EXTERNAL_AUTH_POST_LOGOUT_REDIRECT_KEY="returnTo" \
    {{ brand.docker_image }}
```

!!! note

    Not all IdPs provide standard SAML Single Logout (SLO). Auth0 SAML connections commonly do not expose `singleLogoutService`. If unavailable, {{ brand.name }} will still clear local sessions on logout; for full browser sign-out with Auth0, consider using its OIDC `v2/logout` with `client_id` and `returnTo` allowed.
