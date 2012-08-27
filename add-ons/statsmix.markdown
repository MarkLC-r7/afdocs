---
title: StatsMix
layout: doc-page
weight: 16
---

[StatsMix](http://www.statsmix.com/) makes it easy to track, chart, and share application and business metrics. 

Use StatsMix to:

* Log every time a particular event happens (such as a user creating a new blog post)
* View a real-time chart of these application events in StatsMix's web UI
* Share the charts with users inside and outside your organization
* Create and share custom dashboards that aggregate multiple metrics together
	* [Example dashboard](http://www.statsmix.com/d/0e788d59208900e7e3bc)
	* [Example embedded dashboard](http://www.statsmix.com/example-embedded)

## Install StatsMix

In the “Add-ons” tab in your app console click “Install” for the StatsMix add-on. That’s it!


Next, set up your app to start using the cache. We have documentation for the following languages and frameworks: 

* [Rails](#rails)
* [PHP](#php)

For other languages and frameworks, check out [this page](http://www.statsmix.com/developers/code_libraries).

For the StatsMix API, go [here](http://www.statsmix.com/developers).

## Rails {#rails}

First, install the StatsMix gem:


    $ gem install statsmix

Next, configure the gem according to your version of Rails:

## Rails 2

Add the following to `config/environment.rb`, then restart the server: 


    $ config.gem 'statsmix'

## Rails 3

Add the following to your Gemfile, then restart the server: 


    $ gem 'statsmix'

## Your API Key

AppFog keeps your StatsMix API key in an environment variable called `STATSMIX_API_KEY`, which you can retrieve using the command `ENV['STATSMIX_API_KEY']`. Populate your API key for the StatsMix gem with this: 


    $ StatsMix.api_key = ENV['STATSMIX_API_KEY']

## Using StatsMix to Track Events

Any time you want to track an event in your app, simply call:


	StatsMix.track(name_of_metric, value = 1, options = {})

If a metric with that name doesn't exist in your account, StatsMix will create one automatically. If the profile isn't set, the metric will use the first `profile_id` created in your account. (Developer, Basic, and Standard plans only have one profile.)

## Examples

Track every time a new blog post is created:


    # create a stat with the value 1 (default) for the metric called "Blog Posts"
	StatsMix.track("Blog Posts")
	
Count the number of new user accounts in the last 24 hours (i.e. run daily in a daily rake task):


	count = User.count(:conditions => ["created_at >= ?", 24.hours.ago]) 
    # create a stat with the value count for the metric called "Daily New Users"
	StatsMix.track("Daily New Users", count)

## Adding Metadata

You can optionally "tag" your data with metadata in the `:meta` option. For example, if you have a file upload utility and want to track what kinds of files you are receiving:


    # a user just uploaded a PDF file; track the type of file using metadata
	StatsMix.track("File Uploads", 1, {:meta => {"file type" => "PDF"}})

You can include multiple key-value pairs in the same stat:


	StatsMix.track("File Uploads", 1, {:meta => {"file type" => "PDF", "size" => "1 mb", "country of origin" => "Absurdistan"}})

## Adding Your Own Identifier

If you need the ability to update a stat after the fact, you can pass in a unique identifier `ref_id` (scoped to that metric).

For example, perhaps you want to update the number of users every hour instead of every day. If you pass in the date as `ref_id`, StatsMix will check whether a stat with that `ref_id` already exists for that metric. If found, StatsMix will update instead of inserting.


	count = User.count(:conditions => ["created_at >= ?", Time.now.strftime("%Y-%m-%d")]) 
    # create/update a stat using today"s date as the unique identifier
	StatsMix.track("Daily New Users", count, {:ref_id => Time.now.strftime("%Y-%m-%d")})

## Additional Options

If you’d like to change the timestamp of a stat, pass in the parameter `generated_at`.


    # backdate the stat to yesterday
	StatsMix.track("Blog Posts", 1, {:generated_at => 1.days.ago})

Higher level StatsMix accounts can have additional profiles. A profile can be described as a grouping or "bucket" of metrics. For example, an agency may want to create a profile for each customer.

If you are tracking metrics with the same name in two different profiles, you must specify which profile to use with `profile_id`. (If you don't specify `profile_id`, StatsMix will default to the first profile in your account.)


	StatsMix.track("Blog Posts", 1, {:profile_id => 789})

You can find the profile ID in StatsMix by creating a metric and looking at the API Details under Code Snippets in the Data & API section. (It will also be in the URL as /`profiles/123/metrics/456` - in this case the profile id is 123.)


## Limitations

If you hit a plan level limit (i.e. you go over the number of API requests available to your account), the API will return a 403 Forbidden error and an explanation.

The number of API requests and profiles you can create is based on the type of account you have. For example, Standard plans are limited to 300,000 API requests per month. You can see the current options on StatsMix's [pricing page](http://www.statsmix.com/home/plans).

## Local Setup

When running StatsMix in development, you can tell StatsMix to either ignore calls to it altogether or funnel them all into a single test metric.

For ignoring calls altogether, just put the following in a configuration file such as environment.rb or an initializer:


	StatsMix.ignore = true

For logging to a test metric, add this instead:


	StatsMix.test_metric_name = "My Test Metric"

This routes all `StatsMix.track` calls to "My Test Metric" regardless of the metric name passed.

Note: If StatsMix.ignore is set, then the test metric will not be used unless StatsMix.ignore is first reset to false.

## PHP {#php}

You can get 98% of our functionality by copying & pasting the examples below. 

The basic pattern:


	require "StatsMix.php";
	StatsMix::set_api_key(getenv("STATSMIX_API_KEY");
	StatsMix::track($name_of_metric,$value = 1,$options = array());

Push a stat with the value 1 (default) to a metric called "My First Metric":

	StatsMix::track("My First Metric");

Push the value 20:

	StatsMix::track("My First Metric",20);

Add metadata via the "meta" option in `$options` - you can use this to add granularity to your stats. This example tracks file uploads by file type.


	StatsMix::track("File Uploads", 1, array('meta' => array("file type" => "PDF")));

If you need the ability to update a stat after the fact (i.e. you're updating the same stat several times a day),  you can pass in a unique identifier called `ref_id`, which is scoped to the metric (so you can use the same identifier across metrics) This example uses the current date (in UTC time) for `ref_id`.


	StatsMix::track("File Uploads", 1, array('ref_id' => gmstrftime('%Y-%m-%d'), 'meta' => array("file type" => "PDF")));

If you need to timestamp the stat for something other than now, pass in a UTC datetime with the key `generated_at`


	StatsMix::track("File Uploads", 1, array('generated_at' => gmstrftime('%Y-%m-%d %H:%I:%S',strtotime('yesterday'))));

To turn off tracking in your development environment:


	StatsMix::set_ignore(true);

To redirect all stats in dev environment to a test metric:


	StatsMix::set_test_metric_name("My Test Metric");

If you have multiple profiles in your account, specify which one via `profile_id`:


	StatsMix::track("metric name that may be in multiple profiles", 1, array('profile_id' => "PROFILE_ID"));

To create metrics and stats using a more OO approach, check out the classes **SmMetric** and **SmStat** in StatsMix.php. Using them you can do things like this:

Create a metric:


	$metric = new SmMetric;
	$metric->name = "My Test Metric";
	$metric->save();
	if($metric->error){
		echo "<p>Error: {$metric->error}</p>";
	}
	//view the xml response:
	echo $metric->get_response();

## More Documentation

The StatsMix PHP Library supports all the methods documented [here](http://www.statsmix.com/developers/documentation).

## Contributing to statsmix
 
Check out [this GitHub repo](https://github.com/derekscruggs/PHP-library-for-StatsMix-API) to contribute.

## Copyright

Copyright &copy; 2011 StatsMix, Inc. See [LICENSE.txt](https://github.com/tmarkiewicz/statsmix/blob/master/LICENSE.txt) for further details.
