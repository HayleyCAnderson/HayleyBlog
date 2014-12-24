---
layout: post
title: Using the Socrata Open Data API and Gem to Make Complex Queries
---

Over the past few weeks, I've been working on my capstone project for Metis. My project, SaferNYC, explores data from [NYC Open Data](https://nycopendata.socrata.com/NYC-BigApps/NYPD-Motor-Vehicle-Collisions/h9gi-nx95?), visualizing New York City car collisions involving cyclists and pedestrians. You can view my project at [safernyc.com](http://safernyc.com).

The first step of my project was of course pulling in data from the dataset on NYC Open Data. NYC Open Data is maintained by Socrata, so the API connected with the site is the [Socrata Open Data API](http://dev.socrata.com/consumers/getting-started.html), or SODA. At its most basic, SODA can be queried through a url including parameters writing in SoQL, Socrata's version of SQL.

My query, however, quickly grew out of control for this approach. I only wanted data that was recent, had location information, and involved a cyclist or pedestrian death or injury. This created a url hundreds of characters long, and it just kept growing.

As my query got increasingly unmanageable, I discovered that there is also a gem for the API, [soda-ruby](https://github.com/socrata/soda-ruby). While the gem was very helpful for cleaing up some aspects of my query, the gem does not address the unwieldy SoQL parameters.

The soda-ruby docs contain only one simplistic example and little explanation. It's initially unclear how the gem is supposed to work, but the reality is that a complex query requires injecting strings of pure SoQL - almost exactly what would be used in the url approach.

The main benefit of using the gem is in the setup. Using the gem, environmental variables can be easily included in the Ruby code. Also, clauses such as `$select` (the data points requested), `$where` (requirements the data should meet), and `$order` (how the data will be sorted and returned) can be emphasized as keys pointing to the corresponding SoQL parameters rather than getting lost in a string.

{% highlight ruby %}
require "soda/client"

class DataCollector
  def initialize
    @client = SODA::Client.new(
      domain: ENV["soda_domain"],
      app_token: ENV["soda_application_token"]
    )
  end

  def collisions
    @client.get(
      ENV["soda_dataset_identifier"],
      "$select" => incident_data_points.join(", "),
      "$where" => incident_conditions,
      "$order" => "date ASC"
    )
  end

  private

  # Methods creating strings of SoQL - see below
end
{% endhighlight %}

It's a bit downhill from here. I used a slightly-better-looking array of symbols rather than a string to create the `$select` call, but as you may notice above, the array is simply joined with commas, creating a string of SoQL, before being passed in as a parameter. The symbols correspond to columns in the dataset from NYC Open Data (thus using their naming) and ask that the data returned includes each of these columns for each data point.

{% highlight ruby %}
def incident_data_points
  [
    :borough,
    :contributing_factor_vehicle_1,
    :date,
    :latitude,
    :longitude,
    :number_of_cyclist_injured,
    :number_of_cyclist_killed,
    :number_of_pedestrians_injured,
    :number_of_pedestrians_killed,
    :off_street_name,
    :on_street_name,
    :unique_key,
    :vehicle_type_code1,
    :zip_code
  ]
end
{% endhighlight %}

Tricks like the array of symbols above don't really work for the rest of the parameters because they're more complex. Using strings of SoQL instead gets very ugly very fast, so I broke mine up into numerous small methods. While this is better than one very long string, the methods are quite difficult to follow.

The methods below (which make up the rest of the class show above) contain SoQL giving the following paramaters:

* Data is from dates later than the last incident saved in the database (or, if the database is empty, is from six months ago)

* Data has either latitude and longitude or cross streets and zip code

* Data includes a cyclist or pedestrian fatality or injury.

{% highlight ruby %}
def incident_conditions
  "#{is_new} AND #{has_location_data} AND #{is_relevant}"
end

def is_new
  "date>'#{date_of_last_saved_incident}'"
end

def has_location_data
  "(#{has_latitude_and_longitude} OR #{has_address_information})"
end

def is_relevant
  "(#{pedestrian_involved} OR #{cyclist_involved})"
end

def date_of_last_saved_incident
  if Incident.exists?
    Incident.order(date: :desc).first.date
  else
    current_date = Date.strptime(Time.current.to_s)
    current_date.prev_month(6).strftime("%FT%T")
  end
end

def has_latitude_and_longitude
  "(latitude IS NOT NULL AND longitude IS NOT NULL)"
end

def has_address_information
  "(#{has_cross_streets} AND zip_code IS NOT NULL)"
end

def pedestrian_involved
  "number_of_pedestrians_injured>0 OR number_of_pedestrians_killed>0"
end

def cyclist_involved
  "number_of_cyclist_injured>0 OR number_of_cyclist_killed>0"
end

def has_cross_streets
  "(on_street_name IS NOT NULL AND off_street_name IS NOT NULL)"
end
{% endhighlight %}

This is all necessary because some points in the dataset are missing essential information, such as location, rendering them useless to me. More importantly, there is such a huge volume of data (and an API limit of 1000 data points per call - no more than a month of relevant data), that I can't waste time or space pulling in information that is old, irrelevant, or useless.

These parameters were as annoying to write as they look, but they're relatively straightforward, and I end up with data that includes the information I need but no more. Socrata's [documentation](http://dev.socrata.com/docs/queries.html) on SoQL is much better than their documentation on soda-ruby, and it made the content of this query quite simple to write.

Next, I'll cover the data this query returns and how I clean and store it.
