Servlet                                      Jersey (JAX-RS)

Tomcat maps URL → Servlet.	                 Tomcat maps URL → Jersey Servlet → Resource Method.
You extend HttpServlet.                      You write POJO classes with annotations.
You override doGet(), doPost().              You annotate methods with @GET, @POST.

Listener Interfaces:
Interface												What it listens for
ServletContextListener									Application start/stop
ServletContextAttributeListener							Context attribute changes
ServletRequestListener									Request creation/destruction
ServletRequestAttributeListener							Request attribute changes
HttpSessionListener										Session creation/destruction
HttpSessionAttributeListener							Session attribute changes
HttpSessionBindingListener								Object being bound/unbound to a session
HttpSessionActivationListener							Session activation/passivation

Annotation - Purpose

@Path-Maps URL path.
@GET-Handles GET request.
@POST-Handles POST request.
@PUT-Handles PUT request.
@DELETE-andles DELETE request.
@Produces-What response type is returned.
@Consumes-What request type is accepted.
@PathParam-Reads path variables.
@QueryParam-Reads query parameters.
@HeaderParam-Reads HTTP headers.
@FormParam-Reads HTML form fields.
@CookieParam-Reads cookies.
@Context-Injects servlet-related objects.


ObjectMapper

This is the most important Jackson class you'll encounter initially.
You can manually use it like:

ObjectMapper mapper = new ObjectMapper();

Then:

Student student = mapper.readValue(json, Student.class);

means:
JSON
 ↓
readValue()
 ↓
Student object

And:
String json = mapper.writeValueAsString(student);

means:
Student object
 ↓
writeValueAsString()
 ↓
JSON


                 TOMCAT
                   │
             web.xml
                   │
       "Which servlet handles /api/*?"
                   │
                   ↓
        Jersey ServletContainer
                   │
                   │
          ParkingApplication
                   │
        "Which REST resources?"
                   │
                   ↓
             AuthResource

web.xml
   ↓
Tomcat knows /api/* is Jersey
   ↓
ParkingApplication
   ↓
Jersey scans org.example.auth
   ↓
AuthResource
   ↓
@Path("auth")
   ↓
@Path("login")
   ↓
POST /parking-app/api/auth/login


Database Connection

Your Java/Jersey application
        │
        │ JDBC
        ▼
PostgreSQL JDBC Driver
        │
        │ TCP connection
        ▼
PostgreSQL Server
        │
        ▼
parking_db database

For ur current setup

Java application
      │
      │ DriverManager.getConnection(...)
      ▼
PostgreSQL JDBC Driver
      │
      │ 127.0.0.1:5432
      ▼
PostgreSQL Server
      │
      ▼
parking_db

Let's break your URL apart - jdbc:postgresql://127.0.0.1:5432/parking_db

jdbc

Means:
I'm using Java Database Connectivity.
JDBC is Java's standard API for communicating with relational databases.

postgresql

This tells JDBC:
This connection is for PostgreSQL.
Different databases have different JDBC drivers.

For example:
PostgreSQL → PostgreSQL JDBC driver
MySQL      → MySQL JDBC driver
Oracle     → Oracle JDBC driver

127.0.0.1

This means:
The database server is running on this same computer.
It's the loopback address.

Equivalent to:
localhost
So:
127.0.0.1
and:
localhost
normally point to your own computer.

5432

This is PostgreSQL's default network port.
Think of an IP address as identifying the computer and the port as identifying the service on that computer.
127.0.0.1 : 5432
     │        │
     │        └── PostgreSQL service
     └─────────── your computer
     
parking_db

This is the specific PostgreSQL database you want to connect to.
You might have:
PostgreSQL Server
│
├── parking_db
├── postgres
├── another_database
└── ...

Connection()?

What is Connection?

It's a JDBC object representing an active connection/session between your Java application and PostgreSQL, In the context of JDBC, Connection specifically means a connection between your Java application and a database server

JDBC object	                 Used for
Statement	Simple,            fixed SQL
PreparedStatement	           SQL with parameters
CallableStatement	           Calling database procedures/functions

Why do we need the PostgreSQL JDBC Driver?

Your Java application doesn't directly know how to communicate with PostgreSQL.
Think of the architecture:

Your Java code
      ↓
JDBC API
      ↓
PostgreSQL JDBC Driver
      ↓
PostgreSQL Server

java.sql.Connection, DriverManager, PreparedStatement, etc. are part of the JDBC API.
But Java needs a PostgreSQL-specific driver to actually communicate with PostgreSQL.

DriverManager is part of Java JDBC:

java.sql.DriverManager

Its job is roughly:
Find an appropriate JDBC driver and ask that driver to establish the database connection.

Conceptually:
Your code
   │
   │ DriverManager.getConnection()
   ▼
DriverManager
   │
   │ "I need PostgreSQL"
   ▼
PostgreSQL JDBC Driver
   │
   │ network connection
   ▼
PostgreSQL Server

PostgreSQL
     │
     │ connection established
     ▼
PostgreSQL JDBC Driver
     │
     ▼
DriverManager
     │
     ▼
Connection object

ResultSet:

ResultSet is a JDBC object that holds the rows returned by a SQL SELECT query.
PostgreSQL returns the data, and JDBC gives that returned data to your Java program through a ResultSet

Filter :

@Provider tells Jersey:

"This class is a JAX-RS provider."
A provider is a component that extends or modifies Jersey's request/response processing.
For example, providers can be:

Request filters
Response filters
Exception mappers
Entity readers
Entity writers
Other JAX-RS extension components

So it's not merely a generic "register this class" annotation.

@Path tells Jersey:
"This is a JAX-RS resource."
@Provider tells Jersey:
"This is a JAX-RS provider."

Main difference
	Environment variable	                       System property
Belongs to	OS/process environment	              Java application
Read using	System.getenv()	                      System.getProperty()
Example	GOOGLE_CLIENT_ID=abc	                -DGOOGLE_CLIENT_ID=abc
Set by	Windows, Linux, Docker,server, etc.	    Java command line/application
Available to Processes that receive the           That Java process/JVM
environment	

CLASS
 ↓
General-purpose object
 ↓
Can contain mutable data + lots of behavior


RECORD
 ↓
Data-carrying object
 ↓
Compact syntax
 ↓
Values are final
 ↓
Java generates common methods

Logger:
private static final Logger LOGGER =
        Logger.getLogger(AuthResource.class.getName());

This creates a Java logger for your AuthResource.
Its purpose is to record information about what your application is doing.

Http_transport:

private static final NetHttpTransport HTTP_TRANSPORT =
        new NetHttpTransport();

This is related to making HTTP requests from your Java application to external HTTP services.
NetHttpTransport is from Google's HTTP client library.
Your application might need this when communicating with Google during OAuth.

JSON_FACTORY:

private static final GsonFactory JSON_FACTORY =
        GsonFactory.getDefaultInstance();

This is used for JSON processing by Google's client libraries.
GsonFactory is based on Google's Gson JSON library.
The JSON_FACTORY provides the JSON parser/serializer infrastructure that Google's OAuth/client classes need.

whay can't we jackson here??

Yes, you can use Jackson in your application, but the important point is that JSON_FACTORY is not there simply because your application needs JSON. It is there because the Google library you're using expects a specific JSON factory.

GsonFactory.getDefaultInstance()

which gives you the standard/default Gson factory.
Again, you don't need to create a new one for every request.

Secure_Random:

private static final SecureRandom RANDOM =
        new SecureRandom();

This one is particularly important for authentication/security.
SecureRandom generates cryptographically strong random values.

ResultSet:

ResultSet represents the rows returned by the database.

if (!result.next()
        || !password.equals(result.getString("password"))) {

This is extremely important.
Initially the ResultSet cursor is before the first row.

ResultSet

   cursor
     ↓
   ┌─────────┐
   │ Row 1   │
   ├─────────┤
   │ Row 2   │
   └─────────┘

Google Authentication:

                YOUR APPLICATION
                    |
                    | 1. Click "Sign in with Google"
                    ↓
              /api/auth/google/start
                    |
                    | 2. Generate state
                    | 3. Save state in session
                    | 4. Build Google URL
                    |
                    | 5. Redirect browser
                    ↓
              GOOGLE LOGIN PAGE
                    |
                    | User selects Google account
                    | User gives consent
                    ↓
              Google authenticates
                    |
                    | 6. Google redirects browser
                    ↓
        /api/auth/google/callback?code=...&state=...
                    |
                    | 7. Backend verifies state
                    | 8. Exchanges code for tokens
                    | 9. Verifies Google identity
                    ↓
                YOUR DATABASE
                    |
                    | 10. Find/create user
                    ↓
              Create application session
                    |
                    ↓
              Driver/Admin page

State:

String state = newGoogleState();
This creates a random value.

The Google callback I'm receiving belongs to the login request that MY application started."
This protects the OAuth flow against CSRF/login-request injection attacks.
Think of state as a temporary secret ticket.

https://accounts.google.com/o/oauth2/v2/auth"
                        + "?client_id=" + encode(GOOGLE_CLIENT_ID)
                        + "&redirect_uri=" + encode(googleRedirectUri(request))
                        + "&response_type=code"
                        + "&scope=" + encode("openid email profile")
                        + "&state=" + encode(state)
                        + "&prompt=select_account"

client_id:
client_id=YOUR_CLIENT_ID

This identifies your application to Google.
When you created your OAuth client in Google Cloud, Google gave you something like:
123456789012-abcdefg123.apps.googleusercontent.com

Redirect Url:

redirect_uri=YOUR_CALLBACK
This tells Google:
"After the user finishes authentication, where should you send the browser?"
For example, suppose your backend has:
http://localhost:8080/parking-app/api/auth/google/callbacb

response_type:

response_type=code
This is very important.
You're telling Google:
"After authentication, give me an authorization code."

Why use code?
Because this is the Authorization Code flow.
The important idea is:

Browser
   │
   │ authorization request
   ▼
Google
   │
   │ authorization code
   ▼
Browser → Your backend
              │
              │ exchange code
              ▼
           Google
              │
              │ tokens
              ▼
           Backend

scope:

scope=openid%20email%20profile

This tells Google:
"What information/permissions does my application want?"
There are three scopes here:

openid
email
profile

The %20 is simply URL encoding for a space.
So:
openid%20email%20profile

open_id tells Google:

"I am not just asking for permission to access something. I want Google to authenticate this person and give my application verified identity information."

OAuth 2.0 was designed primarily around authorization:
"Can this application access something on behalf of this user?"

OpenID Connect builds an authentication/identity layer on top of OAuth 2.0:
"Who is this user?"

The important part is:

openid

Google recognizes:
"This application is requesting OpenID Connect authentication."

So Google can return an ID token as part of the OpenID Connect flow.

Your Application
       │
       │ scope=openid
       ▼
     Google
       │
       │ Authenticate user
       ▼
User signs in
       │
       ▼
Google creates ID Token
       │
       ▼
Your Application

prompt:

prompt=select_account
This tells Google how you want the account-selection experience to behave.
select_account essentially asks Google to let the user choose a Google account

GoogleTokenResponse tokenResponse =
                    new GoogleAuthorizationCodeTokenRequest(
                            HTTP_TRANSPORT,
                            JSON_FACTORY,
                            GOOGLE_CLIENT_ID,
                            GOOGLE_CLIENT_SECRET,
                            code,
                            googleRedirectUri(request)
                    ).execute();

GoogleIdToken token =
          googleVerifier().verify(tokenResponse.getIdToken());

What does .execute() mean?
This is very important.

Before:
new GoogleAuthorizationCodeTokenRequest(...)
you are essentially building the request.

When you call:
.execute();
the request actually happens.

Create request object
        ↓
execute()
        ↓
HTTP request sent to Google
        ↓
Google processes it
        ↓
Google sends response
        ↓
GoogleTokenResponse

Token
 ↓
googleVerifier()
 ↓
Verify signature/claims
 ↓
Valid?

GoogleIdToken.Payload payload = token.getPayload();

Think of it like:

Verified Google ID Token
          ↓
       Payload
          ↓
identity information

Now your application can read the user's information.

GeneralSecurityException
Can occur during security-related operations such as token verification.

IOException
Can occur during network communication / reading responses.

First, what is the ID token?

After the user logs into Google, your backend exchanges the authorization code for tokens.
Google gives something conceptually like:

ID Token
   ↓
eyJhbGciOiJSUzI1NiIs...

An ID token is a JWT (JSON Web Token).
Conceptually it contains three parts:

HEADER.PAYLOAD.SIGNATURE

For example:
eyJhbGciOiJSUzI1NiJ9
.
eyJzdWIiOiIxMjM0NSIsImVtYWlsIjoi...
.
ABC123XYZ...

Don't think of this as simply an encoded string. The important part is that the token is signed by Google.

Google
   │
   │ private key
   ↓
CREATE SIGNATURE
   │
   ↓
HEADER.PAYLOAD.SIGNATURE
   │
   │
   ↓
Your Backend
   │
   │ Google's public key
   ↓
VERIFY SIGNATURE

Token Creation:

First, Google decides what information goes into the token
Suppose the user logs in with Google.

Google knows information such as:

{
  "sub": "123456789",
  "email": "user@gmail.com",
  "name": "Dinesh",
  "aud": "MY_GOOGLE_CLIENT_ID"
}

Google creates claims such as these for the payload.
For example:

{
  "iss": "https://accounts.google.com",
  "sub": "123456789",
  "email": "user@gmail.com",
  "aud": "MY_GOOGLE_CLIENT_ID",
  "exp": 1790000000
}

This is the payload JSON.

2. Google creates the Header
Google needs to tell the verifier information about how the token was signed.
For example:

{
  "alg": "RS256",
  "typ": "JWT"
}

Meaning:
alg = RS256
      ↓
algorithm used for the signature

typ = JWT
      ↓
this is a JWT
This is the header JSON.

3. Header and payload are encoded
The JSON itself isn't directly placed into the final token.
Google first converts the JSON into bytes and then uses Base64URL encoding.
For example:

Header JSON
   ↓
Base64URL encode
   ↓
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9

Similarly:

Payload JSON
   ↓
Base64URL encode
   ↓
eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJzdWIiOiIxMj...

So now Google has:

ENCODED_HEADER
.
ENCODED_PAYLOAD

4. Google creates the string that will be signed
Google joins the two encoded parts with a .:

encodedHeader + "." + encodedPayload
For example:

eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20i...

This entire string is called the signing input.

HEADER_ENCODED.PAYLOAD_ENCODED
              ↑
         signing input

5. Google creates the signature
This is the most important part.
Google has a private key.
It takes:

Signing Input
     +
Google's private key
     +
RS256 algorithm
     ↓
SIGNATURE

Conceptually:

SIGNATURE =
RS256(
    Base64URL(Header) + "." + Base64URL(Payload),
    Google_Private_Key
)
The actual cryptographic operation is more specific than this simplified notation, but this is the right mental model.
The important point is:
The signature is created using Google's private key.

6. Google Base64URL-encodes the signature
The binary signature is also Base64URL encoded.
For example:

Binary signature
       ↓
Base64URL
       ↓
SflKxwRJSMeKKF2QT4fwpMeJf36POk6y...

Now we have all three pieces:

ENCODED HEADER
.
ENCODED PAYLOAD
.
ENCODED SIGNATURE

7. Google joins all three
Finally:
HEADER.PAYLOAD.SIGNATURE

For example:
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJzdWIiOiIxMj...
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6y...

That entire string is the ID token.

8. Now your backend receives this token
Your backend gets something like:

HEADER.PAYLOAD.SIGNATURE

Your code does:
googleVerifier().verify(idToken);

The verifier then performs checks.
Very roughly:

                ID TOKEN
                   ↓
        ┌──────────┴──────────┐
        ↓                     ↓
     Header                Payload
        ↓                     ↓
     alg=RS256             aud=YOUR_CLIENT_ID
                           iss=Google
                           exp=...
        │                     │
        └──────────┬──────────┘
                   ↓
             Verify Signature
                   ↓
       Google's public key
                   ↓
             Valid or invalid

Why public key?
Google created the signature using:

Google PRIVATE KEY

Your backend verifies it using the corresponding:

Google PUBLIC KEY

So:

Google
   │
   │ private key
   ↓
CREATE SIGNATURE
   │
   ↓
HEADER.PAYLOAD.SIGNATURE
   │
   │
   ↓
Your Backend
   │
   │ Google's public key
   ↓
VERIFY SIGNATURE

At a high level, the verifier checks important properties of the ID token, including things such as:

1. Is the token structurally valid?
2. Is its signature valid?
3. Was it issued by the expected Google issuer?
4. Is it intended for the configured audience/client ID?
5. Is it otherwise acceptable according to the library's token validation rules?

In your code:
GoogleTokenResponse tokenResponse =
        new GoogleAuthorizationCodeTokenRequest(
                HTTP_TRANSPORT,
                JSON_FACTORY,
                GOOGLE_CLIENT_ID,
                GOOGLE_CLIENT_SECRET,
                code,
                googleRedirectUri(request)
        ).execute();

Google's token endpoint returns something conceptually like:

{
  "access_token": "ya29.a0....",
  "expires_in": 3599,
  "refresh_token": "1//0g....",
  "scope": "openid email profile",
  "token_type": "Bearer",
  "id_token": "eyJhbGciOiJSUzI1NiIs..."
}

GoogleTokenResponse converts that JSON response into a Java object.

The important fields                   Field	Meaning
access_token				 		   Token used to call Google's APIs on behalf of the user
id_token							   JWT containing information about the authenticated Google account
expires_in							   How many seconds the access token is valid
refresh_token						   Can be used to obtain new access tokens; may not be returned in every flow
scope								   Permissions/scopes granted
token_type							   Usually Bearer

1. First, imagine a real parking lot

You arrive at a parking lot.
At the entrance, there is a security guard.

You say:
"I'm Dinesh. I want to enter."
The guard doesn't simply trust you.

Instead, he says:
"Go to Google, prove who you are, and bring me the authorization slip."
Google verifies you and gives your application some information/tokens.

That entire package is what your Java code represents as:

GoogleTokenResponse

Think of it as:
A package that Google gives your backend after exchanging the authorization code.

2. What does this package contain?

Imagine Google gives your parking system a box:

             GOOGLE TOKEN RESPONSE
        ┌─────────────────────────────┐
        │                             │
        │  Access Token                │
        │  ID Token                   │
        │  Expires In                 │
        │  Refresh Token               │
        │  Scope                       │
        │  Token Type                  │
        │                             │
        └─────────────────────────────┘

Each item has a different purpose.
Let's understand each one.

3. access_token

Imagine the parking security guard gives you a special access card.
The card says:

"This person is allowed to access certain Google services."

For example, your application might want to access some Google API.
The access token is used like:

Your Parking Application
          |
          | access_token
          ↓
     Google API

For example:

GET Google API
Authorization: Bearer ya29.xxxxxxxxx

Google sees the access token and says:
"This application has permission to access the requested Google resource."
Important

The access token is primarily about:
What Google resources your application is allowed to access.
It is not the main thing you're using to identify the user in your current code.

Your application mainly uses:

tokenResponse.getIdToken()

4. id_token
   
This is the important one for your application.
Think of this as a Google-issued identity card.
Imagine Google gives your driver:

┌─────────────────────────────┐
│       GOOGLE ID CARD        │
│                             │
│ Name: Dinesh                │
│ Email: dinesh@gmail.com     │
│ Google ID: 123456789        │
│ Issuer: Google              │
│ Audience: Your Application  │
│                             │
│       GOOGLE SIGNATURE      │
└─────────────────────────────┘

Google digitally signs this identity card.
Your backend receives it as a JWT:
HEADER.PAYLOAD.SIGNATURE

For example:

eyJhbGciOiJSUzI1NiJ9.
eyJzdWIiOiIxMjM0NTY3ODkiLCJlbWFpbCI6ImRpbmVz...
.
abcXYZSignature...

Your code does:

GoogleIdToken token =
        googleVerifier().verify(tokenResponse.getIdToken());

You're essentially asking:
"Google, is this identity card really issued by you and has nobody modified it?"
The verifier checks the signature and important claims.

If valid:

GoogleIdToken.Payload payload = token.getPayload();

Then you can read:

String googleSub = payload.getSubject();
String email = payload.getEmail();
String name = (String) payload.get("name");

So:

GoogleTokenResponse
       |
       └── id_token
              |
              └── GoogleIdToken
                     |
                     └── Payload
                            |
                            ├── sub
                            ├── email
                            ├── name
                            ├── aud
                            ├── iss
                            └── etc.
5. expires_in

Now imagine Google gives you an access card.
The card has:
Valid for 1 hour
That's expires_in.

For example:

{
    "expires_in": 3600
}

Means:

3600 seconds
   ↓
60 minutes
   ↓
1 hour

So:

10:00 AM → token issued

10:00 AM → valid
10:30 AM → valid
10:59 AM → valid
11:00 AM → expired

Think:
"How long is the access token valid?"

6. refresh_token

Now imagine your access card expires after one hour.
Instead of going through the entire Google login process again, Google may give your application a special renewal card.
That's the refresh token.

Conceptually:

Refresh Token
      |
      ↓
Google
      |
      ↓
New Access Token

For example:

Access Token
valid for 1 hour
       ↓
expires
       ↓
Refresh Token
       ↓
Google
       ↓
New Access Token

But an important point:
A refresh token isn't necessarily returned in every login/token exchange.
Also, for your simple login flow, you don't need to think of it as something you automatically use every time.

7. scope

Imagine you go to a hotel.
You get a key card.
But the key card might allow:

Room 205       ✓
Gym            ✓
Swimming pool  ✓
Staff room     ✗

Those permissions are similar to scopes.
Your code requests:

scope=openid email profile
That means your application is asking Google for certain permissions/information.

For example:
openid
   ↓
OpenID Connect identity

email
   ↓
Email information

profile
   ↓
Basic profile information

So scope essentially answers:
"What access/information is being requested?"

8. token_type

Now imagine the security guard says:
"This is a Bearer access card."

That's similar to:

{
    "token_type": "Bearer"
}

Bearer basically means:
Whoever possesses this token can present it as the credential.
It is commonly sent like:

Authorization: Bearer ACCESS_TOKEN

So:

Authorization:
       Bearer
         +
    Access Token

Base64:

Base64 converts binary data into text.
Binary bytes
     ↓
   Base64
     ↓
	Text
Why getUrlEncoder() instead of normal Base64?

Normal Base64 can contain characters such as:
+
/
=

Some of these aren't convenient inside URLs.
Java therefore provides:

Base64.getUrlEncoder()
which uses URL-safe characters.

Conceptually:
Normal Base64
A-Z a-z 0-9 + /
              ↑ ↑
          problematic in URLs

URL Base64
A-Z a-z 0-9 - _
              ↑ ↑
          URL-friendly

URLEncoder:
URLEncoder is completely different
URLEncoder is used to percent-encode text for use as URL form/query data.

Session storage:

Your computer
┌─────────────────────────────────────┐
│ Tomcat JVM                          │
│                                     │
│   RAM                               │
│   ┌─────────────────────────────┐   │
│   │ HttpSession                 │   │
│   │                             │   │
│   │ ID = ABC123                 │   │
│   │ user_id = 10                │   │
│   │ name = Jothish              │   │
│   │ role = DRIVER               │   │
│   └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘


GoogleTokenResponse:

new GoogleAuthorizationCodeTokenRequest(
    HTTP_TRANSPORT,        // (1)
    JSON_FACTORY,          // (2)
    GOOGLE_CLIENT_ID,       // (3)
    GOOGLE_CLIENT_SECRET,   // (4)
    code,                   // (5)
    googleRedirectUri(request)  // (6)
)

HTTP_TRANSPORT — the underlying HTTP client implementation (typically NetHttpTransport or ApacheHttpTransport) that will actually send the POST request over the wire.
JSON_FACTORY — the JSON parser (GsonFactory or JacksonFactory) used to deserialize Google's JSON response into a GoogleTokenResponse object.
GOOGLE_CLIENT_ID — identifies which app is asking. Google checks this against the registered OAuth client in your Cloud Console project.
GOOGLE_CLIENT_SECRET — proves the request is genuinely coming from your backend and not an impersonator. This is why this call must happen server-side — a client_secret embedded in a browser or mobile app isn't a secret anymore.
code — the single-use authorization code you received in the redirect from step 2. Single-use is critical: reuse it and Google returns invalid_grant.
redirectUri — this is not used to redirect anywhere here. Its only purpose in this call is a security check: Google verifies this value matches, character-for-character, the redirect_uri that was originally sent in the authorization request in step 1. This prevents an authorization code that leaked from being redeemed by a different endpoint.

The library builds an HTTP POST request to Google's token endpoint (https://oauth2.googleapis.com/token), with a body like:
code=<the code>
client_id=<your client id>
client_secret=<your secret>
redirect_uri=<your redirect uri>
grant_type=authorization_code

JWT Token:

SIGNING (Google):
hash(header + payload) --[sign with private key]--> Signature

VERIFYING (your server):
recovered_hash = Signature --[unlock with public key]-->
fresh_hash     = hash(header + payload)     ← recomputed by your server itself

if recovered_hash == fresh_hash → valid, untampered
if recovered_hash != fresh_hash → invalid, reject

When moving 
beyond Meta’s test environment, you will generally need:
•
A real phone number dedicated to WhatsApp Business
•
A configured production WhatsApp Business Account
•
Business verification, depending on Meta’s requirements for the account
•
An approved display name
•
A permanent or system-user access token instead of a temporary development token
•
Approved message templates
•
Production recipient numbers
•
Proper privacy and consent handling for storing and messaging phone numbers
Registering a number with WhatsApp Business is separate from simply having a PHONE_NUMBER_ID. The test number is enough for development, but it is not suitable as your application’s long-term production sender.

Password Hashing using BCrypt:

The core idea: BCrypt isn't a hash function, it's a key derivation function built from a cipher

This is the first thing that trips people up. BCrypt doesn't use SHA-anything internally. It's built on top of a modified version of the Blowfish block cipher (a symmetric encryption algorithm), repurposed to be deliberately slow. It was designed in 1999 by Niels Provos and David Mazières specifically to solve the password-storage problem — not adapted from a general-purpose hash function after the fact.

Why not just use SHA-256 or MD5?
A cryptographic hash function like SHA-256 is designed to do one thing extremely well: be fast, so you can hash large files or verify data integrity quickly. That's a feature for file checksums and a catastrophic flaw for passwords.

Here's the math that makes this concrete. Suppose you use plain salted SHA-256:
A modern GPU can compute roughly 10+ billion SHA-256 hashes per second.
If a password database leaks, an attacker doesn't need to "break" SHA-256 mathematically — they just try every likely password, hash it, and compare. This is called an offline brute-force / dictionary attack.
At 10 billion guesses/sec, an 8-character password made of mixed case + digits (62^8 ≈ 218 trillion combinations) falls in about 6 hours.

BCrypt fixes this not by being mathematically stronger, but by being deliberately, adjustably slow. If BCrypt takes 250ms per hash instead of 0.0000001ms, the same attack that took 6 hours now takes over 190,000 years. That's the entire point: the algorithm itself is the defense, not just the randomness of the salt.
