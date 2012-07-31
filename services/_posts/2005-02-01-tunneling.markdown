---
layout: doc-page
title: Tunneling    
weight: 1
---

You can interact with your provisioned services interactively by using the `af tunnel` command. This uses an app called "Caldecott", which is a `TCP` proxy over `HTTPS`. Caldecott creates a tunnel that connects a port on your local computer to the service in AppFog. The `af tunnel` command uploads the Caldecott app to your AppFog instance, sets up the tunneling, and offers to start a standard client on your computer to work with the service. This can be useful for managing your services as well as for debugging.

### Prerequisites {#prereqs}

* Caldecott requires Ruby 1.9.2.
* Caldecott starts a client program for the service you want to access, for example `mysql` for `MySQL` or `psql` for `PostgreSQL`. The client must be installed on your computer and be on the execution `PATH` so that Caldecott scripts can start it. If you want to use a different client, Caldecott will display the connection information and credentials you will need to connect to the service. The following table shows the client programs Caldecott can start for each service.

<table class="table table-bordered table-striped">

<tr>
<th>Service</th>
<th>Client</th>
</tr>

<tr>
<td>MongoDB</td>
<td><tt>mongo</tt></td>
</tr>

<tr>
<td>MySQL</td>
<td><tt>mysql</tt></td>
</tr>

<tr>
<td>PostgreSQL</td>
<td><tt>psql</tt></td>
</tr>

<!---
<tr>
<td>rabbitmq</td>
<td><i>none</i></td>
</tr>

<tr>
<td>Redis</td>
<td><tt>redis-cli</tt></td>
</tr>
--->

</table>

### Tunneling

Get a list of your services by using the `af services` command:

{: .prettyprint}
    $ af services

Which should return something like this: 

{: .prettyprint}
    ============== System Services ==============
    
    +------------+---------+-----------------------------+
    | Service    | Version | Description                 |
    +------------+---------+-----------------------------+
    | mongodb    | 1.8     | MongoDB NoSQL store         |
    | mysql      | 5.1     | MySQL database service      |
    | postgresql | 9.1     | PostgreSQL database service |
    +------------+---------+-----------------------------+
    
    =========== Provisioned Services ============
    
    +-----------------------+------------+
    | Name                  | Service    |
    +-----------------------+------------+
    | exampleapp1-mysql     | mysql      |
    | exampleapp2-mysql     | mysql      |
    | exampleappmongodb     | mongodb    |
    | example-postgres      | postgresql |
    +-----------------------+------------+

Create a tunnel to the service with `af tunnel <service>`. For example:

{: .prettyprint}
    $ af tunnel exampleapp1-mysql

The first time you create a tunnel, `af` uploads Caldecott to your Cloud:

{: .prettyprint}
    Uploading Application:
    Checking for available resources: OK
    Processing resources: OK
    Packing application: OK
    Uploading (1K): OK
    Push Status: OK
    Binding Service [exampleapp1-mysql]: OK
    Staging Application: OK
    Starting Application: OK

Then Caldecott creates the tunnel and prompts you to start a client. Here's an example session with `mysql`.

{: .prettyprint}
    Starting tunnel to exampleapp1-mysql on port 10000.
    1: none
    2: mysql
    3: mysqldump
    Which client would you like to start?: 2
    Launching 'mysql --protocol=TCP --host=localhost --port=10000
    --user=um4rwWyhwa07B --password=pBiBlqjINB6cmdd368741dbc1945cfb62315565efcf1b5'
    
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    .
    .
    .
    mysql>

When you exit the client, `af` disconnects the tunnel. 

If you choose the `1: none` or if Caldecott doesn't have a default client for the service, you can simply leave the terminal window open. You can then start your preferred client in another window. Once you're done, hit `Ctrl-C` to exit `af` and close the tunnel.

Note: `af tunnel` is currently only operating on AWS US East infrastructure, but we're working on enabling it for all infrastructures. This should be done very soon. To migrate MySQL data into other infrastructures, you can [use phpMyAdmin](/services/mysql#phpmyadmin).


### More about the Tunnel Command

You can simply enter the `af tunnel` command and respond to the prompts to create a tunnel. The command allows you to select from a list of existing provisioned services, so you don't need to know the service's name ahead of time.

The full syntax of the `af tunnel` command is:

{: .prettyprint}
    af tunnel [<servicename>] [--port <portnumber>] [<clientcmd>]

The `<servicename>` argument is the name of the service, as shown by the `af services` command. If you exclude `<servicename>`, `af` provides a list of existing services to choose from.

The `<portnumber>` parameter is the port to use on the local machine. By default, `af` chooses an available port in the `10000` range.

The `<clientcmd>` argument is the name of the client program to start. Only the client names shown in the table in the [Prerequisites](#prereqs) section are supported for this argument.

### Links

For more information, check out [this blog post](http://blog.cloudfoundry.com/2011/11/17/now-you-can-tunnel-into-any-cloud-foundry-data-service/) and [this doc](http://docs.cloudfoundry.com/tools/vmc/caldecott.html) from Cloud Foundry. You can also check out [Caldecott's GitHub repository](https://github.com/cloudfoundry/caldecott).
