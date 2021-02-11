---
layout: post
title: "Capybara Tip: Current Path Matcher"
date: 2021-02-11
description: "Assert you are on a given path with waiting/retrying built in"
---

Sometimes within a Capybara test you need to verify that you are on a particular page. The obvious 
way to test for this is by making expectations against [`current_path`][capybara-current-path] or [`current_url`][capybara-current-url]. 

The problem with this approach is that the previous action that was taken (such as a button click, or some fancy JS / `History.pushState()`) 
may not have completed when your expectation runs against `current_path`.

One way to address this is to add an expectation that has waiting logic built into it before the expectation 
against `current_path`:

```rb
expect(page).to have_content('Login')
expect(current_path).to eq('/login') 
```

An even better way to do this is to use [`have_current_path`][capybara-have-current-path-docs] which has 
waiting/retry logic built in so you don’t need to remember to always add an extra expectation with waiting above it.

```rb
expect(page).to have_current_path('/login')
```


[capybara-current-path]: https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session:current_path
[capybara-current-url]: https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Session:current_url
[capybara-have-current-path-docs]: https://www.rubydoc.info/github/teamcapybara/capybara/master/Capybara/RSpecMatchers#have_current_path-instance_method
