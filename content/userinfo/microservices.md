+++
author = "RC Staff"
description = ""
title = "Microservices"
date = "2019-09-18T10:08:29-05:00"
draft = false
tags = ["compute","cloud","hpc","containers","dcos","hybrid"]
categories = ["userinfo"]
images = [""]
+++

<p class=lead>
  Microservice architecture is an approach to designing and running applications. They are typically run using containers.
</p>
<p class=lead>
  Containers are portable, efficient, and disposable, and contain code and any dependencies in a single package.
  Containerized microservices typically run a single process, rather than an entire stack within the same computing environment. 
  This allows portions of your application to be replaced or scaled as needed.
</p>

<p class=lead>
  Research Computing runs containers in an orchestration environment named DCOS (Distributed Cloud Operating System), based on Apache Mesos and Apache Marathon.
  This cluster has >1000 cores and ~1TB of memory allocated to running containerized services. DCOS also has over 300TB of cluster storage and can attach to project storage.
</p>

<img src="/images/microservice-cluster.jpg" alt="Microservices Architecture" style="" />

<p class=lead>How are microservices used for research? Typically in one of two ways:</p>

<ol>
  <li class=lead><b>Standalone microservices or small stacks</b> - Such as static HTML websites, interactive or data-driven web applications and APIs, databases, or scheduled task containers.</li>
    <ul style="margin-bottom:2rem;">
      <li>Simple web container to serve Project files to the research community or as part of a publication.
      <li>Reference APIs can handle requests based either on static reference data or databases.
      <li>Shiny Server presents users with interactive plots to engage with your datasets.
      <li>A scheduled job can retrieve remote datasets and stage them for processing.
    </ul>

  <li class=lead><b>Microservices in support of HPC jobs</b> - Some workflows in HPC jobs require supplemental services in order to run, such as relational databases, key-value stores, or reference APIs.</li>
    <ul style="margin-bottom:2rem;">
      <li>Cromwell/WDL pipelines rely on MySQL databases to track job status and state if a portion of your pipeline fails.
      <li>Key-value stores in Redis can track an index of values or a running count that is updated as part of a job.
    </ul>
</ol>

- - -

# Eligibility

<div class="alert alert-danger" role="alert">
To be eligible to run your microservice on our infrastructure, you must meet the following requirements:

<ul>
  <li>Microservices and custom containers must be for <b>research purposes only</b>.
  <li>Your container(s) must <b>pass basic security checks</b>. 
  <li>Containers <b>may not contain passwords</b>, SSH keys, API keys, or other sensitive information.
  <li>If bringing your own container, it must be <b>ready to go</b>! We cannot create custom containers for you.
</ul>
</div>

- - -

# Common Deployments

<table class="table">
  <thead>
    <tr>
      <th scope="col">Service</th>
      <th scope="col"></th>
      <th scope="col">Accessibility</th>
      <th scope="col" style="width:40%;">Details</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:4rem;" src="https://dcos.uvasomrc.io/images/nginx-500x500.png" /></th>
      <td style="font-weight:bold;">NGINX Web Server</td>
      <td>Public</td>
      <td>A fast web server that can run
        <ul>
          <li>Static HTML [<a target="_new" href="http://bioterms.org/">demo</a>]
          <li>Flask or Django apps [<a target="_new" href="http://bartweb.org/">demo</a>] 
          <li>RESTful APIs
          <li>Expose Project storage [<a target="_new" href="http://big.databio.org/">demo</a>]
        </ul>
      </td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:6rem;" src="/images/apache_logo.jpg" /></th>
      <td style="font-weight:bold;">Apache Web Server</td>
      <td>Public</td>
      <td>An extremely popular web server that can run your static HTML, Flask or Django apps, RESTful APIs, or expose files stored in Project storage.</td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:4rem;" src="/images/shiny-server.png" /></th>
      <td style="font-weight:bold;">Shiny Server</td>
      <td>Public</td>
      <td>Runs R-based web applications and offers a dynamic, data-driven user interface. <a href="https://www.rstudio.com/products/shiny/shiny-user-showcase/" target="_new">See a <b>demo</b></a> or try using <a target="_new" href="http://lolaweb.databio.org/"><b>LOLAweb</b></a></td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:4.5rem;" src="/images/mysql_PNG9.png" /></th>
      <td style="font-weight:bold;">MySQL Database</td>
      <td>Grounds only</td>
      <td>A stable, easy to use relational database. Run MySQL in support of your HPC projects in Rivanna or web containers.</td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:6rem;" src="https://dcos.uvasomrc.io/images/mongodb.png" /></th>
      <td style="font-weight:bold;">mongoDB Database</td>
      <td>Grounds only</td>
      <td>A popular NoSQL database. Use mongo in support of your Rivanna jobs or web containers.</td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:4rem;" src="https://dcos.uvasomrc.io/images/redis.svg" /></th>
      <td style="font-weight:bold;">Redis Database</td>
      <td>Grounds only</td>
      <td>An extremely fast, durable, key-value database. Use Redis in support of Rivanna jobs or other processes you run. [<a href="https://try.redis.io/" target="_new">Try Redis</a>]</td>
    </tr>
    <tr>
      <th scope="row" style="text-align:center;"><img style="max-width:4rem;" src="/images/bash_512x512.png" /></th>
      <td style="font-weight:bold;">Recurring Tasks</td>
      <td>n/a</td>
      <td>Schedule or automate tasks or data staging using the language of your choice (bash, Python, R, C, Ruby).</td>
    </tr>
  </tbody>
</table>

- - - 

# Pricing

Currently our microservices cluster is in beta testing. We welcome any single-container applications for free, 
either as a deployment listed above or a ready-to-run container that you bring.

Have a more complicated design? [Contact us](http://uvarc.io/support).

- - -

# Learn More

<span class="badge badge-default">1</span> Microservice architecture is a design principle, or a way of building things.

- [Microservices Whitepaper](/data/Mesosphere-ebook-Designing-Data-Intensive-Applications-by-Oreilly.pdf) from Mesosphere.
- [Best Practices for Microsoft Implementations](/data/Best-Practices-for-Microservices-Whitepaper-Research.pdf) from Mulesoft.
- Here's a brief video summarizing the design principles:

{{< youtube "2yko4TbC8cI" >}}

<div style="width:100%;height:2rem;"></div>

<span class="badge badge-default">2</span> The most popular way to run microservices is inside of containers:

- We teach workshops on containers and how to use them. Browse the course overview for <a href="https://workshops.rc.virginia.edu/lesson/containers/" target="_new">Introduction to Containers</a> at your own pace.
- Docker provides an excellent [Getting Started](https://docs.docker.com/get-started/) tutorial.
- Katacoda offers a great [hands-on Docker training series](https://www.katacoda.com/courses/docker) for free.

- - -

# Singularity

<img align="right" style="max-width:20%;" src="/images/rivanna/singularity-logo.png" alt="Singularity" />

Want to run your container within an HPC environment? It can be done, using Singularity! 

Singularity is a container application targeted to multi-user, high-performance computing systems. It interoperates well with SLURM and with the Lmod modules system. Singularity can be used to create and run its own containers, or it can import Docker containers.

[Read more](/userinfo/rivanna/software/containers/).

- - -

# Contact Us

Submit a consultation request to discuss your microservice implementation.

{{< button button-url="http://uvarc.io/support" button-class="success" button-text="Submit a Request">}}