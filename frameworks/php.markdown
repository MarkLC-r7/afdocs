---
layout: doc-page
title: PHP
weight: 1
---

* [Custom PHP App](#custom)
* [Drupal](#drupal)
* [WordPress](#wordpress)
* [php.ini](#php_ini)
* [Document Root](#docroot)

### Supported Versions

For the most reliable experience, make sure you have the same version of PHP installed on your local development environment as the target AppFog instance. You can check the available runtimes by running: 

{: .prettyprint }
    $ af runtimes
    
    +--------------+-----------------+-----------+
    | Name         | Description     | Version   |
    +--------------+-----------------+-----------+
    | java         | Java 7          | 1.7.0     |
    | php          | PHP 5           | 5.3       |
    | ruby18       | Ruby 1.8.7      | 1.8.7     |
    | ruby192      | Ruby 1.9.2      | 1.9.2p180 |
    | ruby193      | Ruby 1.9.3 p125 | 1.9.3     |
    | python2      | Python 2.7.3    | 2.7.3     |
    | node04       | Node.js 0.4.12  | 0.4.12    |
    | node06       | Node.js 0.6.17  | 0.6.17    |
    | erlangR14B02 | Erlang R14B02   | R14B02    |
    +--------------+-----------------+-----------+

AppFog supports PHP with `Apache 2.2.22` and `mod_php`. You can take a closer look at the PHP and Apache configurations [here](http://phpinfo.aws.af.cm/info.php).

### Persistent Data Storage

AppFog does not yet have a persistent data storage system, though we're working on it. This means that the file system is volatile and any data that needs to be persistent should be included in the code base (by making all changes in a local development environment) or offloaded to a database or an external storage system like Amazon's S3.  

### Services

You can connect your PHP app to AppFog services by using the `VCAP_SERVICES` environment variable, which becomes available to your app when you bind a service to it. You can access the variable in PHP like this: 

{: .prettyprint}
    getenv('VCAP_SERVICES')

For more information on this, check out our [Services Overview](/services/overview) page.

# Custom PHP App {#custom}

### Create the App

Create a directory for the app and change into it:

{: .prettyprint }
    $ mkdir php-example
    $ cd php-example

Create an `index.php` file with the following:

{: .prettyprint .linenums}
    <?php echo "Hello world!"; ?>

### Deploy to AppFog

{: .prettyprint }
    $ af push php-example
    Would you like to deploy from the current directory? [Yn]:
    Detected a PHP Application, is this correct? [Yn]:
    Application Deployed URL [php-example.aws.af.cm]:
    Memory reservation (128M, 256M, 512M, 1G, 2G) [128M]:
    How many instances? [1]:
    Bind existing services to 'php-example'? [yN]:
    Create services to bind to 'php-example'? [yN]:
    Would you like to save this configuration? [yN]:
    Creating Application: OK
    Uploading Application:
        Checking for available resources: OK
        Packing application: OK
        Uploading (0K): OK
    Push Status: OK
    Staging Application 'php-example': OK
    Starting Application 'php-example': OK

{: .prettyprint }
    $ af curl php-example.aws.af.cm
    Hello world!% 

# Drupal {#drupal}

The following is a step-by-step guide to deploying a Drupal app to AppFog.

### Download Drupal

[Download Drupal](http://drupal.org/project/download/), unzip it to a new directory, and change into that directory.

Then create your `settings.php` file:

{: .prettyprint}
    $ cp ./sites/default/default.settings.php ./sites/default/settings.php

### Services

In `settings.php`, replace `$databases = array();` with:

{: .prettyprint .linenums}
    $services = getenv('VCAP_SERVICES'); 
    $services_json = json_decode($services,true); 
    $mysql_config = $services_json["mysql-5.1"][0]["credentials"]; 
    $databases['default']['default'] = array( 
        'driver' => 'mysql',
        'database' => $mysql_config["name"],
        'username' => $mysql_config["user"],
        'password' => $mysql_config["password"],
        'host' => $mysql_config["hostname"],
        'port' => $mysql_config["port"],
    );

### Deploy to AppFog

Push your code, making sure to create and bind a new MySQL service to the app:

    $ af push

### Finish the Drupal Install

Point your browser to your app's install script, in this case drupal-example.aws.af.cm/install.php. That should take you through the rest of the install process. 

### Further Developement

AppFog does not yet have a persistent data storage system, though we're working on it. This means that the file system is volatile and any changes made to the file system by the app will be lost on an app start, stop, or deploy. 

This means you should do any development that makes changes to the file system in a local development environment and then push those changes to AppFog using an `af update`. You can sync any database changes by [tunneling](/services/tunneling).

# WordPress {#wordpress}

The following is a step-by-step guide to deploying a WordPress app to AppFog.

### Download Wordpress

[Download WordPress](http://wordpress.org/download/), unzip it to a new directory, and change into that directory.

Then create your `wp-config.php` file:

{: .prettyprint}
    $ cp wp-config-sample.php wp-config.php

### Services

In `wp-config.php`, replace: 

{: .prettyprint .linenums}
    /** The name of the database for WordPress */
    define('DB_NAME', 'database_name_here');

    /** MySQL database username */
    define('DB_USER', 'username_here');

    /** MySQL database password */
    define('DB_PASSWORD', 'password_here');

    /** MySQL hostname */
    define('DB_HOST', 'localhost');

with: 

{: .prettyprint .linenums}
    $services = getenv("VCAP_SERVICES");
    $services_json = json_decode($services,true);
    $mysql_config = $services_json["mysql-5.1"][0]["credentials"];

    define('DB_NAME', $mysql_config["name"]);
    define('DB_USER', $mysql_config["user"]);
    define('DB_PASSWORD', $mysql_config["password"]);
    define('DB_HOST', $mysql_config["hostname"]);
    define('DB_PORT', $mysql_config["port"]);

### Deploy to AppFog

{: .prettyprint}
    $ af push wordpress-example
    Would you like to deploy from the current directory? [Yn]:
    Detected a PHP Application, is this correct? [Yn]:
    Application Deployed URL [wordpress-example.aws.af.cm]:
    Memory reservation (128M, 256M, 512M, 1G, 2G) [128M]:
    How many instances? [1]:
    Bind existing services to 'wordpress-example'? [yN]:
    Create services to bind to 'wordpress-example'? [yN]: y
    1: mongodb
    2: mysql
    3: postgresql
    What kind of service?: 2
    Specify the name of the service [mysql-d197d]:
    Create another? [yN]:
    Would you like to save this configuration? [yN]:
    Creating Application: OK
    Creating Service [mysql-d197d]: OK
    Binding Service [mysql-d197d]: OK
    Uploading Application:
        Checking for available resources: OK
        Processing resources: OK
        Packing application: OK
        Uploading (5M): OK
    Push Status: OK
    Staging Application 'wordpress-example': OK
    Starting Application 'wordpress-example': OK

### Finish the WordPress Install

Point your browser to your app's install script, in this case wordpress-example.aws.af.cm/wp-admin/install.php. That should take you through the rest of the install process. 

### Further Development

AppFog does not yet have a persistent data storage system, though we're working on it. This means that the file system is volatile and any changes made to the file system by the app will be lost on an app start, stop, or deploy. 

This means you should do any development that makes changes to the file system in a local development environment and then push those changes to AppFog using an `af update`. You can sync any database changes by [tunneling](/services/tunneling).

# `php.ini` {#php_ini}

AppFog does not support direct access to `php.ini`. However, the `AllowOverride` directive in `php.ini` is set to "`All`" which enables you to set the values of the directives in your `.htaccess` or in your PHP file via `ini_set()`.

### Setting Values `.htaccess`

Setting the value of a directive in `.htaccess` is easy. Simply place the following line of code in your `.htaccess` file, and insert the name and value of the directive you want to use:

{: .prettyprint}
    php_value <name> <value>

For more information on this topic, check out the following references: 

* [How to change configuration settings](http://php.net/manual/en/configuration.changes.php)
* [Set php.ini Values Using .htaccess](http://davidwalsh.name/php-values-htaccess)

### Setting Values Using `ini_set()`

Please consult the [PHP manual on `ini_set()`](http://www.php.net/manual/en/function.ini-set.php).

# Document Root {#docroot}

You can modify your document root adding the following into your `.htaccess`:

{: .prettyprint .linenums}
    RewriteEngine on
    RewriteCond %{HTTP_HOST} ^domain.com$ [NC,OR]
    RewriteCond %{HTTP_HOST} ^www.domain.com$
    RewriteCond %{REQUEST_URI} !public/
    RewriteRule (.*) /public/$1 [L]
