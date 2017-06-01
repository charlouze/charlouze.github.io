---
layout: post
title:  "Ionic 3 and Font Awesome"
author: Charlouze
date:   2017-05-31 19:50:15 +0200
category: Ionic
tags: Ionic Ionic3 Font Awesome FontAwesome
comments: true
---

## Requirements

A Ionic 3 project. See [here](https://ionicframework.com/docs/intro/installation/) for more informations.

## Install Font Awesome

It's quite easy: `npm install font-awesome --save  --save-exact`

## Configure Ionic to include Font Awesome

Making Font Awesome available in our app is not that hard... we just need to:
1. configure the build to copy Font Awesome fonts
2. configure the build to include Font Awesome sass path
3. make Font Awesome styles available to our project

### Configure the build

Ionic building system can be easily configured.If you need to know more about it, you can find informations
[here](https://github.com/ionic-team/ionic-app-scripts).

#### 1. Configure copy task

The Ionic copy task, as every other tasks, is configured using a JSON object. Each property of this JSON object is a
copy sub-task. For each sub-task, there is a source `src`, that is an array of directories and files, and a destination
`dest`, that is a path to where you want to copy everything.

Some placeholder can be use as `{% raw %}{{ROOT}}{% endraw %}` for root directory and `{% raw %}{{WWW}}{% endraw %}` for
target directory.

Here is my marvellous `config/copy.config.js` file:

{% highlight javascript linenos %}
// New copy task for font files
module.exports = {
  copyFontAwesome: {
    src: ['{{ROOT}}/node_modules/font-awesome/fonts/**/*'],
    dest: '{{WWW}}/assets/fonts'
  }
};
{% endhighlight %}

Adding a property with a different name than the default `copyFonts` allows to only take care about fa fonts.
Ionic building system automatically adds default configuration.

#### 2. Configure sass task

Sass include paths are configure using the `includePaths` property of the sass configuration. 

Add a `config/sass.config.js` with:

{% highlight sass linenos %}
// Adding Font Awesome to includePaths
module.exports = {
  includePaths: [
    'node_modules/ionic-angular/themes',
    'node_modules/ionicons/dist/scss',
    'node_modules/ionic-angular/fonts',
    'node_modules/font-awesome/scss'
  ]
};
{% endhighlight %}

As you can see, I'm overriding the `includePaths` property.
I need to copy [default config](https://github.com/ionic-team/ionic-app-scripts/blob/master/config/sass.config.js#L43-L47)
if I want the sass task to work properly.

#### 3. Enabling the custom configuration

There are several ways to [enable custom configuration](https://github.com/ionic-team/ionic-app-scripts#overriding-config-files),
I choose to add it to `package.json` `config` property.

{% highlight javascript lineos %}
  "config": {
    "ionic_copy": "./config/copy.config.js",
    "ionic_sass": "./config/sass.config.js"
  }
{% endhighlight %}

### Make Font Awesome available

To use Font Awesome, we need to import it. It's now simple as two lines of code !

Add the code below at the end of your `src/theme/variables.scss` file.

{% highlight sass lineos %}
// Font Awesome
$fa-font-path: $font-path;
@import "font-awesome";
{% endhighlight %}

By default, `$fa-font-path` equals to `../fonts`. We configured fonts file to be copied to `../assets/fonts` which is
the ionic default font path.

## Use Font Awesome

You are now ready to use font awesome. Refer to its [documentation](http://fontawesome.io/examples/)!

## Why not using a ion-icon like component ?

Sure ! My fa-icon component that tries to mimic some aspect of the ion-icon has some nice features:

* Select an icon using the `name` attribute without `fa-` prefix (e.g. `camera-retro`).
* Define the icon color using the `color` attribute (e.g `primary`).
* Define the icon size using the `size` attribute without `fa-` prefix (e.g. `lg`, `2x` or ,`5x`).
* Fix icon width using the `fixed-width` attribute.
* Handle button icon using `icon-left`, `icon-right`, `icon-only` button attributes.

And you can add your own needed features.

### Usage

{% highlight html %}
<!-- basic usage -->
<fa-icon name="camera-retro"></fa-icon>
<!-- basic usage with color -->
<fa-icon name="camera-retro" color="danger"></fa-icon>
<!-- larger icons -->
<fa-icon name="camera-retro" size="4x"></fa-icon>
<!-- fixed width icons -->
<fa-icon name="camera-retro" fixed-width></fa-icon>
<!-- dynamic value -->
<fa-icon [name]="icon"></fa-icon>
<!-- buttons -->
<button ion-button icon-left>
  <fa-icon name="group"></fa-icon>
  people
</button>
{% endhighlight %}

### Source code

`src/components/fa-icon/fa-icon.component.ts`:

{% highlight typescript linenos %}
import {Component, ElementRef, Input, OnChanges, Renderer, SimpleChange, SimpleChanges} from "@angular/core";
import {Config, Ion} from "ionic-angular";

@Component({
  selector: "fa-icon",
  template: "",
})
export class FaIconComponent extends Ion implements OnChanges {
  @Input() name: string;
  @Input() size: string;

  @Input("fixed-width")
  set fixedWidth(fixedWidth: string) {
    this.setElementClass("fa-fw", this.isTrueProperty(fixedWidth));
  }

  constructor(config: Config, elementRef: ElementRef, renderer: Renderer) {
    super(config, elementRef, renderer, "fa");
  }

  ngOnChanges(changes: SimpleChanges): void {
    if (changes.name) {
      this.unsetPrevAndSetCurrentClass(changes.name);
    }
    if (changes.size) {
      this.unsetPrevAndSetCurrentClass(changes.size);
    }
  }

  isTrueProperty(val: any): boolean {
    if (typeof val === "string") {
      val = val.toLowerCase().trim();
      return (val === "true" || val === "on" || val === "");
    }
    return !!val;
  };

  unsetPrevAndSetCurrentClass(change: SimpleChange) {
    this.setElementClass("fa-" + change.previousValue, false);
    this.setElementClass("fa-" + change.currentValue, true);
  }
}
{% endhighlight %}

`src/components/fa-icon/fa-icon.component.scss`:

{% highlight sass linenos %}
// Color icons

@mixin fa-color($color-name, $color) {
  .fa-ios-#{$color-name} {
    color: $color;
  }
  .fa-md-#{$color-name} {
    color: $color;
  }
  .fa-wp-#{$color-name} {
    color: $color;
  }
}

@each $color-name, $color-base, $color-contrast in get-colors($colors) {
  @include fa-color($color-name, $color-base);
}

// Icons and buttons

@mixin button-icon {
  font-size: 1.3em;
  line-height: .67;

  pointer-events: none;
}

[icon-left] fa-icon {
  @include button-icon;

  @include padding-horizontal(null, .3em);
}

[icon-right] fa-icon {
  @include button-icon;

  @include padding-horizontal(.4em, null);
}

[icon-only] fa-icon {
  @include padding(0, .5em);

  font-size: 1.8em;
  line-height: .67;

  pointer-events: none;
}

{% endhighlight %}

### The end

Now you know how to include Font Awesome to your Ionic application.

Here is my [sample project](https://github.com/charlouze/ionic-font-awesome).
