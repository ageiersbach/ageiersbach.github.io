---
layout: post
title:  "Using Babel with Rails"
tags: rails babel es6 npm
date: 2015-11-1
---

Since [Babel](https://babeljs.io/) made it possible to transpile ES6 to javascript supported by the browser, I've been wanting to incorporate it into all of my Rails projects! I've tried to do this a few times now, and so I thought I would share some of my notes on what it would take to set something like this up.

Currently if you look at the setup for Babel on Rails, it will direct you to install the gem ['sprockets-es6'](https://github.com/TannerRogalsky/sprockets-es6), but when you look at that gem's README, it says:

>"This plugin is primarily experimental and will never reach a stable 1.0. The purpose is to test out BabelJS features on Sprockets 3.x and include it by default in Sprockets 4.x"

I'm not very patient, however, and since Sprockets 3.0 hasn't been released yet (it's still in beta), I've had to find my own workarounds.

Originally I tried to create my own js [compressor](http://guides.rubyonrails.org/asset_pipeline.html#using-your-own-compressor), to first transpile the es6 files before concatenating and compressing them. This proved to be really difficult, however, so I turned to using [`npm`](https://www.npmjs.com/) to handle building my es6 scripts separately.

Another option would have been to replace the asset pipeline altogether; but I wasn't *quite* ready for this step, so I chose a half-way step, and created a new top-level directory in my Rails application which had its own package.json and node_modules directory. The file structure looks something like this:

{% highlight text %}

appjs/
    .babelrc
    lib/
    node_modules/
    package.json
    test/
{% endhighlight %}

Note the `.babelrc`: this is new since Babel 5. It's pretty straightforward; it lets you choose which plugins to apply when you use Babel. I used a preset, so my whole babelrc was pretty short:

{% highlight js %}
{
  "presets": ["es2015"],
}
{% endhighlight %}

It is also possible to include your babel configuration in your package.json, but I chose not to do this to keep the package.json a bit more concise.

{% highlight js %}
{
  ...
  "scripts": {
    "test":"mocha --compilers js:babel-core/register",
    "posttest":"babel lib -d ./../app/assets/javascripts/wookiejs"
  },
  ...
  "dependencies": {
    "babel": "^6.0.14",
    "babel-cli": "latest",
    "babel-preset-es2015":"^6.0.14",
    "watch":"latest",
    "mocha":"latest"
  }
}
{% endhighlight %}

Now, when I run `npm test`, it runs the tests, and *if* the build passes, it transpiles the code and sends it to a new directory under app/assets/javascripts. Unfortunately, if I remove files, I have to remember to remove the corresponding transpiled code under app/assets/javascripts. Also, I don't think that this workflow would work well with continuous integration, since it's not testing the transpiled code to ensure that the application is ready for production. A better workflow might be:

{% highlight text %}
watch appjs/lib ->
  on 'create' or 'change' -> transpile to $dest
  on 'remove' -> remove transpiled file in $dest

watch $dest ->
  run tests
{% endhighlight %}
