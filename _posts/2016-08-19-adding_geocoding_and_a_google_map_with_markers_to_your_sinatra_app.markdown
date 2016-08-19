---
layout: post
title:  "Adding Geocoding and a Google Map with markers to your Sinatra app"
date:   2016-08-19 19:26:56 +0000
---


For my Sinatra assessment, I built a site to track breweries/taprooms and rate them. I also added a Google map that took the city and state you entered, geocoded it, and added markers with info to a Google map. It is definitly easier to do when using rails, but who doesn't love a challenge?! (There may have been a hint of sarcasam in that :D ) 

**Setting up your Gems**

I wrote a pretty long walkthrough, kind of for my own reference but if anyone is curious here is a list of the gem I used:

* gem 'json' (if you don't already have it)
* gem 'geocoder'
* gem 'gmaps4rails' (more on this later)

Easy enough. Bundle install, and you're on your way. The biggest stumbling block for me was the need to insert a railtie into the config.ru file in order to load the neccessary files for the geocoding to work. Simly add 'Geocoder::Railtie.insert ' **before** you load any of your controllers. 

**Setting up your Model for Geocoding**

Next, is the model setup. You need to add columns to the table you wish to geocode to store the latitude and longitude that the geocoding API returns. Make sure the columns are floats as well, since the lat/lng will look like '35.986746'. After that make sure you add the geocoder extension to your model 'extend Geocoder::Model::ActiveRecord'. Then add the callbacks to trigger the geocoding, there is some flexibility here but basically you add 'geocoded_by :column (or columns if you add a method to combine some) and after_validation :geocode which runs the process after the records are validated. You can also add after_save :geocode if you wish to geocode in case the address(s) are updated. Check out https://developers.google.com/maps/documentation/geocoding/intro for more info on what can be geocoded (address, city, etc.) and rate limits. Basically, your model should look like something similar to this:

```
def address
    [city, state].join(', ')
  end

  geocoded_by :address
  after_validation :geocode
  after_save :geocode
end
```

So hopefully you now have your lat/lng for each address and you're on the final stretch. The map itself is relatively easy to implement. One final note, check out the geocoder gem readme for info on how to batch geocode if you are adding this to a model that already contains a bunch of data.

**Setting up the Map Controller**

First I added a MapsController, that GETs a /map route. 

Then I created an instance variable called @breweries that found all the breweries for the current user. Then I created an @markers instance variable that created a hash of each brewery with a few specific keys (:title, :lat, :lng, :notes, and :rating) but you can customize this for your needs. The biggest thing is having a key named :lat and :lng. My ‘/map’ route for ref:

```
get '/map' do
@breweries = Brewery.where("user_id = ?", current_user.id).select("name, latitude, longitude, notes, rating")
    	@markers = @breweries.map do |b|
{ :title => b.name, :lat => b.latitude, :lng => b.longitude, :notes => b.notes, :rating => b.rating }
   	 end
   	 erb :'map/map', :layout => false
  end
```

Notice when I rendered my map.erb file I did not use the layout I had for the rest of the site. Again, optional, but it allows you to add the script for the map in the next step and not have to load it for the rest of your site. Not a huge deal but makes it easier IMO.

**Create your Map View**

Last step! 

Use https://github.com/danielriley06/taproom-index/blob/master/app/views/map/map.erb as reference. 

Basically, the key takeaways for your view are to load j query.js and add those two styles that I have in your <head>. You could create a separate map.css file and load that in your head for the styles, but I was too lazy.
And finally add the long script I have at the bottom of your <body> BEFORE you load the google maps api script. The reason for this is when it makes the API call it looks for some of the stuff in that previous script. Your map won’t load if you don’t have the right order. Also you will notice in that last script I have:
```
 <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyAgAEfy82TPTtD9EKfptYXZq7LeLyEGzqw&callback=initMap"></script>
 ```
That highlighted portion needs to be your API key. Easy (and free) just set it up and account on https://developers.google.com/maps/ and add whatever web service API key they give you.

Finally, add ```<div id="map"></div>``` into the body of your map view. You’ll notice in the script the small portion var map = new google.maps.Map(document.getElementById('map') looks for a div with the ‘id’ of “map” and loads it there. The JS can look a little overwhelming but the marker[‘infowindow’] section is what builds each marker window, so tweak it to match your information. The JS for the map uses JSON (which is why we required that gem at the beginning) so var location = <%= @markers.to_json %>; takes the @markers instance variable we created and turns it into JSON for the API to parse. 


Whew. You did it. Definitely recommend checking out google maps developer documents for some guidance, as well as the gem readme's and of course consult your friend Google. 





