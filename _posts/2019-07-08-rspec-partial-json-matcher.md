---
layout: post
title:  "Custom RSpec Matcher For Partial JSON"
date:   2019-07-08
description: "How to write a custom RSpec JSON matcher for a string that is partially made up of JSON."
---

**Note:** In the example below we're probably testing things at the wrong level. The appropriate way to test this would likely be to test the generation of the data, the formatting of the data, and the logging of the message all independently from one another. However, for various reasons we made the trade off to not test each part individually.
{: .note}

We have a case where a message is logged which contains a string prefix followed by a JSON blob.

A typical log message looks something like this:

```
CustomExperiment {"experiment_name":"UserValidator","result":"mismatched","context":{},"control":{"value":"true","duration":1.0},"candidate":{"value":"false","duration":0.5},"execution_order":["candidate","control"]}
```

Our goal is to write tests that can make assertions against the contexts of the JSON blob.

To do this we defined a custom RSpec matcher and used the `include_json` JSON matcher from [`rspec-json_expectations`](https://github.com/waterlink/rspec-json_expectations) that looks like this:

```rb
require 'rspec/expectations'
require 'rspec/json_expectations'

RSpec::Matchers.define :experiment_log_json_including do |expected_json_format|
  match_unless_raises do |log_message|
    log_message_prefix = 'CustomExperiment '
    expect(log_message).to start_with(log_message_prefix)

    json_string = log_message.sub(log_message_prefix, '')
    expect(json_string).to include_json(expected_json_format)
  end
end
```

This allows us to write specs that look like this:
```rb
it 'logs the correct experiment name' do
  expect(Rails.logger).to have_received(:info)
    .with(experiment_log_json_including(experiment_name: 'UserValidator'))
end
```
