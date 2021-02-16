---
layout: post
title: "RSpec Rails Tip: `have_http_status` matcher"
date: 2021-02-16
description: "Tips on using the have_http_status matcher from rspec-rails"
---

Sometimes you need to test the status code of a response in a controller, request, or feature spec.
With the [`rspec-rails`][rspec-rails] gem this can be done cleanly using the [`have_http_status` matcher][rspec-have-http-status-docs].

`have_http_status` can take three different arguments:
1. numeric code (`200`, `404`, etc)
2. status name as defined in `Rack::Utils::SYMBOL_TO_STATUS_CODE` (`:ok`, `:not_found`, etc)
3. generic status type (`:success`, `:missing`, `:redirect`, or `:error`)

This means that these are actually different:

```rb
it { is_expected.to have_http_status(:ok) }
it { is_expected.to have_http_status(:success) }
```

Because `:ok`, as defined in [`Rack::Utils::SYMBOL_TO_STATUS_CODE`][rack-symbol-to-status-code-source], only matches status `200` while `:success` is a generic status type that matches all "successful" status types.

According to the [RSpec source code][rspec-have-http-status-source-code], the generic status types map to the following status codes:

```rb
case expected
when :error, :server_error
  "5xx"
when :success, :successful
  "2xx"
when :missing, :not_found
  "404"
when :redirect
  "3xx"
end
```

[rspec-rails]: https://github.com/rspec/rspec-rails
[rack-symbol-to-status-code-source]: https://github.com/rack/rack/blob/2.2.3/lib/rack/utils.rb#L577:L579
[rspec-have-http-status-source-code]: https://github.com/rspec/rspec-rails/blob/64b2712da9d12c03a582bb26b62337504f8d1b76/lib/rspec/rails/matchers/have_http_status.rb#L338:L347
[rspec-have-http-status-docs]: https://relishapp.com/rspec/rspec-rails/docs/matchers/have-http-status-matcher
