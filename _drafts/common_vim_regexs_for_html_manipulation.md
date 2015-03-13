1. select id attribute from any html tag
  `/ id="([^"]+)"`, id must be followed by a space, this regex refuses all empty id
2. select id attribute from particular tag
3. select class from any html tag
4. select class from particular tag
5. swap variables expression in ruby
:s/\([a-zA-Z0-9_]\+\((.\{-})\)\?\)\s*\([+\-\*]\)\s*\([a-zA-Z0-9_]\+\((.\{-})\)\?\)\s*/\4 \3 \1/
result:
a + b => b + a
a() + b => b + a()
a + b() => b() + a
a() + b() => b() + a()
