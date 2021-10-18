---
layout: post
title: Arel Primer
date: 2021-10-18 00:25:00 +5:30 GMT
share: y
---

## Arel Primer

One of the easiest things about Rails, is writing out SQL quereies without writing out SQL queries.

So instead of writing something like this,

```sql
SELECT "users".* FROM "users" WHERE "users"."role" = 'admin'
```

We can write something like this,

```ruby
User.where(role: :admin)
```

<!--break-->

This is much more readable and effectively abtracts us from writing any SQL.

What's really happening under the hood is that Rails is using the Arel library to build SQL queries. Arel is a powerful SQL AST manager that lets us appropriately combine selection statements for simple to very complicated queries.

However, *reader be cautioned* – Arel is still a private API provided by Rails. Meaning that future versions of Rails could be subject to changes in Arel. Fortunately, the changes have been limited to a minimum with the latest versions.

Let's take the same query and write it using Arel,

```ruby
User.arel_table['role'].eq('admin')
```

This gives us the Arel query which can be converted to SQL.

```ruby
User.arel_table['role'].eq('admin').to_sql
=> "users"."role" = 'admin'
```

This only gives us only a partial query, we now need to chain this to the table in SQL.

```ruby
User.where(User.arel_table['role'].eq('admin'))
```

Now calling `.to_sql` let's see what happens,

```ruby
User.where(User.arel_table['role'].eq('admin')).to_sql
=> SELECT "users".* FROM "users" WHERE "users"."role" = 'admin'
```

Ta da! 



### Advantages of Arel

- Reusability

  It's much easier to reuse Arel queries as they are made up of interlinking nodes – a safer alternative than string interpolation.

- Readability

  Since SQL queries are long strings, the more complex queries are the more unreadable it becomes. It also becomes harder to edit small portions of the query as the entire query has to be understood first.

- Reliability

  If we join to another table, our query will immediately break due to the ambiguity of the `id` column. Even if we qualify the columns with the table name, this will break as well if Rails decides to alias the table name.

- Repetition

  Often times we end up rewriting code that we already have as a scope on multiple classes.



### Arel 101

Arel is quite intuitive to use. Once we pickup on the basic structure, it gets easier to pile things on, called nodes. These nodes can be chained together to create complex queries.

Let's have a first look into how we can get started.

1. Arel Table

   Before we get started creating queries, we need to decide the base table to perform the queries on. 

   ```ruby
   users = User.arel_table
   ```

2. Arel Fields

   We can extract columns to work on as well.

   ```ruby
   users[:role]
   ```

3. Where Queries

   This is *where* things get really interesting! There are many things that can be done and all of them are listed in the [arel](https://github.com/rails/rails/tree/main/activerecord/lib/arel)/**predications.rb** file. Let's go through a couple of them.

   ```ruby
   users[:id].in([1,2,3]).to_sql
   => "`users`.`id` IN (1, 2, 3)"
   
   users[:id].gt(2).to_sql
   => "`users`.`id` > 2"
   
   users[:id].eq(3).to_sql
   => "`users`.`id` = 3"
   ```

   In order to combine it with the ActiveRecord `where` query, simply,

   ```ruby
   User.where(users[:id].gt(2))
   => #<ActiveRecord::Relation [#<User id: 3, email: ...]>
   ```

4. Projection Queries

   These are nothing but `select` queries. This would need to be executed directly as SQL and can not easily be combined with ActiveRecord queries.

   ```ruby
   users.where(users[:role].eq(:admin)).project(:email).to_sql
   => "SELECT email FROM `users` WHERE `users`.`role` = 2"
   ```

   In order to execute it,

   ```ruby
   ActiveRecord::Base.connection.exec_query users.where(users[:role].eq(:admin)).project(:email).to_sql
   # SQL (0.8ms)  SELECT email FROM `users` WHERE `users`.`role` = 2
   => <ActiveRecord::Result:0x00007fd7d5888178 @columns=["email"], @rows=[["adam@example.com"]], @hash_rows=nil, @column_types={}>
   ```

5. Aggregates

   The standard aggregates (sum, average, minimum, maximum and count) are also available.

   ```ruby
   users.project(users[:age].sum)
   users.project(users[:age].average)
   users.project(users[:id].count)
   ```

6. Order

   ```ruby
   users.order(users[:name], users[:age].desc)
   ```

7. Limit & Offset

   ```ruby
   users.take(5) 
   => 'SELECT * FROM users LIMIT 5'
   users.skip(4) 
   => 'SELECT * FROM users OFFSET 4'
   ```

8. Joins

   This is where Arel becomes really useful as it is easy to build complex SQL queries without resorting to writing it out.

   ```ruby
   photos = Photo.arel_table
   users.join(photos) 
   ```

   We can also specify the relationship,

   ```ruby
   users.join(photos, Arel::Nodes::OuterJoin).on(users[:id].eq(photos[:user_id]))
   ```

   We can also use conditions,

   ```ruby
   users.joins(:photos).merge(Photo.where(published: true))
   ```

This covers most scenarios faced in real life. However for a deep dive, I suggest to read the [documentation](https://www.rubydoc.info/gems/arel).



### Arel Compositions

Where Arel becomes really useful is when refactoring code to become more extensible. Let's take an example of a user belonging to multiple organizations in an application. In order to find out what organizations he belongs to we would do,

```ruby
Organization.where(
	Organization.arel_table[:id].in(
    UserOrganization.where(
      UserOrganization.arel_table[:user_id].eq(user.id)
    ).distinct.pluck(:organization_id)
  )
)
```

Now, if we need to find out all the organizations that the user has commented on, we would need to repeat the same,

```ruby
Organization.where(
	Organization.arel_table[:id].in(
    Comment.where(
      Comment.arel_table[:user_id].eq(user.id)
    ).distinct.pluck(:organization_id)
  )
)
```

This is repeated code! Let's see how we can refactor it.

Let's create a class called `Memberships`

```ruby
class Memberships < Struct.new(:user, :member)
  def all
    Organization.where(
      Organization.arel_table[:id].in(organization_ids)
    )
  end

  private
  def organization_ids
    member.where(
      member.arel_table[user_column].eq(user.id)
    ).distinct.pluck(:organization_id)
  end

  def user_column
    return :submitter_id if (member == BlogPost)
    
    :user_id
  end
end
```

And here's how we would use it,

```ruby
Memberships.new(User.first, UserOrganization).all
# User Load (0.4ms)  SELECT  `users`.* FROM `users` ORDER BY `users`.`id` ASC LIMIT 1 
# (6.5ms)  SELECT DISTINCT `user_organizations`.`organization_id` FROM `user_organizations` WHERE `user_organizations`.`user_id` = 1
# Organization Load (1.1ms)  SELECT  `organizations`.* FROM `organizations` WHERE `organizations`.`id` IN (1) LIMIT 11
=> <ActiveRecord::Relation [<Organization id: 1, ... ]>
```



Now if we want to find the organizations that the user has commented on,

```ruby
Memberships.new(User.first, Comment).all
# User Load (0.6ms)  SELECT  `users`.* FROM `users` ORDER BY `users`.`id` ASC LIMIT 1
# (8.9ms)  SELECT DISTINCT `comments`.`organization_id` FROM `comments` WHERE `comments`.`user_id` = '1'
# Organization Load (0.4ms)  SELECT  `organizations`.* FROM `organizations` WHERE `organizations`.`id` IN (1) LIMIT 11
=> <ActiveRecord::Relation [<Organization id: 1, ... ]>
```

