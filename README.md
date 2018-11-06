# Coll Proxy Issue

There is a issue with `CollectionProxy#any?`, I have not been able to figure it out.
If the following command is the first command after a load/reload in a development console or 
web server, it returns `nil` instead of `true`.
  ```ruby
    # This command is a simplification of what we use. 
    # We do not use `Person.all` nor do we return a `CollectionProxy`
    Person.all.any? { |r| Person.first.accounts }
  ```
In my testing I have found that this is only a problem if the first execution 
of the block must return truthy and involve a `CollectionProxy`. I have tested 
the on Rails 5.2.3 and Rails edge, the problem exists on both.


Here are the steps to reproduce the problem:
* Clone this repository
* Run the following in the cloned repository
  ```bash
  bundle install
  rails db:setup
  rails c
  ```
* Call the command `Person.all.any? { |r| Person.first.accounts }`

Here is a sample output:
```
Loading development environment (Rails 5.2.1)
irb(main):001:0> Person.all.any? { |r| Person.first.accounts }
  Person Load (1.2ms)  SELECT "people".* FROM "people"
  Person Load (0.3ms)  SELECT  "people".* FROM "people" ORDER BY "people"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> nil
irb(main):002:0> Person.all.any? { |r| Person.first.accounts }
  Person Load (0.2ms)  SELECT "people".* FROM "people"
  Person Load (0.1ms)  SELECT  "people".* FROM "people" ORDER BY "people"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> true
irb(main):003:0> reload!
Reloading...
=> true
irb(main):004:0> Person.all.any? { |r| Person.first.accounts }
  Person Load (0.1ms)  SELECT "people".* FROM "people"
  Person Load (0.1ms)  SELECT  "people".* FROM "people" ORDER BY "people"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> nil
irb(main):005:0> Person.all.any? { |r| Person.first.accounts }
  Person Load (0.1ms)  SELECT "people".* FROM "people"
  Person Load (0.1ms)  SELECT  "people".* FROM "people" ORDER BY "people"."id" ASC LIMIT ?  [["LIMIT", 1]]
=> true
```
  
