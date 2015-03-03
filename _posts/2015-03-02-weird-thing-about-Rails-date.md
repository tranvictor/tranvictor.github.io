---
layout: post
title: "Weird thing about Rails date api"
date: 2015-03-02
tags: rails
image: "/images/date_api.png"
---

Rails has excellent support for timezone which brings developers conveniences working with time. Two of the most well-known API for `date` are `Date.tomorrow` and `Date.yesterday`.

About a month ago, I got into a problem that I had to use those APIs to determine a particular time point belongs to yesterday, today, or tomorrow. At that time, I thought about 3 APIs `Date.yesterday`, `Date.today` and `Date.tomorrow` with no doubt. They must have worked correctly as they sound. However, they actually dont!

## Tomorrow can be equal to Today!

It's weird to see that `Date.yesterday` is equal to `Date.today` which makes no sense but it's true. Open your `rails c`:

```ruby
# I set in my timezone to Pacific Time (US & Canada) on rails config file
# But my system time zone is UTC +7

2.1.3 :001 > Time.zone
 => #<ActiveSupport::TimeZone:0x007f8446262380 @name="Pacific Time (US & Canada)", @utc_offset=nil, @tzinfo=#<TZInfo::TimezoneProxy: America/Los_Angeles>, @current_period=nil>
2.1.3 :002 > Date.today
 => Mon, 02 Mar 2015
2.1.3 :003 > Date.current
 => Sun, 01 Mar 2015
2.1.3 :004 > Date.yesterday
 => Sat, 28 Feb 2015
2.1.3 :005 > Date.tomorrow
 => Mon, 02 Mar 2015
```

As you can see there, `Date.today` and `Date.tomorrow` are equal. Why? It turns out that Ruby actually has `Date.today` API and Rails team didn't want to redefine this method, they created new method called `Date.current` so `Date.current`, `Date.yesterday` and `Date.tomorrow` are consistent but `Date.today`.

`Date.today` doesn't respect Rails timezone configuration, it uses System timezone by default.

`Date.current` on the other hand is defined by Rails thus it uses Rails timezone.

## Conclusion

Working with Rails, always remember to use `Date.current` with `Date.yesterday` and `Date.tomorrow` instead of `Date.today`.
