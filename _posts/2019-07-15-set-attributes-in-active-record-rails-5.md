---
layout: post
title:  "Different Ways to Set Attributes in ActiveRecord (Rails 5)"
date:   2019-07-15
description: "Comparing the different ways to set attributes in ActiveRecord (Rails 5)"
---

Rails 5 has a rich API that allows you to update your `ActiveRecord` objects in several different ways. 
Some methods have slightly different behavior which can sometimes result in unexpected consequences so 
it’s important to understand their differences.

Over the years I came across [a really helpful cheat sheet outlining the various methods
available for Rails 4][rails-4-post] so I figured I'd put together something similar for Rails 5!

Below is a cheat sheet with in-depth information for Rails 5.

# Cheat Sheet

| Method                                    | Uses Default Accessor | Saves to Database | Runs Validations | Runs Callbacks | Updates [`updated_at/on`][timestamps] | Respects Readonly |
| :---------------------------------------- | :-------------------- | :---------------- | :--------------- | :------------- | :------------------- | :------------- |
| [`attribute=`][attribute=]                | Yes                   | No                | _n/a_{: .na }    | _n/a_{: .na }  | _n/a_{: .na }        | _n/a_{: .na }  |
| [`attributes=`][attributes=]              | Yes                   | No                | _n/a_{: .na }    | _n/a_{: .na }  | _n/a_{: .na }        | _n/a_{: .na }  |
| [`assign_attributes`][assign_attributes]  | Yes                   | No                | _n/a_{: .na }    | _n/a_{: .na }  | _n/a_{: .na }        | _n/a_{: .na }  |
| [`write_attribute`][write_attribute]      | No                    | No                | _n/a_{: .na }    | _n/a_{: .na }  | _n/a_{: .na }        | _n/a_{: .na }  |
| [`[]=`][Hash=]                            | No                    | No                | _n/a_{: .na }    | _n/a_{: .na }  | _n/a_{: .na }        | _n/a_{: .na }  |
| [`update`][update]                        | Yes                   | Yes               | Yes              | Yes            | Yes                  | Yes            |
| [`update_attribute`][update_attribute]    | Yes                   | Yes               | No               | Yes            | Yes                  | Yes            |
| [`update_attributes`][update_attributes]  | Yes                   | Yes               | Yes              | Yes            | Yes                  | Yes            |
| [`update_column`][update_column]          | Yes                   | Yes               | No               | No             | No                   | Yes            |
| [`update_columns`][update_columns]        | Yes                   | Yes               | No               | No             | No                   | Yes            |
| [`User.update`][User.update]              | Yes                   | Yes               | Yes              | Yes            | Yes                  | Yes            |
| [`User.update_all`][User.update_all]      | No                    | Yes               | No               | No             | No                   | No             |
{: .compact-table.ar-attributes-table }

[rails-4-post]: https://davidverhasselt.com/set-attributes-in-activerecord/
[rails-6-post]: 2020-01-30-set-attributes-in-active-record-rails-6.md

[attribute=]: https://apidock.com/rails/ActiveRecord/AttributeMethods/Write/attribute=
[attributes=]: https://api.rubyonrails.org/v5.2/classes/ActiveModel/AttributeAssignment.html#method-i-attributes-3D
[assign_attributes]: https://api.rubyonrails.org/v5.2/classes/ActiveModel/AttributeAssignment.html#method-i-assign_attributes
[write_attribute]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/AttributeMethods/Write.html#method-i-write_attribute
[Hash=]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/AttributeMethods.html#method-i-5B-5D-3D
[update]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence.html#method-i-update
[update_attribute]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence.html#method-i-update_attribute
[update_attributes]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence.html#method-i-update_attributes
[update_column]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence.html#method-i-update_column
[update_columns]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence.html#method-i-update_columns
[User.update]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-update
[User.update_all]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Relation.html#method-i-update_all

[timestamps]: https://api.rubyonrails.org/v5.2/classes/ActiveRecord/Timestamp.html
