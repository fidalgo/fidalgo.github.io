---
layout: post
title: Migrate a Rails application to use UUID with PostgreSQL
subtitle: A tale about migrate an ongoing developing application to use UUID instead of Bigints for ids
tags: ruby rails uuid postgresql
---


UUID stands for Universally Unique IDentifier, a standard used to get unique identifiers for resources.
The major benefit of using UUID as identifier is that it can be generated without collisions in different systems, so we can have several systems
geneting IDs.

# What are UUIDS and when to use them

In the context of a Rails application, dropping a sequential identification, hides some internal details on the system, but also eases the migration and merging data
from different instances or datastores.
Imagine a case when you have several shards and you want to merge it later, for example you have a shard by year and after 5 years you want to move the oldest ones to an
archive database, that is barely accessed, you'll be able to merge the data without a problem, if you use UUIDS for IDs and foreign keys.
Another use case is when you have several databases, that will fed another systems, so you'll ensure that every record is unique.

Also ensures we don't exhaust all the integers and enters in [IDpocalypse](https://finance.yahoo.com/news/panic-now-twitter-run-numbers-193901005.html)


## Are UUIDS the silver bullet of identifiers

Long story, short: **There are no silver bullets! Period!**

The first drawback you will find, specially in a Rails application, is the lack of sorting. When you use a sequential number as ID you have a primary way to sort, that matches the
insertion time. So when moving to UUIDS you need to ensure that you have a proper sort, usually by using `created_at`.

Another one is the size, with an Integer you have 4 bytes instead of 36 from the UUIDS, this matters in terms of space for the `id` field itself and the indexes, since we're indexing a
String.

Also you cannot use UUIDs for sorting, since they are randomly generated, so the fields using UUIDs cannot be used for sharding data, but surely there's a lot of other possibilities to
be the source of sharding.


# How to use UUIDs in Ruby on Rails 5

When [Ruby on Rails 5.0 was released](http://guides.rubyonrails.org/5_0_release_notes.html), it introduced the possibility to use UUIDs as primary keys, when generating database migrations. More info about it in [this pull request](https://github.com/rails/rails/pull/21762)

If you're using PostgreSQL >= 9.4, ActiveRecord will use the `gen_random_uuid` function from `pgcrypto` extension.

## Add a migration

To enable the extension we need to create a migration:

`$ rails generate migration enable_pgcrypto_extension`

Then change the migration to something alike this:

```ruby
class EnablePgcryptoExtension < ActiveRecord::Migration[5.1]
  def change
    enable_extension 'pgcrypto' unless extension_enabled?('pgcrypto')
  end
end
```

## Configure generators

In your `config/application.rb` file, add:

```ruby
class Application < Rails::Application
  config.active_record.primary_key = :uuid

  config.generators do |g|
    g.orm :active_record, primary_key_type: :uuid
    g.orm :active_record, foreign_key_type: :uuid
  end
end
```

Now if you generate a new migration

`$ rails generate model post title:string`

you'll have something like this:

```ruby
class CreatePosts < ActiveRecord::Migration[5.1]
  def change
    create_table :posts, id: :uuid do |t|
      t.string :title

      t.timestamps
    end
  end
end
```

# How to deal with existing migrations

## The dirty way

For the most applications, you'll need to handle this differently, because I'm going against the conventions on Rails.

Since my application is still on development, so no clients (no clients, no money) and data, so I can afford to drop the databases and recreate the migrations.

So I've ended up adding a new migration to enable the PostgreSQL extension, like mentioned before, and changed the `create_table` arguments to add `id: :uuid`.

But there's something worth to share, regarding the foreign keys: you need to set the type, otherwise it will be assumed as `serial`.

So if you have an Order model, you'll need to end up with something like:

```ruby
class CreateOrders < ActiveRecord::Migration[5.1]
  def change
    create_table :orders, id: :uuid do |t|
      t.belongs_to :customer, type: :uuid, index: true
      t.decimal :total, precision: 10, scale: 2, null: false
      t.text :note
      t.timestamps
    end
  end
end
```
in this case, the `belongs_to` specifies the foreign key, and since the Costumer `id` has the `uuid` type, we need to be consistent when reference it.

## The Rails way

To deal with the existing migrations, in the Rails way, you need to create more migrations, to change the `id`, `references`, `belongs_to` columns to `uuid` type.

# Caveats

Beware that `.first` and `.last` methods will lead to inconsistent results, because each method means lowest and highest ID, respectively.
You can circumvent this by adding a default scope to all records, so in you `app/models/application_record.rb` file change to:

```ruby
class ApplicationRecord < ActiveRecord::Base
  self.abstract_class = true
  default_scope { order(created_at: :asc) }
end
```


# Conclusions

It's very easy to setup the UUIDs in a Ruby on Rails application when you're using PostgreSQL, and you're avoiding some pains of data merge in the future.
Surely you loose the ability to easily identify your records, but it would pay off in the end of day.
Also, since you're using your native database functions, the performance penalty shouldn't be noticeable, but that's an exercise you need to do in order to evaluate if it's worth or not... and in case you want to try it, I hope I've helped you to setup it.
