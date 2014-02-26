# css perf

**css-perf** is a completely unscientific way of testing CSS performance. Most of these tests will revolve around methodologies and techniques for determining effective CSS architecture. Put another way, I want to know what works best given a particular comparison of CSS strategies.



## Table of contents

- [Attribute vs class selectors](#attribute-vs-class-selectors)
- [Box-sizing resets](#box-sizing-resets)
- [Grid techniques](#grid-techniques)
- [background vs background-color](#background-vs-background-color)



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


## Tests results

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

## Further tests to run

- Box-sizing reset strategies
- Multiple attribute selectors per page



## Feedback

Tell me I'm wrong, that I'm an idiot, or that I'm missing a test case you'd like to see. Let me know with an issue, or open a pull request. Whatever it is, just go for it.



## License

Released with <3 under MIT by Mark Otto.
