---
layout: post
title: "Capybara Tip: Disabling CSS Animations"
date: 2020-07-06
description: "How to configure Capybara to disable CSS animations and transitions"
---

You can disable CSS animations and transitions globally by setting the [`Capybara.disable_animation` option](capybara-config-docs).

Stick this in your `rails_helper.rb`:
```rb
Capybara.disable_animation = true 
```

Instead of a boolean, you can set this option to a CSS selector and it will only disable animations that match that selector:

```rb
Capybara.disable_animation = '#with_animation a'
```

For a better understanding of how this works take a look at [the tests for `Capybara.disable_animation` in the Capybara source code](capybara-disable-animation-tests).


[capybara-config-docs]: https://www.rubydoc.info/gems/capybara/Capybara/SessionConfig#disable_animation-instance_method
[capybara-disable-animation-tests]: https://github.com/teamcapybara/capybara/blob/d76728fe9551106af1211ed8b670c851f47a7976/spec/shared_selenium_session.rb#L362:L438

