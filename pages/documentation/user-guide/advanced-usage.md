---
layout: page-fullwidth
show_meta: false
title: "Gauges Advanced Usage"
subheadline: "Canvas Gauges User Guide"
teaser: "Tips and tricks for advanced usage of canvas gauges"
header: false
permalink: "/documentation/user-guide/advanced-usage"
breadcrumb: true
---
<div class="row">
<div class="medium-4 medium-push-8 columns" markdown="1">
<div class="panel radius toc" markdown="1">
<h4>Table of Contents</h4>
{: #toc }
*  TOC
{:toc}

<h4>User Guide Chapters</h4>
<ul>
    <li><a href="{{site.url}}/documentation/user-guide/">Introduction</a></li>
    <li><a href="{{site.url}}/documentation/user-guide/configuration">Configuration</a></li>
    <li><a href="{{site.url}}/documentation/user-guide/custom-builds">Creating Custom Builds</a></li>
    <li><a href="{{site.url}}/documentation/user-guide/using-as-component">Using As Components</a></li>
    <li><a href="{{site.url}}/documentation/user-guide/scripting-api">Scripting API</a></li>
    <li><a href="{{site.url}}/migration/">Migration Guide</a></li>
</ul>
</div>
</div><!-- /.medium-4.columns -->

<div class="medium-8 medium-pull-4 columns" markdown="1">

## Async Loading

Canvas gauges support async loading of their code. If you are using gauges as HTML-components on your page async load of the script is usually safe. So it is enough just to load gauges gode as following:

~~~html
<script async src="gauge.min.js"></script>
~~~
From other hand if you are using scripting on a page for gauges initialization async loading may break your code execution. In this case it is better to wrap gauges initialization with the function which then should be used as handler for gauge script loading, for example:

~~~html
<!doctype html>
<html>
<head>
  <title>Canvas Gauges Async Scripting</title>
</head>
<body>

<!-- ... some code before ... -->

<canvas id="scripted-gauge"></canvas>

<script>
function initGauge() {
    var gauge = new RadialGauge({
        renderTo: 'scripted-gauge',
        width: 300,
        height: 300
    }).draw();
}
</script>

<script async src="gauge.min.js" onload="initGauge()"></script>

<!-- ... some code after ... -->

</body>
</html>
~~~

## Advanced Animations

### Improving Performance On Old Browsers

Currently canvas gauge supports animations using ```requuestAnimationFrame``` calls. For older browsers without support it fallback to use timers, which are not that efficient, so on old platforms you may see performance degrade.

One of a tricks here could be disabling animations if browser does not support requestAnimationFrame if you face a performance problem. This could be done like:

~~~javascript
var gauge = new RadialGauge({
    // ... config options .. 
    animation: !!window.requestAnimationFrame
});
~~~

Of course, feature detection may be done more clever way, using vendor prefixed feature detection, as far as gauges detects them also.

### Using custom Animation Rules

Currently canvas gauges provides a various pre-defined animation rules, like:

 - linear
 - quad
 - quint
 - cycle
 - bounce
 - elastic

and their opposites:

 - dequad
 - dequint
 - decycle
 - debounce
 - delastic

Therefore, if it is not enough it is provide a way to create your own rules, which will be used during animations.

Defining a rule must follow the interface:

~~~javascript
public AnimationRule: function(percent: number): number
~~~

So it is simply a function which takes a percent of animation completion as an argument and transforms it by some mathematical rule.

For example, implementation of linear rule looks like this:

~~~javascript
var linearRule = function(percent) {
    return percent;
};

var gauge = new RadialGauge({
    // ... some options ...
    animationRule: linearRule
});
~~~

Or a bit complicated elastic rule:

~~~javascript
var gauge = new LinearGauge({
    // ... some options ...
    animationRule: function(percent) {
        return 1 - (function(p) {
            var x = 1.5;
            return Math.pow(2, 10 * (p - 1)) * 
                   Math.cos(20 * Math.PI * x / 3 * p);
        })(1 - percent);
    }
});
~~~

So there is no limits except the fantasy of developer to make any type of animation.

## Integration With Custom Fonts

Canvas gauge provides a basic interface to customize fonts of the text element used during the gauge rendering process.

It is done with generic configuration options:

 - fontValue: string font-family
 - fontNumbers: string font-family
 - fontUnits: string font-family

By the way there could be some issues to solve if you are going to use custom loaded fonts on a web page.

As far as gauges are rendered as-fast-as-possible it means that the font can be loaded on the page **after** the gauge has been rendered. And it requires to re-draw the gauge after the font loading. As far as canvas gauge library follow the strategy of providing a minimalist code it won't try to detect font loading, as far as majority of the users may even not need this feature. Those who require this feature have to take care about font loading themselves.

But font loading detection could be not that simple task, especially for old browsers. Modern browsers provide experimental ```document.fonts``` interface which is referring to [CSS Font Loading API](https://developer.mozilla.org/en-US/docs/Web/API/CSS_Font_Loading_API)

If you are targeting to a newest browser only it's not that hard to do. All you need is to wait until font is loaded and redraw the gauge. To hide font-change effect on the gauge it will be enough to make canvas element hidden by default with CSS, like:

~~~html
<link href="https://fonts.googleapis.com/css?family=Orbitron"
      rel="stylesheet">
<script src="gauge.min.js"></script>
<canvas
    data-type="radial-gauge"
    data-font-value="Orbitron"
    data-width="300"
    data-height="300"
    style="visibility:hidden"
></canvas>
<script>
document.fonts.forEach(font => {
    font.loaded.then(() => {
        // using match, because in FF it contains quote marks
        if (font.family.match(/Orbitron/)) {
            let gauge =  document.gauges[0];
            gauge.update();
            gauge.options.renderTo.style.visibility = 'visible';
        }
    });
});
</script>
~~~

If there is a need to support older browsers it may require to write your own solution or to use some 3d-party solution like [WebFontLoader](https://developers.google.com/fonts/docs/webfont_loader) from Google.

## DOM Mutations Support In Old Browsers

In old browsers canvas gauge may not work properly as a web-component. Due to a strategy of minimalist code we were not include any polyfill for [MutationObserver](https://developer.mozilla.org/docs/Web/API/MutationObserver). So if you need to support this feature for some old platforms you have to load some polyfill for MutationObserver **before** loading canvas gauge library code.

For example you can use [this one](https://github.com/webcomponents/webcomponentsjs) or it's [ancestor](https://github.com/Polymer/MutationObservers) (because of minimalism, despite the fact it's deprecated).

## Manual DOM Mutations Control

Sometimes it may be required to disable automatic DOM document parse for some reason (for example, in case of performance optimizations, etc.). This could be easily achieved by defining a global constant `GAUGE_NO_AUTO_INIT` and set it to some truthy value before loading main gauges JavaScript code. For example, such gauge code base loading:

~~~html
<script>var GAUGE_NO_AUTO_INIT = true;</script>
<script src="../gauge.min.js"></script>
~~~

will prevent automatic DOM parsing of the page to initialize gauges. This may improve
page load time. Then, when it is required each gauge could be re-initialized by using the following BaseGauge interface.

~~~
BaseGauge.fromElement(canvasGaugeElement);
~~~

</div><!-- /.medium-8.columns -->
</div><!-- /.row -->
