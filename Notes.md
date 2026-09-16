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
