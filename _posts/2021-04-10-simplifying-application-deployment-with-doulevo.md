---
layout: post
title: "Simplifying application deployment with Doulevo"
date: '2021--04-10 08:30:00'
permalink: simplifying-application-deployment-with-doulevo
post_class: feature
disqus_id: simplifying-application-deployment-with-doulevo
published: true
excerpt: "
Removing complex setup and configuration whilst avoiding vendor lock-in
"
---

**Removing complex setup and configuration whilst avoiding vendor lock-in**

This post imagines Doulevo: a command line tool and cloud backend that simplifies building, running, testing and deploying distributed applications and makes development more straightforward for all software developers.

You can think of Doulevo as like _Heroku_ but it is open source and builds on industry standard and open source technologies like Docker and Kubernetes.

This document represents an idea in development and will likely change in the future.

Doulevo is Greek for “put to work”.

# Primary aim

* Doulevo reduces the amount of overhead required to configure and deploy distributed applications giving more power to small teams who have limited resources.

# Key points

*   Doulevo provides a simpler means to build, run, test and deploy distributed applications and _parts of_ distributed applications.
*   Doulevo is open source, and builds on industry-standard and open source technologies like Docker and Kubernetes.
*   Doulevo supports services of all sizes (on the spectrum from monolith to microservices) and services are created with virtually any technology stack.
*   Doulevo can create boilerplate service projects for you or you can initialize it to work with existing projects.
*   Doulevo eschews vendor lock-in:
    *   Bring or build your own backend infrastructure.
*   Doulevo creates boilerplate configuration for testing and deploying your application.
    *   But you can peel back the layers of abstraction to customize application configuration or _eject_ the entire configuration and build scripts to take full control.
*   Includes everything you need:
    *   Embedded Docker and Kubectl (or BYO)
    *   Use whatever tech stack without installing specific tools (thanks to Docker)
*   There are opportunities for an enterprising startup or cloud vendor to commercialize and simplify creation of the Doulevo backend (e.g. one-click backend setup).

# Towards simplicity

Building and deploying distributed applications, whilst more accessible than ever before, is still more complicated than it needs to be. 

Modern tools for building applications like Docker and Kubernetes are fantastic and they have advanced and even revolutionised application development in recent years.

Docker provides standardization between development and production environments. It allows us to package our services for deployment and abstracts away the details of the technology stack used within the service.

Kubernetes gives us automated orchestration of containers across a elastically scalable cluster of computers and provides an API for automating deployments.

Importantly these open source tools have allowed us to break away from cloud-vendor lock-in.

As great as these technologies are, they are still complicated. They are complicated to learn and they are complicated to set up and configure. Of course they are also complicated to maintain over time. 

I believe there is a room for a new tool that builds on these standard technologies and that greatly simplifies the following:

*   Creation and management of services and their configuration;
*   Setup of development and testing infrastructure;
*   Development and maintenance of automated deployment pipelines; and
*   Creation and maintenance of backend infrastructure.

# Creating a new service project

A Doulevo project is simply a service built on virtually any technology stack (e.g. a microservice built on Node.js).

You can create a new Doulevo service with boilerplate code and configuration by invoking the _create_ command:
```bash
doulevo create <project-type> <project-name>
```

For example to create a Node.js project:
```bash
doulevo create nodejs my-new-javascript-service
```

Or a Python project:
```bash
doulevo create python3 my-new-python-service
```

Doulevo knows how to create projects for different technology stacks because it is plugin-based. Community created plugins extend the capabilities of Doulevo to work with many programming languages and many different backend environments. I’ll say more about plugins soon.

# Initializing an existing service project

You can also create a Doulevo project from an existing code-base using the _init_ command:
```bash
doulevo init
```

This _create _and _init _commands are interactive and allow you to select the name of the application and to authenticate with the application’s Doulevo backend. The authentication process should be simple and quick, akin to how cloud-vendors provide a command that allows you to download configuration to authenticate with your Kubernetes cluster.

The type of project (e.g. Node.js) is automatically detected (where possible) and the user is then prompted to confirm the project type or choose an alternative project type. The user may also have to enter commands to build, run and test the project (unless they can be automatically detected, for example the command `npm start` and `npm test` used by Node.js). 

# Configuration

Creating or initializing a project creates a configuration file (e.g. `.doulevo.yaml)` and a hidden cache directory (e.g. `.doulevo`). The configuration file records the project type (e.g. Node.js) and the commands to build, run and test the service. 

The cache directory records authentication details and caches any plugins that have been downloaded to make Doulevo work for various project types. The configuration file should be committed to version control, the cache directory should not.

# Many services

A distributed application is composed of many services and services can be located separate on your development computer or they can be co-located under a parent directory.

Each service on your development computer can be separately authenticated with the backend. This means you can be working with services for different applications or different backends by working in different directories on your development computer. 

# Running a service

A service is executed on the local development computer by invoking the _run_ command:
```bash
doulevo run
```

The service is automatically integrated with other services already running locally that are configured to be part of the same application. 

Services running in the same application use DNS to find each other and HTTP or messaging (e.g. RabbitMQ) to communicate with each other. Ports are mapped to the host computer based on the configuration file so that the service can be tested by any means possible from the development computer (e.g. browser, Postman, VS Code Rest Client, automated tests, etc).

Running a service automatically generates a development Dockerfile appropriate for the particular project type, builds the Dockerfile and then runs it. Live code reload is automatically supported, code is automatically synchronized through to the container running the service and the service is automatically restarted when code is edited.

# Testing a service

A service can be tested by two means. The first is through whatever ordinary automated testing you can use for your runtime environment. For example using Node.js I’d use Jest or Mocha to test my code in the normal way. The second is by running tests against the running service itself.

We invoke _test_ command to run both types of tests:
```bash
doulevo test
```

Doulevo first runs the runtime specific tests (e.g. by running `npm test` for Node.js). If those tests pass it then builds and runs the container for the service and runs tests against it (using a testing framework of your choice).

# Deploying a service

A service can be deployed to its configured backend by invoking the _deploy _command:
```bash
doulevo deploy
```

This automatically generates a prod Dockerfile for the service that is appropriate for the particular project type. It then builds the Docker image, pushes it to a private container registry running in the backend then deploys the image as a container to the application’s backend (e.g. a Kubernetes cluster). 

The configuration file specifies if the service is a _gateway_ for the application, gateways automatically have external end points generated. This makes the service publicly accessible. If not configured as a gateway, the service is only accessible internally within the backend. Whether a gateway or not, the service is automatically configured to be accessible via DNS within the backend. 

Doulevo automatically supports deployment to separate backend environments based on the Git branch that is being deployed. The main or master branch is automatically deployed to the development environment in the backend. Test and prod branches are respectively deployed to test and prod environments. Of course the names of the branches, the names of the environments and the number of environments is completely configurable. The configuration file can contain separate configuration sub-sections for each environment, for example 1 replica of the service in test and 3 replicas in prod.

Files called `.include` and `.exclude` control which files are included in the deployed service (this generates a `.dockerignore` file as part of the build process).

# Many projects

A distributed application is composed of many services. These can be independent projects in separate directories on the development computer, each project can be run using `doulevo run` or deployed with `doulevo deploy`.

Sets of services or indeed all the services for the whole application can be co-located in separate subdirectories under a multi-service repository (either mono- or meta-repository formats are ok). 

When services are combined together you can invoke `doulevo run` or `doulevo deploy`, to run and deploy sets of services together at the same time. You can indeed run or deploy an entire application this way which minimises the cost and complexity of initiating a new distributed application. Combining services this way is great for developing and testing different configurations of your application.

Note that Doulevo is smart and it won’t build and deploy a service unless the code has changed. So invoking `doulevo deploy` on a multi-service repository will only deploy those services that need to be deployed (unless you use the `--force` option to force a deployment to occur even if the code hasn’t changed).

# Automated deployment

You can have automated deployment of your Doulevo services by simply invoking `doulevo auth` and `duolevo deploy` from the CD pipeline of a service or multi-service code repository (as supported by GitHub, GitLab, Bitbucket and other version control providers).

You can also run tests in your CD pipeline by invoking `doulevo test`.

# The backend

Deployment works through plugins, so the Doulevo backend can be any cloud-based containers platform. However I’d like to see a canonical/reference backend built on Kubernetes that automatically includes a container registry configured to work with Doulevo.

I hope that it can be made easy to deploy the canonical Doulevo backend directly from the marketplace of any of the cloud vendors. This would mean a couple of clicks (and choosing a password or API key) is all that’s required to deploy a Doulevo backend to your cloud vendor of choice.

# Managed Doulevo

There is a commercial opportunity here for an enterprising startup or cloud-vendor to provide one-click deployment of Doulevo. 

Managed Doulevo should include:
*   A dashboard for
    *   management of the backend;
    *   Connecting code repositories and automatically creating deployment pipelines;
    *   Managing service deployments, environments and secrets; and
    *   Viewing and searching aggregated application logging.

# Plugin-based extensibility

Doulevo is built to be extended. It is built on plugins that allow it to be completely repurposed.

The _create _type of plugin knows how to create or initialize projects for various technology stacks. For example the _Node.js_ plugin knows how to create a Node.js project and the _Python3_ plugin knows how to build a Python project and so on.

The _build_ type of plugin builds a service. This defaults to using Docker to build an image. Underneath the _Docker build_ plugin are various other plugins that know how to create Dockerfiles for various project types. The various _create_ plugins can be reused for this.

The _deploy_ type of plugin knows how to deploy a service to a particular type of backend. The canonical backend for Doulevo knows how to deploy services to Kubernetes, but other cloud-based container runtimes can be supported using this type of plugin. 

# Customziation

Doulevo offers multiple levels of customization. 

Normally Doulevo automatically generates configuration files  such as Dockerfiles and Kubernetes files for your services. But you can also explicitly provide dev, test and prod Docker and Kubernetes files to override Doulevo’s automatic configuration generation.

You can invoke the _generate _command to automatically generate all configuration files into your project:
```bash
doulevo generate
```

After generating your configuration files you can then customize the ones you want to customize and delete the ones you don’t want to customize.

# My inspiration

Although the idea for Doulevo has been forming in my mind for some time (building an open source _Heroku_ is a common idea) but it came to fruition recently when I tried the _Parcel_ web site bundler. 

Parcel is like Webpack, but it eliminates almost all of the complex configuration that is required by Webpack. I then thought to myself, I already have great general-purpose recipes for building distributed applications, but they all require a lot of complex configuration. If only I could build something like Parcel, but for distributed application development that we can automatically generate all the boilerplate configuration (Docker files, Docker-Compose files, Kubernetes Yaml files, etc) for the recurring recipes that we use.

# Conclusion

Doulevo is a command line tool and backend that I have dreamed up to simplify building, testing and deployment of distributed applications.

Doulevo is built on industry standard open source tools like Docker and Kubernetes. It automates all the painful parts of building development, testing and deployment infrastructure. 

At the same time Doulevo frees you from vendor lock-in.

If this idea appeals to you, please let me know. I’m planning on doing a video soon to really get this idea across. For updates you can [follow me on Twitter](https://twitter.com/ashleydavis75).
