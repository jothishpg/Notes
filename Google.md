https://developers.google.com/identity/protocols/oauth2/web-server

Parameters
client_id	Required
The client ID for your application. You can find this value in the Cloud Console Clients page.

redirect_uri	Required
Determines where the API server redirects the user after the user completes the authorization flow. The value must exactly match one of the authorized redirect URIs for the OAuth 2.0 client, which you configured in your client's Cloud Console Clients page. If this value doesn't match an authorized redirect URI for the provided client_id you will get a redirect_uri_mismatch error.

Note that the http or https scheme, case, and trailing slash ('/') must all match.

response_type	Required
Determines whether the Google OAuth 2.0 endpoint returns an authorization code.

Set the parameter value to code for web server applications.

scope	Required
A space-delimited list of scopes that identify the resources that your application could access on the user's behalf. These values inform the consent screen that Google displays to the user.

Scopes enable your application to only request access to the resources that it needs while also enabling users to control the amount of access that they grant to your application. Thus, there is an inverse relationship between the number of scopes requested and the likelihood of obtaining user consent.

We recommend that your application request access to authorization scopes in context whenever possible. By requesting access to user data in context, using incremental authorization, you help users to understand why your application needs the access it is requesting.

access_type	Recommended
Indicates whether your application can refresh access tokens when the user is not present at the browser. Valid parameter values are online, which is the default value, and offline.

Set the value to offline if your application needs to refresh access tokens when the user is not present at the browser. This is the method of refreshing access tokens described later in this document. This value instructs the Google authorization server to return a refresh token and an access token the first time that your application exchanges an authorization code for tokens.

state	Recommended
Specifies any string value that your application uses to maintain state between your authorization request and the authorization server's response. The server returns the exact value that you send as a name=value pair in the URL query component (?) of the redirect_uri after the user consents to or denies your application's access request.

You can use this parameter for several purposes, such as directing the user to the correct resource in your application, sending nonces, and mitigating cross-site request forgery. Since your redirect_uri can be guessed, using a state value can increase your assurance that an incoming connection is the result of an authentication request. If you generate a random string or encode the hash of a cookie or another value that captures the client's state, you can validate the response to additionally ensure that the request and response originated in the same browser, providing protection against attacks such as cross-site request forgery. See the OpenID Connect documentation for an example of how to create and confirm a state token.

Important: The OAuth client must prevent CSRF as called out in the OAuth2 Specification . One way to achieve this is by using the state parameter to maintain state between your authorization request and the authorization server's response.
include_granted_scopes	Optional
Enables applications to use incremental authorization to request access to additional scopes in context. If you set this parameter's value to true and the authorization request is granted, then the new access token will also cover any scopes to which the user previously granted the application access. See the incremental authorization section for examples.

enable_granular_consent	Optional
Defaults to true. If set to false, more granular Google Account permissions will be disabled for OAuth client IDs created before 2019. No effect for newer OAuth client IDs, since more granular permissions is always enabled for them.

When Google enables granular permissions for an application, this parameter will no longer have any effect.

login_hint	Optional
If your application knows which user is trying to authenticate, it can use this parameter to provide a hint to the Google Authentication Server. The server uses the hint to simplify the login flow either by prefilling the email field in the sign-in form or by selecting the appropriate multi-login session.

Set the parameter value to an email address or sub identifier, which is equivalent to the user's Google ID.

prompt	Optional
A space-delimited, case-sensitive list of prompts to present the user. If you don't specify this parameter, the user will be prompted only the first time your project requests access. See Prompting re-consent for more information.

Possible values are:

none	Don't display any authentication or consent screens. Must not be specified with other values.
consent	Prompt the user for consent.
select_account	Prompt the user to select an account.
