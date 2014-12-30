---
layout: post
title: Parsing JSON and Passing Data Into a Table
---

Last week I talked about [querying data from an open data API](/2014/12/24/using-the-socrata-open-data-api-and-gem-to-make-complex-queries) for use in my project, [SaferNYC](http://safernyc.com). In this post, I'll talk about the results of those queries and how to deal with that data.

The Socrata Open Data API, like many APIs, returns data in the form of JSON. For all better purposes, it's an array of hashes, with each hash, in this case, corresponding to a collision. Each key in the hash is a column, such as "date" or "latitude." As an example, click [here](https://data.cityofnewyork.us/resource/h9gi-nx95.json?$select=borough,%20contributing_factor_vehicle_1,%20date,%20latitude,%20longitude,%20number_of_pedestrians_killed,%20off_street_name,%20on_street_name,%20vehicle_type_code1,%20zip_code&$where=number_of_pedestrians_killed%3E0%20AND%20date%3E'2014-10-31T00:00:00'%20AND%20date<'2014-12-01T00:00:00') to run a query for information on pedestrians killed in November.

The incidents/hashes in the array can be accessed using `.map`, and the value of each column can be accessed using Ruby hash syntax. Because the API only gives 1000 results (less than a month of collisions) per query, I realized early on that I needed to save the data to a database.

{% highlight ruby %}
class NypdCollisionData
  def initialize
    @data_collector = DataCollector.new
  end

  def get_incidents
    collisions = @data_collector.collisions
    collisions.map do |raw_incident|
      Incident.create(IncidentParser.new(raw_incident).parse)
    end
  end
end
{% endhighlight %}

In this class, `collisions` is the JSON (`DataCollector` was the focus of my last post), and `Incident.create` is being passed a hash containing the parameters for a new row in the table. The parse method simply does some minor cleaning of the data to prepare it to enter the table.

{% highlight ruby %}
# Bear with me on the awkward line breaks

class IncidentParser
  def initialize(raw_incident)
    @raw_incident = raw_incident
  end

  def parse
    {
      borough: @raw_incident["borough"],
      cause: @raw_incident["contributing_factor_vehicle_1"],
      cyclists_injured: @raw_incident["number_of_cyclist_injured"].to_i,
      cyclists_killed: @raw_incident["number_of_cyclist_killed"].to_i,
      date: @raw_incident["date"],
      key: @raw_incident["unique_key"],
      latitude: @raw_incident["latitude"].to_f,
      longitude: @raw_incident["longitude"].to_f,
      cross_street_name: @raw_incident["off_street_name"],
      on_street_name: @raw_incident["on_street_name"],
      pedestrians_injured: @raw_incident["number_of_pedestrians_injured"].to_i,
      pedestrians_killed: @raw_incident["number_of_pedestrians_killed"].to_i,
      vehicle_type: @raw_incident["vehicle_type_code1"],
      zip_code: @raw_incident["zip_code"]
    }
  end
end
{% endhighlight %}

More serious work is not needed here because essential columns with null values are weeded out in the query (see my [last post](/2014/12/24/using-the-socrata-open-data-api-and-gem-to-make-complex-queries)), and `.to_i` and `.to_f` eliminate null values in other important columns (by changing them to 0 or 0.0, which are appropriate values in this case). Incidents like the ones seen in your test query that have no location data will not even make it this far, so time will not be wasted attempting to parse those incidents.

There's one more step before the data in the table is viable. Many (usually about two thirds) of the incidents that have sufficient location data are still lacking latitude and longitude, and those values are now 0.0 in the database. Fortunately, this problem is very easily solved using the [Geocoder gem](http://www.rubygeocoder.com). Just a few lines of code, and the (hopefully) correct latitude and longitude will be entered into the database.

{% highlight ruby %}
class Incident < ActiveRecord::Base
  geocoded_by :cross_streets
  after_validation :geocode, if: ->(incident){ incident.latitude == 0.0 }

  def cross_streets
    "#{on_street_name} and #{cross_street_name}, #{borough}, NY #{zip_code}"
  end

  # Additional methods shown below
end
{% endhighlight %}

And finally, we'll want to be able to easily query for incidents that occurred within a certain number of months and didn't somehow fail geocoding and end up staying in the Gulf of Guinea.

{% highlight ruby %}
def self.good_data_in_time_range(number_of_months)
    where(date: between_dates(number_of_months)).where.not(latitude: 0.0)
end

def self.between_dates(number_of_months)
  most_recent_date = Date.strptime(Incident.order(date: :desc).first.date)
  start_date = most_recent_date.prev_month(number_of_months)
  start_date.strftime("%FT%T")..most_recent_date.strftime("%FT%T")
end
{% endhighlight %}

So I've taken JSON returned from a query to the SODA API, parsed it, entered it into a database, and geocoded it, and can run a query to get back relevant, useful data points from my database. Next time, I'll go over creating functional GeoJSON from these data points.
