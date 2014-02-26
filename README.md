# css perf

**css-perf** is a completely unscientific way of testing CSS performance. Most of these tests will revolve around methodologies and techniques for determining effective CSS architecture. Put another way, I want to know what works best given a particular comparison of CSS strategies.



## Tests results

Explanation and analysis between sample pages featuring high number of elements.


### Attribute vs class selectors

This tests the rendering performance of the same page of 5,000 elements with a particular class. An attribute selector looks something like `[class^="column-"] { ... }`. A class selector looks like `.column-class { ... }`. **At first glance, the class selector page renders 43ms faster (a 9.3% improvement) in Safari 7.0.1.**

| Page                                                            | Render time |
|-----------------------------------------------------------------|-------------|
| [Attribute selectors](http://mdo.github.io/attribute-selectors/)| 485ms       |
| [Class selectors](http://mdo.github.io/class-selectors/)        | 442ms       |

The time spent recalculating styles is neglible I think, but the difference here *is* interesting. Consider the following context for this test:

- The test page contains only one element, with one class, and the same content.
- The CSS contains two selectors—the `body` and the repeated element. Most pages, and most sites/applications, don't work this way.
- In many situations, sites might have a few dozen or couple hundred repeated elements per page (think tables, buttons, icons, grids, etc). Each of those series will have it's own CSS, with any kind of CSS selector.

In other words, this *super* edge case comparison of a large number of the same elements isn't that accurate. I'd wager most sites would see a larger disparity when sticking to classes only, as opposed to wide use of attribute selectors.

**Conclusion?** Based on this test, my personal experience with Twitter, Bootstrap, and GitHub, and the feedback of others, I'd stick to classes for widely used components.


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

| Page                                                                      | Render time |
|---------------------------------------------------------------------------|-------------|
| [Standard box-sizing reset](http://mdo.github.io/box-sizing-reset/)       | 55.5ms      |
| [Split box-sizing reset](http://mdo.github.io/box-sizing-reset-separate/) | 47.3ms      |

The problem with this test is that the **page rendering time is super inconsistent.** The first test numbers are above. Subsequent refreshes yield wildly different numbers. Sometimes the render time is doubled or the improvement reversed between the two options.

**Conclusion?** I have no idea.



## Further tests to run

- [x] Box-sizing reset strategies
- [ ] Multiple attribute selectors per page
- [ ] Grids: floats vs inline-block vs table layout



## Feedback

Tell me I'm wrong, that I'm an idiot, or that I'm missing a test case you'd like to see. Let me know with an issue, or open a pull request. Whatever it is, just go for it.



## License

Released with <3 under MIT by Mark Otto.
