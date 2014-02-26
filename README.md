# css perf

**css-perf** is a completely unscientific way of testing CSS performance. Most of these tests will revolve around methodologies and techniques for determining effective CSS architecture. Put another way, I want to know what works best given a particular comparison of CSS strategies.



## Table of contents

- [How "testing" is done](#how-testing-is-done)
- [Original test results](#test-results)
  - [Attribute vs class selectors](#attribute-vs-class-selectors)
  - [Box-sizing resets](#box-sizing-resets)
  - [Grid techniques](#grid-techniques)
  - [background vs background-color](#background-vs-background-color)
- [Average test results](#average-test-results)
  - [Safari 7.0.1](#safari-701)
  - [Chrome 33](#chrome-33)
  - [Differences between original test results](#differences-between-test-results)
  - [Updated conclusions from averages](#updated-conclusions-from-averages)
  - [A note on CSS performance testing](#a-note-on-css-performance-testing)
- [Feedback](#feedback)
- [License](#license)

## How "testing" is done

Yes, it's in quotes for a reason. None of this is *super* accurate, but it's all very interesting to me. I merely use page loads at this point to gauge overall time spend on a particular page by Safari and Chrome.

More specifically:

* I open the page, open the Inspector, and reload the page.
* In Chrome, I have the cache disabled when Inspector is open. I also Cmd-Shift-R for a full refresh. (Unsure about doing this Safari—there's no clear preference.)
* In Safari 7.0.1, I use the computed time after the finished page load from the top middle of the Inspector.
* In Chrome 33, I use the Timeline pane's Event view for a final computed time.

Things worth noting about this process:

* None of this is super accurate.
* These are *local* page load times via `file:///` URLs.
* Nothing is averaged. This is a single page load.
* The only platform tested is OS X (curently 10.9.1).
* I'm no developer tools expert (in any browser).
* Firefox's dev tools are horrible to use and I haven't included them in testing for now.

Additional context and background information is provided in the test results sections below.


## Original test results

Explanation and analysis between sample pages featuring high number of elements.

---

### Attribute vs class selectors

This tests the rendering performance of the same **page of 5,000 elements with the same class**. An attribute selector looks something like `[class^="column-"] { ... }`. A class selector looks like `.column-class { ... }`. **At first glance, the class selector page renders 43ms faster (a 9.3% improvement) in Safari 7.0.1.**

| Page                                                            | Safari 7.0.1 | Chrome 33 |
|-----------------------------------------------------------------|--------------|-----------|
| [Attribute selectors](http://mdo.github.io/attribute-selectors/)| 485ms        | 260.17ms  |
| [Class selectors](http://mdo.github.io/class-selectors/)        | 442ms        | 244.37ms  |

The time spent recalculating styles is neglible I think, but the difference here *is* interesting. Consider the following context for this test:

- The test page contains only one element, with one class, and the same content.
- The CSS contains two selectors—the `body` and the repeated element. Most pages, and most sites/applications, don't work this way.
- In many situations, sites might have a few dozen or couple hundred repeated elements per page (think tables, buttons, icons, grids, etc). Each of those series will have it's own CSS, with any kind of CSS selector.

In other words, this *super* edge case comparison of a large number of the same elements isn't that accurate. I'd wager most sites would see a larger disparity when sticking to classes only, as opposed to wide use of attribute selectors.

**Conclusion?** Based on this test, my personal experience with Twitter, Bootstrap, and GitHub, and the feedback of others, I'd stick to classes for widely used components.

---

### Box-sizing resets

This tests the rendering performance of the (relatively new) standard `box-sizing: border-box;` reset. Popularized by Paul Irish with his [* { Box-sizing: Border-box } FTW](http://www.paulirish.com/2012/box-sizing-border-box-ftw/) post, it looks like this:

```css
*,
*:before,
*:after {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```

I've had a hunch that this is slower for one reason or another than say splitting up the selectors like so:

```css
* {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
*:before,
*:after {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```

And as it turns out, it is slower in the first run. **Splitting the selector up saves `8.2ms` (a 16% improvement) in page render time** as reported by a single load in Safari 7.0.1.

| Page                                                                      | Safari 7.0.1 | Chrome 33 |
|---------------------------------------------------------------------------|--------------|-----------|
| [Standard box-sizing reset](http://mdo.github.io/box-sizing-reset/)       | 55.5ms       | 108.61ms  |
| [Split box-sizing reset](http://mdo.github.io/box-sizing-reset-separate/) | 47.3ms       | 98.87ms   |

The problem with this test is that the **page rendering time is super inconsistent.** The first test numbers are above. Subsequent refreshes yield wildly different numbers in both Safari 7.0.1 and Chrome 33. Sometimes the render time is doubled or the improvement reversed between the two options.

**Conclusion?** I have no idea.

---

### Grid techniques

This test compares three CSS grid techniques: `float`s, `display: inline-block;`, and `display: table-cell;`. The test page has a few hundred columns in standard layouts, so it's a super basic set of pages. The only differences are in the actual grid CSS.

| Page                                                    | Safari 7.0.1 | Chrome 33 |
|---------------------------------------------------------|--------------|-----------|
| [Floats](http://mdo.github.io/grid-floats/)             | 246ms        | 424.94ms  |
| [Inline-block](http://mdo.github.io/grid-inline-block/) | 306ms        | 439.19ms  |
| [Flexbox](http://mdo.github.io/grid-flexbox/)           | 252ms        | 262.41ms  |
| [Tables](http://mdo.github.io/grid-tables/)             | 271ms        | 265.53ms  |

Some background and context:

- Use floats is super straighfoward—float the columns, clear them with a row. I honestly don't see why folks *don't* love floated grid systems more.
- `inline-block` grids require some `white-space: nowrap;` or `font-size: 0;` hackery to collapse whitespace. Why bother resetting something at such a high level in your page?
- Flexbox is interesting, and this test doesn't make full use of available flexbox properties for the columns.
- Tables are super impractical for grids because there is no perf gain when using `table-layout: fixed;`, a property that tells the browser to only scrape a table's first row of cells to determine *every* cell's width for super fast rendering.

**Conclusion?** Floats have been CSS stable for many years, and I see no reason to move away. That said, I have no idea why the numbers are so crazy different between Safari and Chrome—especially for `float`s and `inline-block`.

---

### Background vs background-color

Comparison of 18 color swatches rendered 100 times on a page as small rectangles, once with `background` and once with `background-color`.

| Page                                                       | Safari 7.0.1 | Chrome 33 |
|------------------------------------------------------------|--------------|-----------|
| [background](http://mdo.github.io/background/)             | 44.9ms       | 34.45ms   |
| [background-color](http://mdo.github.io/background-color/) | 87.5ms       | 69.34ms   |

While these numbers are from a single page reload, with subsequent refreshes the render times changed, but the *percent difference* was basically the same every time.

**That's a savings of almost 42.6ms, almost twice as fast, when using `background`** instead of `background-color` in Safari 7.0.1. Chrome 33 appears to be about the same.

This honestly blew me away because for the longest time for two reasons:

- I usually always argue for explicitness in CSS properties, especially with backgrounds because it can adversely affect specificity down the road.
- I thought that when a browser sees `background: #000;`, they really see `background: #000 none no-repeat top center;`. I don't have a link to a resource here, but I recall reading this somewhere.

**Conclusion?** Stick to `background`, I guess. Ugh.

---

## Average test results

This is a second set of test results, averaging 10 page load times in Safari 7.0.1 and Chrome 33. Cache was always disabled in both browsers with Inspector open. **All times are in milliseconds.**

### Safari 7.0.1

|`background`|`background-color`|`box-sizing`|`box-sizing split`|`attr selectors`|`class selectors`|`grid - floats`|`grid - inline-block`|`grid - table`|`grid - flexbox`|
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
|34.8|35.7|48.3|56.3|103|92.7|155|174|129|129|
|35.4|40.7|45.5|54.1|113|103|171|190|131|130|
|34.4|37.2|48.1|47.5|114|89.4|167|177|148|122|
|35.8|74.8|51.8|48.9|103|89.1|208|184|141|121|
|51.6|39.7|46.5|64.3|106|100|165|183|147|125|
|37.6|36|42.5|45.8|118|104|168|181|128|121|
|33.4|44.9|88.8|68.9|113|103|179|214|141|133|
|31.8|37.1|45.4|41.5|115|88.3|170|185|137|125|
|31.8|38|62.3|45|110|99.4|164|177|143|135|
|57.8|36.9|39.4|44.1|116|102|151|182|145|115|
|||||||||||
|38.44|42.1|51.86|51.64|111.1|97.09|169.8|184.7|139|125.6|

### Chrome 33

|`background`|`background-color`|`box-sizing`|`box-sizing split`|`attr selectors`|`class selectors`|`grid - floats`|`grid - inline-block`|`grid - table`|`grid - flexbox`|
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
|54.71|98.8|115.84|118.03|296.68|239.62|399.82|492.73|297.2|302.14|
|76.97|60.43|111.39|102.29|302.84|249.69|405.4|444.31|274.17|267.38|
|54.47|56.31|100.94|98.88|274.51|249.12|409.19|446.83|273.26|276.91|
|55.06|56.01|105.1|104.13|277.23|243.7|409.89|451.68|267.64|274.92|
|56.12|59.47|97.72|106.09|289.38|266.6|404.1|444.12|275.73|271.13|
|76.37|56.68|100.32|100.2|281.62|254.81|412.51|446.36|271.66|266.55|
|59.59|55.37|103.25|119.43|269.07|247.56|406.63|458.51|279.44|262.78|
|55.77|59.18|105.26|102.27|270.95|244.96|405.68|473.82|270.63|266.63|
|77.66|58.08|103.28|100.28|268.88|245.84|408.32|440.71|270.76|279.45|
|53.73|59.21|98.98|105.21|266.24|255.96|398.81|469.13|269.87|271.98|
|||||||||||
|62.045|61.954|104.208|105.681|279.74|249.786|406.035|456.82|275.036|273.987|

### Differences between test results

The differences in times between these averages and the individual tests earlier is astonishing. Further testing—without me at the helm—is definitely needed. Here's what's different:

- In the original tests (all still documented above), single page loads were used for each test. Each browser had it's cache disabled with the Inspector open during those tests.
- In the averaged tests, the same conditions applied—cache disabled, Inspector open. Larger bumps in times can be seen, but very few like the ones I saw originally.
- The above averages are from my second attempt at averaging. The first set (of three refreshes) produced much higher inconsistencies that matched the original tests. I cannot explain the change, save for the larger sample size.
- Ever after this larger average test run, I still see crazy differences in numbers. One part of this can be the cache. With the cache re-enabled, the numbers vary *even more* greatly with each refresh, in Safari and Chrome.

Bottom line, though? The disparity is not at huge as I saw earlier when averaged out, apparently.

### Updated conclusions from averages

Let's revisit the conclusions I had earlier from single tests against the averages:

- **Attribute vs class selectors:** Winner is still class selectors. *Only thing faster than classes is ids.*
- **Box-sizing reset:** All together or broken apart, they're still about the same. *This sounds way saner to me.*
- **Float techniques:** Apparently `display: table;` beats out `float`s now. *I think this is the case because we repeat the same grid examples several times, so there's less computation to do still given `table-layout: fixed;`.*
- **Background or background-color:** Safari says `background`, Chrome says whatever—I'm in the same boat. *I still lean towards specificity here as a more maintainable property in larger code bases.*

So, the averages match some of the earlier expectations. That last one is the most important since [it blew up on Twitter](https://twitter.com/mdo/status/438584284241084418). The difference is clearly not as stark when averaged out.

### A note on CSS performance testing

**These kind of tests are cheats and always going to be somewhat inaccurate from the real world.** They involve duplicating series of the same elements over and over to stress test a page. That's *useful*, but that's not everything. In the future, a more accurate real world example page is needed.


---

## Feedback

Tell me I'm wrong, that I'm an idiot, or that I'm missing a test case you'd like to see. Let me know with an issue, or open a pull request. Whatever it is, just go for it.



## License

Released with <3 under MIT by Mark Otto.
