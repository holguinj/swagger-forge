Swagger Forge
=============
### Overview
This is a proof of concept for interactive documentation of the Puppet Forge API using the [swagger-ui](https://github.com/wordnik/swagger-ui) framework. Here's a rundown of its current features:

* detailed documentation of available API endpoints, parameters, and schemas
* interactive query formation using form elements
* ability to send queries to the Puppet Forge server and display responses

Here are a few drawbacks:

* it's a bit ugly right now (but there's nothing a bit of CSS couldn't fix)
* queries and responses have to be routed through a third-party proxy due to an annoying issue with [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)
* there may/will be additional API endpoints that aren't documented, and adding new endpoints is very time-consuming


### Installation
swagger-forge consists of static HTML and JavaScript, so it has no runtime requirements *per se*. That said, the repo includes a small script (`runserver`) that starts a simple webserver for demonstration purposes. To get it up and running on your machine, open up a terminal window and follow these steps:

1. `git clone git@github.com:holguinj/swagger-forge.git`
2. `cd swagger-forge`
3. `./runserver`
4. Open `http://localhost:8000` in your browser.

Feel free to give it a try and let me know if you have any suggestions or other feedback.