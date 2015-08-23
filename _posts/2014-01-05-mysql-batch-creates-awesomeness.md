---
layout: post
title: "MySQL Batch Create's Awesomeness"
date: 2014-01-05
categories: activerecord mysql
---

<p>Awhile back I had to insert tons of data into a MySQL database. The app had already been built and using a different database was not an option. The dataset I was creating was massive. I needed to create 100k or more records.</p>
<p>I attempted to do this with a nice clean ActiveRecord statement:</p>

{% highlight ruby %}
100000.times do |record|
  Record.create!({
    attribute_1: "attribute 1",
    attribute_2: "attribute 2",
    ...
    attribute_20: "attribute_20"
  })
end
{% endhighlight %}

<p>The results were not pretty. Although I didn't hit any timeouts, the query was taking HOURS - and likely would have timed out on a server that had less memory than my VM. This was not going to work because this was just one part of a module I was writing that needed to run.</p>
<p>I spent a lot of time researching creating lots of records at once and decided there was only one conclusion &mdash; the solution was MySQL batch insert. <strong>Note: this method will not work if you have complicated callbacks that need to run after each record is created</strong>. Here's how I handled the situation:</p>

{% highlight ruby %}
# setup the array to hold the values for the batch insert
record_values = []

# for this case we will pretend we have an array that needs to be sorted into readable data
record_data.each do |record|
  record_values << "('#{record.something}','#{record.something_else}','#{Time.now}'...'#{record.attribute_20})"
end

# set the number of records you want MySQL to handle at once. Too many records and MySQL will run out of memory
batch_size = 20000

# create a while loop to batch insert the records
while !record_values.empty?
  # use shift to move down the array based on batch size
  record_values_shitfted = record_values.shift(batch_size)
  
  # create the required sql string
  record_values_sql = "INSERT INTO records(attribute_1, attribute_2, attribute_3...attribute_20) VALUES#{record_values_shifted.join(", ")}"
  
  # execute sql
  ActiveRecord::Base.connection.execute(record_values_sql)
end
{% endhighlight %}

<p>This was infinitely faster than ActiveRecord could be and unfortunately it doesn't have a way to do a batch insert. Sometimes straight up MySQL is faster and better than ActiveRecord.</p>
