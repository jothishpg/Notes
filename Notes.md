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
