.. _getting-started:

.. Defined for inline java code highlighting
.. role:: java(code)
   :language: java

.. toctree::
   :name: getting_started_toc
   :includehidden:

Getting Started
===============

Here we will go through the basic steps on setting up a Beadledom project. We will go over
the key components needed to make sure your new service is bootstrapped and running correctly;
additionally, we will highlight the major pieces of the service so that you can extend it to
fit your needs. Finally we will point to some additional resources that will show some of the
additional functionality that you can get with the Beadledom framework.

Before we Start
---------------

We know there are a lot of different build tools available for Java, and we think that is great.
However for Beadledom we currently use `maven <https://maven.apache.org/>`_ as our build tool and
dependency manager. The script we will install to help us create our first Beadledom project is
really just a wrapper around `maven archetypes <https://maven.apache.org/archetype/index.html>`_ that let us
remove a lot of the boilerplate around project creation.

Installing the Installer
------------------------

Before we create our first Beadledom project we need to download the
`beadledom installer <https://github.com/cerner/beadledom/blob/master/archetype/bootstrap/>`_ utility.
If you would like to take a look at the installer source code before running it you can find it
`here install.sh script <https://github.com/cerner/beadledom/blob/master/archetype/bootstrap/bin/>`_.

Then to install execute in your shell

.. code-block:: bash

  curl -L ${beadledom.url}/beadledom-archetype/downloads/install.sh | sh

Once the download has completed you should be greeted by some sort of success message and now be
able to run

.. code-block:: bash

  beadledom --help

which should print the usage details to the screen.

Creating Our First Project
--------------------------

We are going to use the default archetype `simple-service <https://github.com/cerner/beadledom/blob/master/archetype/simple-service/>`_
in our example because it is fairly basic while
doing a good job at showing off a fair amount of the base feature set that Beadledom offers. In order to
kick off our project creation we just need to run

.. code-block:: bash

  beadledom new

or

.. code-block:: bash

  beadledom new simple-service

You will be prompted for several things many of which will have some relatively sane defaults. For
our example we are going to choose the following values

:Package Name: com.example
:Project Name: AwesomeThing
:Artifact Id: awesome-thing
:Group Id: com.example
:Beadledom Version: (whatever the default)

Feel free to choose whatever you like, we are just calling it out here to make it easier to references
pieces of the service later.

Running Our Service
-------------------

After a successful project creation we can go ahead and build our project and start the service

.. code-block:: bash

  > cd awesome-thing
  awesome-thing> mvn clean install
  . . .
  [INFO] BUILD SUCCESS
  . . .
  awesome-thing> cd service
  awesome-thing/service> mvn tomcat7:run-war

Once the service has started we can navigate to our swagger resource to explore the rest of the
default ``api``.

.. code-block:: html

  http://localhost:8080/awesome-thing-service/meta/swagger/ui

Go ahead and play around with the ``healthcheck`` and ``hello`` resources for a while. Once you
get tired of that move onto the next section where we will breakdown the components.

The Breakdown
-------------

Our project has three maven sub-modules: ``api``, ``client`` and ``service``. We will go through each
of these and talk about the classes they contain.

API sub-module
~~~~~~~~~~~~~~

This module contains the interfaces defining your service ``api`` and the models defining your
``responses``. The code living here ends up being used by both the ``client`` and the ``service``.

HelloWorldResource
++++++++++++++++++

.. code-block:: java

  @Api(value = "/hello", description = "Retrieve hello world data")
  @Path("/hello")
  public interface HelloWorldResource {

    @ApiOperation(
        value = "Retrieves hello world data.",
        response = HelloWorldDto.class)
    @ApiResponse(code = 200, message = "Success")
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public GenericResponse<HelloWorldDto> getHelloWorld();
  }

This is the definition for the ``/hello`` resource (and by resource we are talking about an api
resource). There is a lot going on here so we will add some comments to explain what each of these
lines are doing.

.. code-block:: java

  /**
   * Swagger annotation defining the base resource for the generated swagger documentation.
   */
  @Api(value = "/hello", description = "Retrieve hello world data")

  /**
   * JAX-RS annotation defining the base URI path for the resource.
   */
  @Path("/hello")
  public interface HelloWorldResource {

  /**
   * Swagger annotation defining the documentation for the HTTP verb this method provides. It
   * also includes the Dto class that is used for the response.
   */
    @ApiOperation(
        value = "Retrieves hello world data.",
        response = HelloWorldDto.class)

    /**
     * Swagger annotation for the different status codes our resource can
     * return (500 is often omitted).
     */
    @ApiResponse(code = 200, message = "Success")

    /**
     * JAX-RS annotation for the HTTP verb this method is providing functionality for.
     */
    @GET

    /**
     * JAX-RS annotation for the media type our service returns.
     */
    @Produces(MediaType.APPLICATION_JSON)

    /**
     * Method definition for the resource. The GenericResponse is an implementation provided
     * by Beadledom. It allows consumers (mostly of the client) to get a type safe response
     * from the server instead of having to do the casting or deserialization of the Json
     * payload themselves.
     */
    public GenericResponse<HelloWorldDto> getHelloWorld();
  }

And that's pretty much all that is needed to define an end-point. To learn more about how to
extend your path and about additional annotations that can be used in defining a resource we
suggest taking a look at `RESTEasy's documentation <http://resteasy.jboss.org/docs.html>`_ to
get started (RESTEasy is the current implementation of the JAX-RS standard that Beadledom is using).

HelloWorldDto
+++++++++++++

.. code-block:: java

  @ApiModel(value = "HelloWorldDto")
  public class HelloWorldDto {

    @ApiModelProperty(required = true, value = "name of the message author")
    private String name;
    @ApiModelProperty(required = true, value = "message content")
    private String helloWorldMessage;

    private HelloWorldDto(String name, String message) {
      this.name = Preconditions.checkNotNull(name);
      this.helloWorldMessage = Preconditions.checkNotNull(message);
    }

    /**
     * Creates a new {@link HelloWorldDto} instance.
     *
     * @param name author to be associated with the message
     * @param message the message to be returned
     * @return {@link HelloWorldDto} encapsulating a name and message
     */
    public static HelloWorldDto create(String name, String message) {
      return new HelloWorldDto(name, message);
    }

    /** Returns name. */
    public String getName() {
      return name;
    }

    /** Returns hello world message. */
    public String getHelloWorldMessage() {
      return helloWorldMessage;
    }
  }

This class is much less exciting than the `HelloWorldResource`_. The few things to note here are the
Swagger annotations. :java:`@ApiModel(value = "HelloWordDto")` defines the high level API model,
and as we are sure you have already guessed :java:`@ApiModelProperty` allows us to add some documentation
around the various fields that are going to be a part of our response.

Service sub-module
~~~~~~~~~~~~~~~~~~

Now that we have defined our ``api`` lets dive into the implementation and how this all gets
wired together. You'll notice that within our ``service`` sub-module we have not only our source
code directory we also have a resources and webapp directory. Lets spend just a short amount of
time going over what is in our non source code directories.

webapp/META-INF
+++++++++++++++

:Files:
  - **web.xml** - deployment descriptor that loads our context listener class as our servlets entry point

resources
+++++++++

:Files:
  - **log4j.properties** - configuration for our logger
  - **stagemonitor.properties** - configuration for `stagemonitor <http://www.stagemonitor.org/>`_
  - **build-info.properties** - properties from the current build used in health checks

All of the above mentioned configuration files are pretty standard. There are lots of other options
that can be tinkered with but we suggest you head to the developers pages for the various libraries
to learn more.

AwesomeThingContextListener
+++++++++++++++++++++++++++

.. Note:: Your class names might be slightly different depending on what you choose for your
   ``Project Name``.

.. code-block:: java

  public class AwesomeThingContextListener extends ResteasyContextListener {
    @Override
    protected List<? extends Module> getModules(ServletContext context) {
      return Lists.newArrayList(new AwesomeThingModule());
    }
  }

There isn't a lot going on in this file, but it is very important. This is the entry point to
our service and where we define our base Guice modules and configuration. We won't dive much
into the configuration component if you want to learn more about it (and we recommend you do)
you can find more documentation `here <https://github.com/cerner/beadledom/tree/master/configuration>`_.

The other method we are overriding here is providing our ``AwesomeThingModule`` so why don't we
take a look at that next.

AwesomeThingModule
++++++++++++++++++

.. code-block:: java

  public class AwesomeThingModule extends AbstractModule {
    protected void configure() {
      install(new ResteasyModule());

      BuildInfo buildInfo = BuildInfo.load(getClass().getResourceAsStream("build-info.properties"));
      bind(BuildInfo.class).toInstance(buildInfo);
      bind(ServiceMetadata.class).toInstance(ServiceMetadata.create(buildInfo));
      bind(HelloWorldResource.class).to(HelloWorldResourceImpl.class);
    }

    @Provides
    SwaggerConfig provideSwaggerConfig(ServiceMetadata serviceMetadata) {
      SwaggerConfig config = new SwaggerConfig();
      config.setApiVersion(serviceMetadata.getBuildInfo().getVersion());
      config.setSwaggerVersion(SwaggerSpec.version());
      return config;
    }
  }

This is where we install the main Beadledom module :java:`install(new ResteasyModule());`. Inside
this module is where Beadledom is going to install and bootstrap the various features it needs
in order to make our service tick. Below are some of the many awesome features that we get just in installing `ResteasyModule` without any extra efforts

- `Configuration <https://github.com/cerner/beadledom/tree/master/configuration#beadledom-configuration>`_ - lets us access all the configuration through a consistent API.
- `Health <https://github.com/cerner/beadledom/tree/master/health#beadledom-health>`_ - Health checks for the services
- `Jackson <https://github.com/cerner/beadledom/tree/master/jackson#beadledom-jackson>`_ - Serialization-DeSerialization for the payload
- `Swagger <https://github.com/cerner/beadledom/tree/master/swagger#beadledom-swagger>`_ - Enables the `Open API <https://openapis.org/specification>`_ documentation and `Swagger UI <http://swagger.io/swagger-ui/>`_

The next chunk of code

.. code-block:: java

    BuildInfo buildInfo = BuildInfo.load(getClass().getResourceAsStream("build-info.properties"));
    bind(BuildInfo.class).toInstance(buildInfo);
    bind(ServiceMetadata.class).toInstance(ServiceMetadata.create(buildInfo));

We pull in the properties from the ``build-info.properties`` and make them available to the service
and more importantly the healthcheck.

In the next binding :java:`bind(HelloWorldResource.class).to(HelloWorldResourceImpl.class);` we are
telling Guice that whenever we ask for an instance of `HelloWorldResource`_ we want it to inject
the implementation `HelloWorldResourceImpl`_.

That's it! There are additional Beadledom modules that we can install for other functionality, and
we could even create our own modules or bindings to provide additional classes our service will need
to operate. We recommend that you take some time and become familiar with `Guice <https://github.com/google/guice/wiki/GettingStarted>`_
so that you will know all it enables you to do.


HelloWorldResourceImpl
++++++++++++++++++++++

Last but not least, we have our implementation of the `HelloWorldResource`_ interface.

.. code-block:: java

  public class HelloWorldResourceImpl implements HelloWorldResource {
    public GenericResponse<HelloWorldDto> getHelloWorld() {
      return GenericResponses
          .ok(HelloWorldDto.create("Beadledom", "Hello World!"))
          .build();
    }
  }

That's it. Not very impressive we know, but it is just a starter service after all! More realistic
services will implement whatever sort of business logic, parameter validation, reading and writing
from databases here that they need to back their apis.

Client sub-module
~~~~~~~~~~~~~~~~~

There is a lot going on in the client sub-module, and none of it really has anything to do with
the implementation of your service. What we are able to do is build a client whose api is based
on the same JAX-RS annotations you use for your server. That makes it really easy to make clients
for your Java consumers; however, it won't help you much with consumers of other languages. Instead
of trying to explain all the concepts that are going on within the client we recommend you take a
look at our `client documentation <https://github.com/cerner/beadledom/tree/master/client>`_.

Where to go now
---------------

The service we just created and went through is just using a subset of features that Beadledom
can provide. If you would like to learn more about what Beadledom has to offer take a look at our
`table of contents <https://github.com/cerner/beadledom/tree/master>`_ and the related
documentation and code. If you find a bug or have an idea of a feature Beadledom is lacking open
an issue or pull request we are always happy to collaborate to help everyone get to a better place.

Now go build something awesome!
