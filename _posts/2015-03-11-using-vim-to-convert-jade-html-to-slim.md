---
layout: post
title: "Using Vim to convert Jade html to Slim"
date: 2015-03-11
image: /images/jade_to_slim_vim_script.tiff
tags: vim rails
---

For every Rails developers, dealing with erb template engine is messy and unconfortable. They usually choose to use other template engine such as harm, slim ... However, designers may attach themselves in Node.js environment so they tend to use jade template engine. Using Vim and a little bit regex substitution can help you convert a jade file to slim file really easily.

## Differences between jade and slim we can deal with
  1. Slim doesn't use `,` (comma) to separate attributes as jade, it uses space.
  2. Slim prefers to define attributes after id and classes.

     Slim way:

     ```
     div#id.classes data-attr1="something" data-attr2="something"
     ```
     Jade can be like this:

     ```
     div#id(data-attr1="something", data-attr2="something").classes
     ```
  3. Jade allows non-value attributes, Slim doesn't

     Slim way:

     ```
     div#id.classes data-attr1="true"
     ```

     Jade way:

     ```
     div#id.classes data-attr1
     ```

## Using Vim to turn a Jade file to Slim
  1. Move all classes and id to front of attributes

    ```
    :%s/\((.\+)\)\(\.[^ ]*\)/\2 \1/g
    ```
  2. Remove commas between attributes

    ```
    :%s/\(".\{-}"\|data-\w\+\),/\1 /g
    ```
  3. Remove parentheses around attributes

    ```
    :%s/(\(.\{-}=".\{-}"\(\s\+.\{-}=*\(".\{-}"\)*\)*\))/ \1/g
    ```

## Result
From this:

```html
link(rel="stylesheet", href="css/ads.css")

div(style="width: 640px; margin: 20px auto").cl-wr
  .cl-ab
    a(href="index.html", target="_blank") Recommended by
  ul.cl-awi.cl-gr-3
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/1.jpg)")
        h4.cl-t [MWC 2015] HTC RE có phụ kiện mới: Đế khủng long, gậy tự sướng, cáp cứng...
        p.cl-s Tinh tế
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/2.jpg)")
        h4.cl-t [MWC 2015] Trên tay Galaxy S6 Edge màn hình cong 2 phía và S6 thường: nhẹ, đẹp, cao cấp
        p.cl-s Tinh tế
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/3.jpg)")
        h4.cl-t [MWC 2015] Trên tay HTC One M9: Máy đẹp, cấu hình tốt, phần mềm ngon
        p.cl-s Tinh tế
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/4.jpg)")
        h4.cl-t [MWC 2015] HTC ra mắt Grip - vòng đeo tay theo dõi sức khoẻ, màn hình cong, có GPS, 199$
        p.cl-s Tinh tế
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/5.jpg)")
        h4.cl-t [MWC 2015] Đang tường thuật sự kiện ra mắt Samsung Galaxy S6 / S6 Edge
        p.cl-s Tinh tế
    li
      a(href="", target="_blank")
        .cl-i(style="background-image: url(images/ads/6.jpg)")
        h4.cl-t Rò rỉ hình ảnh Samsung Galaxy S6 Edge
        p.cl-s Tinh tế
  .cl-ab
    a(href="index.html", target="_blank") Recommended by
```

to this:

```html
link rel="stylesheet"  href="css/ads.css"

div.cl-wr  style="width: 640px; margin: 20px auto"
  .cl-ab
    a href="index.html"  target="_blank" Recommended by
  ul.cl-awi.cl-gr-3
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/1.jpg)"
        h4.cl-t [MWC 2015] HTC RE có phụ kiện mới: Đế khủng long, gậy tự sướng, cáp cứng...
        p.cl-s Tinh tế
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/2.jpg)"
        h4.cl-t [MWC 2015] Trên tay Galaxy S6 Edge màn hình cong 2 phía và S6 thường: nhẹ, đẹp, cao cấp
        p.cl-s Tinh tế
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/3.jpg)"
        h4.cl-t [MWC 2015] Trên tay HTC One M9: Máy đẹp, cấu hình tốt, phần mềm ngon
        p.cl-s Tinh tế
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/4.jpg)"
        h4.cl-t [MWC 2015] HTC ra mắt Grip - vòng đeo tay theo dõi sức khoẻ, màn hình cong, có GPS, 199$
        p.cl-s Tinh tế
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/5.jpg)"
        h4.cl-t [MWC 2015] Đang tường thuật sự kiện ra mắt Samsung Galaxy S6 / S6 Edge
        p.cl-s Tinh tế
    li
      a href=""  target="_blank"
        .cl-i style="background-image: url(images/ads/6.jpg)"
        h4.cl-t Rò rỉ hình ảnh Samsung Galaxy S6 Edge
        p.cl-s Tinh tế
  .cl-ab
    a href="index.html"  target="_blank" Recommended by
```

## Making it a script for reusage

  Open new file to write the script to, you can also open vim history to get all of our commands

  The file would look like this:

  ```
  function! JadeToSlimFunction()
    silent! %s/\("\w*"\|data-\w\+\),/\1 /g
    silent! %s/\(".\{-}"\|data-\w\+\),/\1 /g
    silent! %s/(\(.\{-}=".\{-}"\(\s\+.\{-}=*\(".\{-}"\)*\)*\))/\1/g
    silent! %s/(\(.\{-}=".\{-}"\(\s\+.\{-}=*\(".\{-}"\)*\)*\))/ \1/g
  endfunction

  command! JadeToSlim call JadeToSlimFunction()
  ```

  You can use this script on a Jade file to turn it to Slim with just:

  ```
  :JadeToSlim
  ```

## Conclusion
  It's always cool and fun to play with text especially with Vim. If you have heard of Vim but still don't dare to use this. Be brave, learn it. Once you have Vim in your inventory, you will forget all other editors.
