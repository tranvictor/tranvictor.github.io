 * AdBlock doesn't block script from particular domain, it doesn't care about script domain
 * AdBlock has bunch of css selector rules to match ad elements to hide
 * AdBlock will not be able to hide ad elements if the CSS path is not defined in the list



# Proposed solutions
  Main idea is to change ad markup id and class

  1. Do it on backend side (ruby)
    * change the id and css on html markup
    * change the id and css on embeded css
    * do this job on schedule
  2. Do it on frontend side (js)
    * pro
      * dont have to manage crob job on server side
      * reduce computation on server side
    * cons
      * trade off some display time (time for ad to display on the site) because we have to hide the ad, change DOM and CSS


# How do we change id and class
  1. Figure out a random seed key
  2. Map old ids, classes to new ids, classes based on the seed key
  3. Replace Id, classes on HTML and css to new Ids and classes (we can do it because in our css, classes and ids are totally separated, I mean there's no id or class which is a substring of another id or class)
