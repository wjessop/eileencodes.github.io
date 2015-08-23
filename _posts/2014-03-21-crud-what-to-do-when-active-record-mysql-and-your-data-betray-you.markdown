---
layout: post
title: "CRUD! What to do When Active Record, MySQL, and Your Data Betray You"
date: 2014-03-21
categories: activerecord mysql
---

<p>On March 21st 2014 I gave my first conference talk at Mountain West Ruby. For the past year I have dealt with issues with Active Record that generally stemmed from a) that PhishMe has a LOT of data, and b) I didn't always understand how Active Record would translate into MySQL.</p>
<p>The slideshow is available on <a href="https://speakerdeck.com/eileencodes/crud-what-to-do-when-active-record-mysql-and-your-data-betray-you" target="_blank">Speaker Deck</a>.</p>
<p>The application discussed in this presentation is a real application and available on <a href="https://www.github.com/eileencodes/crud_project" target="_blank">github</a>.</p>
<p>For each of the fundamental CRUD functions; create, read, update, and delete, I'll demonstrate a problem I ran into with Active Record and the solution I came up with. My examples are based on an application that is an address book. The models and their relationships in an address book are easy to understand and explain. I didn't want to audience to get hung up on the application we were discussing.</p>
<p>In this address book application there is a User model. Users have many contacts and have many categories. Each contact belongs to a user and can have many categories through categorizations. Categories belong to a user and can have many contacts through categorizations. Contacts and categories are part of a many-to-many association and are connected through categorizations. Categorization belongs to a contact and belongs to a category for the join table.</p>
<h3>Create</h3>
<p>Imagine we have a CSV spreadsheet that we want to use to create our contacts. This spreadsheet has ten thousand rows. We could just run through each row of the CSV and insert each individual contact.</p>
{% highlight ruby %}
CSV.foreach("#{filepath}", headers: true) do |csv| 
  Contact.create!({
    first_name: csv[0],
    last_name: csv[1],
    birthday: csv[2],
    ...
  })
end
{% endhighlight %}
<p>This will create the following SQL constructing an INSERT statement for each individual contact that needs to be added to the database.</p>

{% highlight text %}
INSERT INTO `contacts` (`first_name`, `last_name`, `birthday`,...) VALUES ('John', 'Smith', '1987-02-01',...);
{% endhighlight %}
<p>This INSERT statement will be run ten thousand times, which will take quite awhile. What if there was a way to insert more than one record at a time?</p>
<p>After a lot of research I found the quickest way was to use MySQL's Batch Insert. This method will speed up our creation of ten thousand records quite a bit. Now don't be scared but this means that we're getting our hands dirty with raw SQL. It's not often this happens in Rails but unfortunately there is no comparable method in Active Record so we are going to abandon it for this example.</p>
<p>Using the same spreadsheet of ten thousand contacts we'll create the records using MySQL Batch Insert.</p>
{% highlight ruby %}
contact_values = []
CSV.foreach("#{filepath}", headers: true) do |csv|
  contact_values << "('#{csv[0]}','#{csv[1]}','#{csv[2]}'...)"
end

batch_size = 2000

while !contact_values.empty?
  contacts_shifted = contact_values.shift(batch_size)
  contacts_sql = "INSERT INTO contacts(first_name, last_name,     
                  birthday,...)
                  VALUES#{contacts_shifted.join(", ")}"
  ActiveRecord::Base.connection.execute(contacts_sql)
end
{% endhighlight %}
<p>First we'll read each row of the CSV and create an array of all of the values that need to be inserted into the database. Then we set a batch_size. This is really important because although MySQL can handle multiple contacts it can't handle all ten thousand being inserted at once. We'll end up blowing our MySQL innodb_buffer_pool_size or max_query_size if we're not careful. After a lot of trial and error I found 2k was a reliable setting for my servers, but you may have to experiment on your own.</p>
<p>Until we've inserted all of the contacts, the contact_values array is shifted by batch size. Shift removes the specified number of values from the front of the array and returns them. It does this until the contact values array is empty.</p>
<p>An SQL statement is then build with the attribute names and values to be inserted. We need to join the shifted contacts to complete the value list. Batch insert is made possible through the insert syntax by feeding it the values for each record that we want created. This can be quite tedious if you have a lot of columns because the columns names and values from the array must line up perfectly.</p>
<p>And finally we connect to the database and execute the INSERT statement. I'd like to note that this example assumes we have sanitized the input against SQL injection.</p>
<p>Batch insert creates the following MySQL query.</p>
{% highlight text %}
INSERT INTO contacts(first_name, last_name, birthday,...)
VALUES('Lauretta','Senger','1987-02-01',...), ('Jane','Roob','1987-02-01',...), ('Blaze','Lakin','1987-02-01',...),
('Elton','Cormier','1987-02-01',...),
('John','Kohler','1987-02-01',...),
('Clementine','Marvin','1987-02-01',...),
('Ova','Aufderhar','1987-02-01',...)
...
{% endhighlight %}
<p>It looks a lot similar to the other create statement, except it's chaining all the values instead of making a new INSERT statement for each contact. Let's take a look at how these queries benchmark.</p>
<p>When using the Benchmark module from the Ruby Standard Library the output represents user CPU time, system CPU time, the sum of user and system CPU and the elapsed real time. For my examples we're going to focus on the total time of user plus system.</p>
<p>For the example where we created each record individually it took 45.9 seconds in total time. That's a long time to wait for ten thousand records to be inserted into the database. MySQL Batch Insert took less than 3 seconds. That's a huge difference.</p>
{% highlight text %}
ActiveRecord Benchmark Data:
          user     system      total         real
 =>  44.740000   1.180000  45.920000 ( 51.095556)



MySQL Batch Insert Benchmark Data:
          user     system      total         real
 =>   2.710000   0.050000   2.760000 (  3.227031)
{% endhighlight %}
<p>Benchmarking times may vary a little based on garbage collection, allocated memory, and the version of ruby and rails that you're using, but that does not change the fact that MySQL Batch Insert is drastically faster than creating and saving each individual record. But because we are not saving each record no callbacks will be fired. We're completely skipping the model and going straight to the database.</p>
<h3>Read</h3>
<p>Let's say we want to output the first name of each contact in the database. There are multiple ways to achieve the same result.</p>
<h4>Reading Records with .each</h4>
<p>We can run each on contacts and output the first name, but if we have a lot of records this is going to be quite costly to memory. A single SQL statement is run and all the records are collected at once.</p>
{% highlight ruby %}
Contact.where(user_id: 1, country: "USA").each do |contact|
  puts contact.first_name
end

SELECT `contacts`.* FROM `contacts` WHERE 
`contacts`.`user_id` = 1 AND `contacts`.`country` = 'USA'
=>
Lauretta
Shana
Jason
Jermain
Blaze
Jessy
Bradly
...
{% endhighlight %}
<h4>Reading Records with .find_each</h4>
<p>.find_each is a great way to save both time and memory. .find_each will collect our data in batches of 1000. Regardless of speed we want to be sure we are always using our server resources effectively and efficiently. .find_each helps us do that by running limited select statements so we don't blow our memory.</p>
{% highlight ruby %}
Contact.where(user_id: 1, country: "USA").find_each do |contact|
  puts contact.first_name
end

SELECT `contacts`.* FROM `contacts` WHERE 
`contacts`.`user_id` = 1 AND `contacts`.`country` = 'USA' 
ORDER BY `contacts`.`id` ASC LIMIT 1000
=>
Lauretta
Shana
Jason
...
SELECT `contacts`.* FROM `contacts` WHERE `contacts`.`user_id` 
= 1 AND `contacts`.`country` = 'USA' AND (`contacts`.`id` > 
1001) ORDER BY `contacts`.`id` ASC LIMIT 1000
=> ...
{% endhighlight %}
<h4>Reading Records with .pluck</h4>
<p>Since we are only outputting the first_name of each contact we can use .pluck to get just that attribute. This is going to be much faster because this method creates an array of strings instead of returning objects. We only will have the first name attribute though and not the rest of the record.</p>
{% highlight ruby %}
Contact.where(user_id: 1, country: "USA")
.pluck(:first_name).each do |first_name|
  puts first_name
end

SELECT `contacts`.`first_name` FROM `contacts` WHERE `contacts`.`user_id` = 1 AND `contacts`.`country` = 'USA'
 => ["Lauretta", "Shana", "Jason", "Jermain", "Blaze", "Jessy", "Bradly", "Emma", "Cruz", "Elton", "Dashawn", "Rosanna", "Ryan", "Leonel", "Ashly", "Mittie", "Tobin", "Antonio", "Chad", "Lauryn", "Sydnie", "Sebastian", "Johnpaul", "Yasmeen", "Junior", "Monroe", "Avery",...]
{% endhighlight %}
<h4>So which one was fastest?</h4>
{% highlight text %}
.each Benchmark Data:
         user     system      total         real
=>   0.950000   0.060000   1.010000 (  1.050266)

.find_each Benchmark Data:
         user     system      total         real
=>   0.900000   0.040000   0.940000 (  0.976979)

.pluck Benchmark Data:
         user     system      total         real
=>   0.080000   0.020000   0.100000 (  0.126814)
{% endhighlight %}
<p>.each benchmarked at 1 second. .find_each was not much faster but again with collecting records we're more concerned with memory than time. 10k records won't hurt our memory much but 100k will have much more of an impact. With pluck we can see a lot of time is saved. It's a lot faster than .each or .find_each since we are only collecting the records first name.</p>
<p>I understand you may not be impressed with these numbers but what would these benchmarks look like if we had 100k records?</p>
<p><img src="/assets/chart.png" alt="Read chart benchmarks" width="100%" /></p>
<p>Here we can see when the dataset increases from 10k to 100k records the length of time queries take increases quite a bit and the savings are more obvious. On the y axis we have the number of seconds and the x axis represents the number of records. For 100k records .each in blue takes almost 11 seconds, .find_each in purple takes 9.43 and in red is the fastest at 2.11 seconds.</p>
<p>Another interesting method that Active Records provides is .find_by_sql. This method allows us to craft custom SQL queries. At times when writing our queries this may be faster and more efficient because Active Record doesn't always know the best way to get the data we're looking for. We can compare dates or optimized queries for performance with .find_by_sql.</p>
{% highlight ruby %}
Contact.find_by_sql([
         "SELECT first_name FROM contacts WHERE 
 user_id=? AND    
         country=? AND birthday > ?",
         1, 'USA','1987-01-01'
       ]).each do |contact|
  puts contact.first_name
end
{% endhighlight %}
<h3>Update</h3>
<p>Let's say we wanted to change all of our categorizations from the "Coworkers" category to the "Networking" category. In this query we are collecting all the records and updating the category_id attribute on each one using update_attributes. We are instantiating and updating each individual record for all ten thousand.</p>
{% highlight ruby %}
category = Category.where(name: "Networking").first

Categorization.all.each do |categorization|
  categorization.update_attributes(category_id: category.id)
end
{% endhighlight %}
<p>This produces the following SQL, constructing an UPDATE statement for all ten thousand records.</p>
{% highlight text %}
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 1
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 2
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 3
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 4
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 5
UPDATE `categorizations` SET `category_id` = 1 WHERE `categorizations`.`id` = 6
...
{% endhighlight %}
<p>A better and faster way to update all the contacts categories from the "Coworkers" category to the "Networking" category would be to use update_all.</p>
{% highlight ruby %}
category = Category.where(name: "Networking").first

Categorization.update_all(category_id: category.id)
{% endhighlight %}
<p>This method creates a single SQL UPDATE statement. Records are not instantiated and are all updated at once without running save on each record. Again since we are not saving individual objects, callbacks will not be fired.</p>
<code>UPDATE `categorizations` SET `categorizations`.`category_id` = 1
</code>
<h4>Benchmark Data</h4>
<p>The first query where we update each individual record one at a time takes almost 14 seconds. That's not too long but we can do better. .update_all is so fast for ten thousand records that it barely registers as taking any time at all. That's incredible savings we're seeing here.</p>
{% highlight text %}
update_attributes Benchmark Data:
          user     system      total         real
 =>  12.990000   0.870000  13.860000 ( 17.156265)


update_all Benchmark Data:
          user     system      total         real
 =>   0.000000   0.000000   0.000000 (  0.042140)
{% endhighlight %}
<h3>Delete</h3>
<p>To talk about delete we first need to discuss the differences between delete_all, destroy_all and how setting a dependency on a has_many association affect their outcome.</p>
<h4>Contact.destroy_all with a dependency set</h4>
<p>In cases where we are deleting a model through destroy_all AND the dependency is set to delete_all or destroy the Contact and all associated categorizations will be removed. destroy_all on Contact will remove contacts individually and fire callbacks when a dependency is specified. From the SQL you can see the associated categorization is selected, removed, and then the parent contact is then deleted. It will do this individually for all ten thousand contacts and all associated categorizations in the database.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations, dependent: :destroy
has_many :categories, through: :categorizations

Category
has_many :categorizations, dependent: :destroy
has_many :contacts, through: :categorizations

# query run
Contact.destroy_all

# sql produced
SELECT `categorizations`.* FROM `categorizations` WHERE `categorizations`.`contact_id` = 1
DELETE FROM `categorizations` WHERE `categorizations`.`id` = 1
DELETE FROM `contacts` WHERE `contacts`.`id` = 1
{% endhighlight %}
<h4>Contact.delete_all with a dependency set</h4>
<p>If we instead run delete_all on Contact only those contacts will be removed, regardless if the dependency is set to delete_all or destroy. The following SQL statement will be produced. No categorizations will be removed when using a delete_all on contacts. delete_all will not destroy individual contact and will not fire callbacks when run on a parent model. All contacts are removed at once.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations, dependent: :delete_all
has_many :categories, through: :categorizations

Category
has_many :categorizations, dependent: :delete_all
has_many :contacts, through: :categorizations

# query run
Contact.delete_all

# sql produced
DELETE FROM `contacts`
{% endhighlight %}
<h4>Contact.delete_all with no dependency set</h4>
<p>If no dependency is set the delete_all and destroy_all both ignore the related categorizations and only remove the contacts. The difference is in how they remove those contacts. delete_all removes them all at once with the exact same SQL statement demonstrated earlier. It ignores the categorizations as it did before.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations
has_many :categories, through: :categorizations

Category
has_many :categorizations
has_many :contacts, through: :categorizations

# query run
Contact.delete_all

# sql produced
DELETE FROM `contacts`
{% endhighlight %}
<h4>Contact.destroy_all with no dependency set</h4>
<p>destroy_all without a dependency removes each contact individually, but not categorizations. This is because no dependency is specified.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations
has_many :categories, through: :categorizations

Category
has_many :categorizations
has_many :contacts, through: :categorizations

# query run
Contact.destroy_all

# sql produced
DELETE FROM `contacts` WHERE `contacts`.`id` = 1
DELETE FROM `contacts` WHERE `contacts`.`id` = 2
DELETE FROM `contacts` WHERE `contacts`.`id` = 3
...
{% endhighlight %}
<h4>Deleting Associated Records Through a Parent Record</h4>
<p>It's important to keep these details in mind when setting up how our models are associated. These dependencies can have some interesting side effects when delete_all or destroy are run on associated records through a parent model.</p>
<p>What if we wanted to run a query like <code>category.contacts.destroy_all</code>. Regardless of the dependencies this code has a big problem. When run this won't delete the contacts that are related to the category because category does NOT own contacts. The only way category knows about contacts is through categorizations. Therefore, this code is only going to delete the associated categorizations and not the contacts.</p>
<p>We should change that to be more clear. No one has their model associations memorized, and many developers will think the contacts are being deleted, not the contacts. So let's write <code>category.categorizations.destroy_all</code></p>
<p>Unfortunately destroy_all is going to be slow. We're not deleting category so we aren't going to gain anything from the destroy_all callbacks. I want to just the delete the categorizations. Let's instead run <code>category.categorizations.delete_all</code>.</p>
<p>The relationships between category and categorizations is called a CollectionProxy. The Rails docs describe a CollectionProxy as the middleman between the object that holds the association - in our case the category - and the associated object - the categorizations. Let's take a look at how dependency settings on a has_many association affect deleting records through a CollectionProxy.</p>
<p>If we run <code>category.categorization.delete_all</code> and have no dependency set, instead of removing the categorization the category_id in those records will be set to null. This is because the default dependency setting on a CollectionProxy is to nullify. The records will be left in the database instead of being removed. We can see an update statement is run setting the categorizations category id to null instead of removing the record.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations
has_many :categories, through: :categorizations

Category
has_many :categorizations
has_many :contacts, through: :categorizations

# query run
category.categorizations.delete_all

# sql produced
UPDATE `categorizations` SET `categorizations`.`category_id` = NULL WHERE `categorizations`.`category_id` = 1
{% endhighlight %}
<p>What if we set the dependency option to destroy? Well in that case each record would be instantiated and deleted individually. If that is what we wanted we would have just used destroy_all instead of delete_all. Generally when using delete_all we want the records to be deleted at once and not fire callbacks because we're trying to save time and memory. We don't want to wait for individual object to be removed. It defeats the purpose of using delete_all in the first place.</p>
{% highlight ruby %}
# model relationships
Contact
has_many :categorizations, dependent: :destroy
has_many :categories, through: :categorizations

Category
has_many :categorizations, dependent: :destroy
has_many :contacts, through: :categorizations

# query run
category.categorizations.delete_all

# sql produced
DELETE FROM `categorizations` WHERE `categorizations`.`id` = 1
DELETE FROM `categorizations` WHERE `categorizations`.`id` = 2
DELETE FROM `categorizations` WHERE `categorizations`.`id` = 3
DELETE FROM `categorizations` WHERE `categorizations`.`id` = 4
...
{% endhighlight %}
<p>Alright, so what happens if we set the dependency to delete_all? It should be fast and efficient, right? This query takes <em><strong>130 seconds</strong></em>. 130 seconds. Why? What's going on here?</p>
{% highlight text %}
category.categorizations.delete_all

          user     system      total         real
 => 130.080000   0.120000 130.200000 (130.308334)
{% endhighlight %}
<p>I expected the SQL statement to be <code>DELETE FROM `categorizations` WHERE `categorizations`.`category_id` = 1</code> but instead I got:</p>
{% highlight text %}
DELETE FROM `categorizations` WHERE `categorizations`.`category_id` = 1 AND 
`categorizations`.`id` IN (1, 2, 3, 4, 5, 6, 7, 8, 9,
10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 
23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46...10000)
{% endhighlight %}
<p>This is because when deleting a CollectionProxy with a delete_all dependency setting, the query returns an array of all records that have been removed. Because it needs to collect all the objects and return them, this ends up making the query very slow. This is definitely not from delete_all, and this isn't really fast. So how do we fix this problem?</p>
<p>There's an easy and clear way to solve this. I find deleting records through an association to be risky and complicated. We already know what we want to delete so we can collect the categorizations through the category ID and delete them directly. The code is super clear, concise and benchmarks at remarkable speed for 10k records. It even produces the super simple delete statement we were expecting earlier.</p>
{% highlight ruby %}
# get the category
category = Category.where(name: "Networking").first

# find categorizations by category ID and delete directly
Categorization.where(category_id: category.id).delete_all

# benchmark data      
        user     system       total          real
 =>   0.010000   0.000000   0.010000 (  0.014286)


# sql produced
DELETE FROM `categorizations` WHERE `categorizations`.`category_id` = 1
{% endhighlight %}
<h3>Conclusion: What Have We Learned?</h3>
<ul>
<li>Active Record is a great tool, but we shouldn't let it's magical properties make us lazy.</li>
<li>Our assumptions about Active Record can have major consequences if we aren't paying close attention to how our queries are being translated into sequel. This is especially try for large datasets.</li>
<li>We can overcome most problems with Active Record by changing what we're asking it to do, and being more aware of the consequences of the code that we write.</li>
<li>There are some really great tools out there to help you profile your queries and memory usage.
<ul>
<li><a href="http://newrelic.com" target="_blank">New Relic</a> can help identify places where we might have slow SQL queries that are consuming our memory.</li>
<li><a href="https://github.com/MiniProfiler/rack-mini-profiler" target="_blank">MiniProfile</a> adds a badge to your application that records query times form the database.</li>
<li>And the <a href="https://github.com/flyerhzm/bullet" target="_blank">Bullet gem</a> identifies N+1 queries and let's you know when you should add eager loading</li>
</ul>
</li>
</ul>
<p>I hope you enjoyed this post and the related slideshow!</p>
