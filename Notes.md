Servlet                                      Jersey (JAX-RS)

Tomcat maps URL → Servlet.	                 Tomcat maps URL → Jersey Servlet → Resource Method.
You extend HttpServlet.                      You write POJO classes with annotations.
You override doGet(), doPost().              You annotate methods with @GET, @POST.

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
