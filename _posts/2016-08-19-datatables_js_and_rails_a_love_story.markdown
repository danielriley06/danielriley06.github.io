---
layout: post
title:  "DataTables.js and Rails: A love story"
date:   2016-08-19 19:55:50 +0000
---


So one thing I have realized that has been both a blessing and a curse is I 1) set really lofty goals for features for my assements and 2) those typically correlate to requiring skills that are one step ahead of where I am in my "learning to develop" career. For my rails assessment it was using DataTables! If I was a true masocist I would have gone with some angular tables (which I have dabbled with previously) but I decided to go with the easier option in my opinion.

There are a couple gems out there that deal with datatables, and specifically DataTables.js (https://datatables.net/). I tried out several of them and settled on jquery-datatables-rails. Quickly I realized that I was semi-lacking in how to handle the AJAX->jQuery relationship, at least 2 weeks ago. I soon discovered using ajax-datatables-rails in conjuction. 

Side note: you could set up the data necessary for tables in your controller and convert that to JSON but skinny controllers and all that, plus I wanted to have some more control over the data itself so I created some controllers seperately for my tables, but more on that later. There's more than one way to skin a cat, as they say.

**Setting up your Gems**

gem 'jquery-datatables-rails'
gem 'ajax-datatables-rails'

bundle install. You know the drill.

Now, at this stage you need to ask yourself if how you want to style the tables. If you search for the railscast from years ago and follow their method it will definitly work (and I recommend watching it if you get stuck) but it will also look like it belongs on a website from 1998 preferibly on geocities. Not even sure if that was possible or if jQuery existed yet, but you get the point. If you are leaning towards bootstrap for the rest of your site, then make sure you have the necessary gems installed for that, like twitter-bootstrap-rails. You may also need jquery-ui-rails, can't remember to be honest but it's in my gemfile :P

Also, important note from the readme if you get to the end and rails throws an error regarding sprockets/assets:

> The jquery-datatables-rails gem is listed as a convenience, to ease adding jQuery dataTables to your Rails project. You can always add the plugin assets manually via the assets pipeline. If you decide to use the jquery-datatables-rails gem, please refer to its installation instructions here.

**Generating and setting up your datatables controller/helper methods? And Controller**

Not really sure what it technically qualifies as, but it is ruby and it does set up the data needed for your tables. Run 'rails generate datatable MODEL' in your terminal to generate. The setup is pretty easy, especially the more straightforward your query is to get the data. It can get tricky if you want to use Associated and nested models depending on your grasp of table associations and the activerecord query interface. 

Follow the instructions included on the readme here: https://github.com/antillas21/ajax-datatables-rails and overall is pretty simple like I stated. 

> NOTE: For sortable_columns, assign an array of the database columns that correspond to the columns in our view table. For example [users.f_name, users.l_name, users.bio]. This array is used for sorting by various columns. The sequence of these 3 columns must mirror the order of declarations in the data method below. You cannot leave this array empty as of 0.3.0.

Next follow along in the readme and add the necessary call to the above model_datatable.rb in your controller. Again, easy peasy especially if you generated your controller and already have the format response for JSON.

**Setting up your View and Javascript**

More easiness! Setting up the view is simple, and if you are 'bootstrapping' your table just follow the bootstrap documentation for tables and set everything up in your view like you normally would for a static table. There are only two important caveats: 1) You have to have a matching number of column headers in your view, as you do being returned in the JSON or else if can cause issues and 2)leave the <tbody> blank. The gem knows to insert the table there.

```
<table id="users-table" data-source="<%= users_path(format: :json) %>">
  <thead>
    <tr>
      <th>First Name</th>
      <th>Last Name</th>
      <th>Brief Bio</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
```

Finally the Javascript itself can be slightly tricky. (Especially if you want to display two different tables in one view, that was quite the learning experience). If you just want to the basic configuration for the table, and only have one table on your site, its simply a matter of copy and pasting the following and changing the id to match your table. Once you get into customization and multiple tables it can be a little more challenging FYI.

```
# users.coffee

$ ->
  $('#users-table').dataTable
    processing: true
    serverSide: true
    ajax: $('#users-table').data('source')
    pagingType: 'full_numbers'
    # optional, if you want full pagination controls.
    # Check dataTables documentation to learn more about
    # available options.
```

Once you have your coffeescript setup, restart your rails server and refresh that page. Assuming you set everything up correctly, you should have a sortable, searchable, paginated table. I noticed that if no table appears at all, I typically had an issue with coffeescript->view setup. If table appears but its blank and you get an errror, most likely is an issue with your user_datatable.rb (or whatever your model is.)

I get the feeling that I missed some issues that I had early in the setup phase, but overall this is a really easy addition to your site that will make it look so much better when displaying larger tables. 

Good luck!


