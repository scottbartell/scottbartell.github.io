---
layout: post
title:  "Different Ways to Set Attributes in ActiveRecord (Rails 6)"
date:   2020-01-30
description: "Comparing the different ways to set attributes in ActiveRecord (Rails 6)"
---

Rails 6 has a rich API that allows you to update your `ActiveRecord` objects in several different ways. 
Some methods have slightly different behavior which can sometimes result in unexpected consequences so 
it’s important to understand their differences.

**Note:** This article was written for Rails 6. See cheat sheets for other versions here:
{: .note}
* [Rails 3][rails-3-post] (external link)
* [Rails 4][rails-4-post] (external link)
* [Rails 5][rails-5-post]
* [Rails 7][rails-7-post]
{: .note}

Here's a cheat sheet highlighting the differences between all the methods available for setting attributes in Rails 6:

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
| [`update_attributes`][update_attributes][^update_attributes_deprecated] | Yes                   | Yes               | Yes              | Yes            | Yes                  | Yes            |
| [`update_column`][update_column]          | Yes                   | Yes               | No               | No             | No                   | Yes            |
| [`update_columns`][update_columns]        | Yes                   | Yes               | No               | No             | No                   | Yes            |
| [`User.update`][User.update]              | Yes                   | Yes               | Yes              | Yes            | Yes                  | Yes            |
| [`User.update_all`][User.update_all]      | No                    | Yes               | No               | No             | No                   | No             |
| [`User.upsert`][User.upsert]              | No                    | Yes               | No               | No             | No                   | No             |
| [`User.upsert_all`][User.upsert_all]      | No                    | Yes               | No               | No             | No                   | No             |
| [`User.insert`][User.insert]              | No                    | Yes               | No               | No             | No                   | No             |
| [`User.insert_all`][User.insert_all]      | No                    | Yes               | No               | No             | No                   | No             |
{: .compact-table.ar-attributes-table }

---

[rails-3-post]: https://davidverhasselt.com/5-ways-to-set-attributes-in-activerecord-in-rails-3/
[rails-4-post]: https://davidverhasselt.com/set-attributes-in-activerecord/
[rails-5-post]: 2019-07-15-set-attributes-in-active-record-rails-5.md
[rails-7-post]: 2022-04-12-set-attributes-in-active-record-rails-7.md

[attribute=]: https://apidock.com/rails/ActiveRecord/AttributeMethods/Write/attribute=
[attributes=]: https://api.rubyonrails.org/v6.0/classes/ActiveModel/AttributeAssignment.html#method-i-attributes-3D
[assign_attributes]: https://api.rubyonrails.org/v6.0/classes/ActiveModel/AttributeAssignment.html#method-i-assign_attributes
[write_attribute]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/AttributeMethods/Write.html#method-i-write_attribute
[Hash=]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/AttributeMethods.html#method-i-5B-5D-3D
[update]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence.html#method-i-update
[update_attribute]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence.html#method-i-update_attribute
[update_attributes]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence.html#method-i-update_attributes
[update_column]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence.html#method-i-update_column
[update_columns]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence.html#method-i-update_columns
[User.update]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-update
[User.update_all]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Relation.html#method-i-update_all
[User.upsert]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-upsert
[User.upsert_all]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-upsert_all
[User.insert]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-insert
[User.insert_all]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Persistence/ClassMethods.html#method-i-insert_all

[timestamps]: https://api.rubyonrails.org/v6.0/classes/ActiveRecord/Timestamp.html
[update_attributes_deprecated_source]: https://github.com/rails/rails/commit/5645149d3a27054450bd1130ff5715504638a5f5

[^update_attributes_deprecated]: `update_attributes` [has been deprecated][update_attributes_deprecated_source] in favor of `update` in Rails 6.
