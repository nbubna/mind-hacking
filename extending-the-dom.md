Before you read this post, you should definitely [read this post][read], in which [kangax][kangax] skillfully explains the risks and pitfalls of extending "host" objects, particularly as experienced by the Prototype library. It both proceeds and concludes with strong warnings against the practice, allowing for it only in careful polyfills and specific, "controlled" environments.

[read]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/
[kangax]: http://twitter.com/kangax

It was good advice, but things have changed a great deal in the last three years. The DOM is no longer the dark and dangerous world of cross-browser challenges that it was. It's time to start re-evaluating the risks and benefits:


### Past Risks Of Extending The DOM
#### [Lack of specification (prototype exposure not guaranteed)][Lack]

[Lack]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#lack_of_specification

Guarantees and specifications are important. But the reality is that prototype exposure for the DOM has become remarkably consistent and ubiquitous in modern browsers. As long as you are willing to sacrifice the rapidly fading market share of IE < 9 and the like, there is little to fear here.

#### [Host objects have no rules (unpredictable interactions)][Host]

[Host]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#host_objects_have_no_rules

This argument was always weak and gets weaker as browsers (especially IE) rapidly improve, because even when they weren't improved, those creating wrapper libraries also had to watch for this. To say, "extending DOM objects is kind of like walking in a minefield" is to say the same about directly using the DOM, whether or not you extend it. In any case, all but one of the examples are for IE, presumably older versions. The remaining example (overriding `target` on events) is hardly difficult to avoid. I'm not even sure why you'd want to do that.

The bottom line is that these particular minefields have grown dramatically less dangerous to navigate as vendors have been working to clear them and everyone learns the trouble spots. It never hurts to step lightly when you leave the well-trod paths, but by all means, it's time to relax a little here.

#### [Chance of collisions (hard to scale APIs safely)][Chance]

[Chance]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#chance_of_collisions

IE8 and earlier conflated element properties and attributes, as IE8 disappears, so do the problems this raised. However, the browser is admittedly becoming a richer environment, more features, more libraries, more API surface occupied. But the expansion is not even; some notably different naming patterns have evolved. Specifically, those developing the DOM tend to choose names quite unlike those developing with the DOM.

The `querySelectorAll` function is probably the perfect example of the difference in naming philosophy. In most JavaScript frameworks, this is wrapped to be something that values conciseness or readability over technical descriptiveness. We would use something like `find`, a name that i'd wager is extremely unlikely to ever conflict with future DOM specifications. Similar safe names are usually easy to find, and future DOM features are not often hard to avoid. Of course, technical descriptiveness and readable conciseness most certainly do overlap, but when they do, it almost always means that the DOM feature is what you need (or close enough). Just limit yourself to a conditional polyfill in such cases. I won't deny that it can be a bit of a dance, but it is a very easy one to learn.

The other conflict area is with named forms and named fields therein. Named forms are automatically exposed on the `document` object. This is, in my opinion, still a very good reason to avoid extending the `document` object itself, at least not with any single-word name, as those are more likely to be form names. Named form fields being made available on the form element itself is a much more difficult challenge. However, this is no more a problem for DOM extenders than it is for wrappers. Try calling `hide()` on a jQuery-wrapped `<form><input name="style"></form>` and you'll see what i mean. No matter what approach you use, you must always take special care and consideration when interacting with a `<form>` element. Here there be dragons!

#### [Performance overhead (manual extension doesn't scale)][Performance]

[Performance]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#performance_overhead

Since we are addressing the risk for modern browsers and kangax's discussion of manual extension presumes the 
technique to be a workaround for older browsers with poor DOM prototype exposure, it is tempting to dismiss this out of hand. However, manual extension is still a legitimate technique for any [library][library] trying to "step lightly" and not employ the blanket coverage provided by prototype extension. The good news here is that when you are stepping lightly, you are extremely unlikely to be extending so much or so often that you need to worry about performance. The modern DOM needs far fewer extensions than it used to, especially if you try to restrain your extensions to [powerful utilities][powerful] instead of clusters of [simplistic aliases][simplistic].

[library]: http://nbubna.github.io/HTML
[powerful]: http://nbubna.github.io/HTML#each(property)
[simplistic]: https://github.com/jquery/jquery/blob/master/src/event-alias.js#L1-3

#### [IE DOM is a mess (memory leaks and attr/prop confusion)][IE]

[IE]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#ie_dom_is_a_mess

The conflating of properties and attributes stopped after IE8. And the [famous memory leaks were largely resolved][famous] earlier than that. Moving on...

[famous]: http://stackoverflow.com/questions/1999840/javascript-circular-references-and-memory-leaks

#### [Browser bugs (known flaws in IE8 and  older)][Browser]

[Browser]: http://perfectionkills.com/whats-wrong-with-extending-the-dom/#bonus_browser_bugs

Bugs will always be there for those stuck in the past, but in the present, they are closely hounded Wrappers and extenders alike ought still keep a sharp eye out, especially in bleeding-edge features, but the core DOM APIs are now regularly battle-tested by hordes of barbarian code all over the planet. 

#### So What Risk Is Left?

Not too much. I believe the list of risks particular to DOM extenders has been reduced to these very manageable risks:

* New code being opened in old browsers
* Name conflicts with future/proprietary DOM enhancements
* Name conflicts with named forms

Dealing with older browsers in this case means not falling back to increasingly desperate or risky measures but directing them to upgrade instructions or a reduced-functionality version. Well-established techniques abound: conditional comments, [Modernizr][Modernizr], even server side response changes are legitimate.

[Modernizr]: http://modernizr.com/

Dealing with future/proprietary enhancements mostly just requires checking before extending the prototypes or the elements themselves. Easy peasy, just remember to do `if (!('name' in object))` instead of `if (object.name)`. And you may be surprised by [how thoroughly you can avoid trouble][how] when extending objects, if you feel so inclined.

[how]: https://gist.github.com/nbubna/6180113

Dealing with named forms means you avoid extending the `document` object, and when you break that rule, never use any name a form might have (i.e. `foo()` is dangerous, `getFoo()` is less so). 

### Benefits of Extending The DOM

#### Rich API That's Getting Richer

The modern DOM is a rich API, as capable as its wrappers, if not as concise. And it's getting richer all the time. Those writing wrappers rely on plugin authors to keep up with new features being added to the DOM, or simply leave it to application developers to unwrap the DOM when they need access. Extending the DOM gives you everything the DOM has to offer today, no waiting for new releases or new plugins. Yes, it doesn't protect you from functions that lack cross-browser success yet, but the need for this diminishes every month as more and more users switch to [evergreen browsers][evergreen]. Where features are missing, polyfills tend to appear in the wilds of GitHub much sooner than they do in wrappers like jQuery.

[evergreen]: http://tomdale.net/2013/05/evergreen-browsers/

#### Educating Developers

When application developers use jQuery, it is rare that they learn anything but jQuery. They tend to think in jQuery and put all their plugins on the `$` whether they need to be or not. They rarely learn to appreciate the [wealth of knowledge][wealth] out there that is not at api.jquery.com. While they're working actively on the DOM, they know very little about it. Libraries that extend the DOM enhance it instead of hiding it away, enhancing developer's familiarity with the DOM. More developers knowing the DOM, means more developers ready and able to fix bugs, innovate new features, and generally push things forward in the browser environment.

[wealth]: https://developer.mozilla.org/en-US/docs?menu

#### Trim The Fat

Wrapper APIs can be fairly small, [zepto][zepto] sneaks under 10Kb when you minify and gzip it. But a part of that 10Kb is devoted to just renaming the DOM interface, rather than adding features. Any way your slice it, wrapping one API with a completely alternate API comes at a cost. That cost is small when the API being wrapped is flawed and inconsistent, but as the flaws and inconsistencies get ironed out, the cost becomes ever less worthwhile. The bandwidth headaches of the burgeoning mobile world only make the extra weight more of an issue.

[zepto]: http://zeptojs.com/

### Conclusions

#### Best. DOM Ecosystem. Ever.

The "evergreen browser" is ascendant and their market share is wonderfully fractured, with IE 6/7/8 plunging to joyful depths. I can't recall the vendors ever being so motivated to concern themselves with compatibility. And where they do fail in that, those failings are often patchable with [polyfills and shims][polyfills] or even some [transpiler][transpiler] magic. This is the age of plentiful answers on [stackoverflow][stackoverflow]. This is the age of simple patching and forking on [GitHub][GitHub]. Our client side [developer tools][developer], [editors][editors] and [command line][command] have never been so capable. [Package][Package] [managers][managers] are [crowded][crowded] with assets ready to be installed. It is time to stop being afraid of handling the DOM.

[transpiler]: https://npmjs.org/browse/keyword/transpiler
[stackoverflow]: http://stackoverflow.com/
[GitHub]: http://github.com/
[polyfills]: https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills
[developer]: https://developers.google.com/chrome-developer-tools/
[editors]: http://sublimetext.com/
[command]: http://gruntjs.com/
[Package]: http://npmjs.org/
[managers]: http://bower.io/
[crowded]: http://wheelcode.blogspot.com/2013/05/a-pox-on-both-your-package-managers.html

#### Always Room For Improvement

The DOM is not always pleasant to use. It is not always concise or convenient. There remains a healthy need for libraries to improve upon it. But as IE8 fades into blessed oblivion, it is time to acknowledge that much of what was wrong with extending the DOM in the past has faded also. When you get an urge to smooth those rough spots, add features, and maybe even offer some more concise interface for the DOM, don't be afraid to consider doing it directly, without a wrapper. Because a lot has changed in the last four years, and it only looks to be getting [better][better].

[better]: http://nbubna.github.io/HTML

#### Two Valid, Incompatible Approaches

The prototypes are exposed and you can extend them. This should outperform wrappers and manual extensions, but does raise the risk of conflicts somewhat. Don't do this until you know the DOM (it's present variations and future direction) well enough to choose names that work well to avoid conflicts.

If you are more conflict-shy than performance-focused, you can step lightly and create libraries that only extend when and where they are used and no more, preferable with a small number of additional properties too. Such libraries would not be worth the penalty for DOM-intensive applications but may be able to achieve [features][features] prototype-based DOM extenders never could.

[features]: http://nbubna.github.io/HTML#dot-traversal

I see great value in both techniques, but for different situations. It would probably not even be difficult for some libraries to create similar, parallel versions for the different use cases. Mixing the two in a single implementation does not (for the moment) appear to be worthwhile.

#### The Deciding Factor

Can you afford to deny (full) support to IE8 users? Windows XP stops getting security updates in a matter of months, and it's approaching demise is helping to purge IE8 from the ecosystem. But that [may take a while][may], and meanwhile, IE8 still carries [just under 10%][just] of the market. Whether that number is high enough to avoid DOM extenders is not a question with a single answer. Google has cut off IE8 support already and their [next platform][next] won't even support IE9, while some businesses still demand IE6 support for web applications. Do you want to press the web forward or pander to stubborn legacy users? Every choice is a trade off.

[may]: http://slashdot.org/story/13/08/08/027211/china-has-a-massive-windows-xp-problem
[just]: http://www.sitepoint.com/browser-trends-may-2013/
[next]: http://www.polymer-project.org/compatibility.html

For myself, i am looking toward the future. I won't rule out supporting some number of features on older browsers, but i will not use them as a starting point or baseline. It is far more important that my applications work well in the exploding mobile device market than the shrinking IE8 market. Limited resources dictate prioritizing growth over legacy. The extra bytes that come with the one-version-fits-all approach are not appreciated by mobile users, especially international ones. So, those with antiquated browsers will get the browsehappy.com treatment until sufficient resources are available to deliver a functional version specifically for them. I may not always design UX [mobile first][mobile], but it now seems obvious to approach platform decisions from that perspective.

[mobile]: http://www.abookapart.com/products/mobile-first

[Leave your comments here][comments]

[comment]: https://github.com/nbubna/mind-hacking/issues/1

