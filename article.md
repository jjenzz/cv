# Handy Advanced Sass

I was really quite excited when I first came across [Sass][1]. At the time, it was purely [Sass variables][2] that pinned my interest. They've improved my development process tremendously but there was so much more to learn and having dug deeper into Sass, there is simply **no way** I would choose to work without Sass as part of my development stack anymore. In this article I am going to walk through a few of the examples why and hopefully I'll be able to share something with you that you weren't previously aware of.

## Control Directives

This is when things start to become more akin to a programming language than CSS and they even come with a warning:

> **Note that control directives are an advanced feature, and are not recommended in the course of day-to-day styling**. They exist mainly for use in mixins, particularly those that are part of libraries like Compass, and so require substantial flexibility.

But don't let that put you off. Once you have gotten to grips with these, you can speed up your productivity ten-fold.

### @if

The `@if` directive takes a condition to evaluate and returns the nested styles if the condition is truthy (not `false` or `null`). Specifying what to return if the condition is falsey can be done using the `@else` statement.

For example, you could create a mixin that handles text insetting based on how light or dark the text colour is.

	@mixin text-inset($colour, $opacity: 0.7) {
	  
	  @if lightness($colour) < 50% {
	    // if the text colour is a dark colour
	    // we need a white shadow, bottom, right
	    text-shadow: 1px 1px 1px rgba(#fff, $opacity);
	  } @else {
	    // else it must be light so we need
	    // a black shadow, top, left
	    text-shadow: -1px -1px 1px rgba(#000, $opacity);
	  }
	    
	  // set the text colour
	  color: $colour;
	    
	}

The lightness function used above returns a number between 0% and 100% that represents the hue component of a colour. Therefore, we can check if the text is a light or dark colour and apply the text inset accordingly.

<pre class="codepen" data-height="300" data-type="result" data-href="frsqo" data-user="jjenzz" data-safe="true"><code></code><a href="http://codepen.io/jjenzz/pen/frsqo">Check out this Pen!</a></pre>
<script async src="http://codepen.io/assets/embed/ei.js"></script>

### @for

blah

### @each

blah

### @while

blah


## Passing Blocks of CSS to Mixins

There are times, especially when building responsive websites, that I find myself repeating snippets of CSS with the only difference being the content contained within them. Fortunately, with Sass, you can make your life that little bit simpler by passing entire blocks of CSS to a mixin and then specifying where that CSS should be compiled using the [`@content` directive][4].


### A Basic Example

    @mixin apply-to-ie7 {
      // IE7 hack from 
      // http://www.paulirish.com/2009/browser-specific-css-hacks/
      *:first-child+html & {
        @content;
      }
    }
    
#### Usage

    .btn {
      display: inline-block;
      
      @include apply-to-ie7 {
        overflow: visible;
        display: inline;
        zoom: 1;
      }
    }
    
Anything you write inside the curly braces of the `@include` will be generated in place of the `@content` directive in our mixin.
    
#### CSS Output 

    .btn {
      display: inline-block;
    }
    
    *:first-child+html .btn {
      overflow: visible;
      display: inline;
      zoom: 1;
    }

### Advanced Media Query Example

As a more advanced example, I have used the content directive in the past to make my media queries more manageable:

	// breakpoints defined in settings
	$break-medium: 	31em !default;
	$break-large:  	60em !default;
	$break-x-large: 75em !default;			

    @mixin breakpoint($type, $fallback:false, $parent:true) {
        
      @if $type == medium {
        @media (min-width: $break-medium) {
          @content;
        }
      }
      
      @if $type == large {
        @media (min-width: $break-large) {
          @content;
        }
      } 
      
      @if $type == x-large {
        @media (min-width: $break-x-large) {
          @content;
        }
      }
          	
   	  @if $fallback {
   	  	@if $parent {
   	  	  .#{$fallback} & {
   	  	    @content;
   	  	  }
   	  	} @else {
   	  	  .#{$fallback} {
   	  	    @content;
   	  	  }
   	  	}
      }
    }
    
Here our mixin will accept a `$type` (`medium`, `large` or `x-large`), an optional fallback class (for IE) and a `$parent` boolean (`true` or `false`) that tells our fallback whether it's being included inside or outside a CSS declaration.
    
#### Usage

    .features__item {
      width: 100%;
    }
    
    .foo {
      color: red;
    }
    
    .bar {
      color: green;
      
      // use inside a declaration
      @include breakpoint(medium) {
        color: red;
      }
      
      @include breakpoint(large, lt-ie9) {
        color: blue;
      }
    }
    
    // use outside any declarations
    @include breakpoint(medium) {
      .features__item {
        width: 50%;
      }
    
      .foo {
        color: blue;
      }
    }
    
    // remember to tell the fallback that 
    // it's not within a declaration
    @include breakpoint(large, lt-ie9, false) {
      .features__item {
        width: 25%;
      }
    
      .foo {
        color: green;
      }
    }  
    
#### CSS Output

	.features__item {
	  width: 100%;
	}
	
	.foo {
	  color: red;
	}
	
	.bar {
	  color: green;
	}
	
	@media (min-width: 31em) {
	  .bar {
	    color: red;
	  }
	}
	
	@media (min-width: 60em) {
	  .bar {
	    color: blue;
	  }
	}
	
	.lt-ie9 .bar {
	  color: blue;
	}
	
	@media (min-width: 31em) {
	  .features__item {
	    width: 50%;
	  }
	
	  .foo {
	    color: blue;
	  }
	}
	
	@media (min-width: 60em) {
	  .features__item {
	    width: 25%;
	  }
	
	  .foo {
	    color: green;
	  }
	}
	
	.lt-ie9 .features__item {
	  width: 25%;
	}
	
	.lt-ie9 .foo {
	  color: green;
	}


## The Extend Directive

You may have noticed in the past that your stylesheets can become quite large when using Sass's `@include` directive. This is because every time you `@include`, you are **duplicating** the styles instead of **grouping** your declarations. Fortunately, Sass provides the option to `@extend` a class which will group your declarations and avoid unnecessary bloat. 

The extend directive is one of my favourite features of Sass!

### A Basic Example
   
    .inline-block {
	  display: inline-block; 
	  vertical-align: baseline; 
	  *display: inline; 
	  *vertical-align: auto;
  	  *zoom: 1;
    }
    
    .btn {
      @extend .inline-block;
      padding: 0.5em 1em;
    }
    
    .foo {
      @extend .inline-block;
      color: red;
    }
    
#### CSS Output

    // selectors are grouped
    .inline-block, .btn, .foo {
	  display: inline-block; 
	  vertical-align: baseline; 
	  *display: inline; 
	  *vertical-align: auto;
  	  *zoom: 1;
    }
    
    .btn {
      padding: 0.5em 1em;
    }
    
    .foo {
      color: red;
    }
    
Instead of adding the styles from `.inline-block` into both of our `.btn` and `.foo` declarations, the extend directive grouped our classes alongside the `.inline-block` class, reducing duplication significantly.

### Placeholders

In the example above our `.inline-block` class was also generated within the group of declarations and had we not even extended the `.inline-block` class, it still would have been generated in our final CSS file as follows:

    // no grouping because it wasn't
    // extended, but still exists
    .inline-block {
	  display: inline-block; 
	  vertical-align: baseline; 
	  *display: inline; 
	  *vertical-align: auto;
  	  *zoom: 1;
    }
    
    .btn {
      padding: 0.5em 1em;
    }
    
    .foo {
      color: red;
    }
    
This is not ideal and is exactly where [Placeholder Selectors][5] come in handy. Any CSS declared within a placeholder **will not be generated** in your final CSS file **unless it has been extended**. It also means that other developers cannot inadvertantly hook onto these styles in their markup. I therefore prefer to extend placeholders instead of classes at all times and reserve classes for their intended role as markup hooks.

Placeholders are declared and extended exactly the same way as classes except the dot is replaced with a percentage symbol:

    %inline-block {
	  display: inline-block; 
	  vertical-align: baseline; 
	  *display: inline; 
	  *vertical-align: auto;
  	  *zoom: 1;
    }
    
    .btn {
      @extend %inline-block;
      padding: 0.5em 1em;
    }
    
    .foo {
      @extend %inline-block;
      color: red;
    }

#### CSS Output

    .btn, .foo {
	  display: inline-block; 
	  vertical-align: baseline; 
	  *display: inline; 
	  *vertical-align: auto;
  	  *zoom: 1;
    }
  
    .btn {
      padding: 0.5em 1em;
    }
    
    .foo {
      color: red;
    }
    
Now, the selectors are grouped as before but without an `.inline-block` class amongst the group. Much better!

#### When to Placeholder

A general rule of thumb that I tend to stick to is that if there is a block of CSS you would have bundled into a mixin but the mixin does not require any paramaters (such as a clearfix mixin) then it would probably be better of as a placeholder. 

Having said that, due to the media directive's inability to extend an external placeholder, I sometimes keep the mixin for use within media queries and then include it in my placeholder for use everywhere else:

    // keep the mixin for use 
    // within media queries
    @mixin clearfix {
      *zoom: 1;
      
      &:before,
      &:after {
        content: "";
        display: table;
      }
      
      &:after {
        clear: both;
      }
    } 
    
    // use the placeholder
    // for everything else
    %clearfix {
      @include clearfix;
    }

#### Nesting Placeholders

Placeholders are much like any other selector so they can be nested:

	%grid {
	  overflow: hidden;
	  
	  %column {
	    min-height: 1px;
	  }
	  
	  @for $i from 1 through 12 {
	    %columns-#{$i} {
	      width: ((100 / 12) * $i) * 1%;
	    }
	  }
	}

	.foo, 
    .bar {
	  @extend %grid; 
	}

	.baz {
	  @extend %column;
	  @extend %columns-5;
	}
	
	.qux {
	  @extend %column;
	  @extend %columns-3;
	}
	
… which would generate the following:

	.foo,
	.bar {
	  overflow: hidden;
	}
	
	.foo .baz,
	.bar .baz, 
	.foo .qux,
	.bar .qux {
	  min-height: 1px;
	}
	
	.foo .qux,
	.bar .qux {
	  width: 25%;
	}
	
	.foo .baz,
	.bar .baz {
	  width: 41.66667%;
	}

If we focus on the second block of CSS within the output above, this happens because anything that extends `%column` will be nested within anything that extends `%grid` unless we also nest our `@extend`. So, if you wanted to stop `.baz` from nesting within `.bar`, you would do this:

	.foo {
	  @extend %grid; 
	  
	  .baz {
	    @extend %column;
	    @extend %columns-5;
	  }
	}
	
	.bar {
	  @extend %grid;
	}
	
	.qux {
	  @extend %column;
	  @extend %columns-3;
	}
	
… which generates: 

	.foo, 
	.bar {
	  overflow: hidden;
	}
	
	.foo .baz, 
	.foo .qux, 
	.bar .qux {
	  min-height: 1px;
	}
	
	.foo .qux, 
	.bar .qux {
	  width: 25%;
	}
	
	.foo .baz {
	  width: 41.66667%;
	}
	
Furthermore, due to the placeholder nesting, if you were to extend `%column` but not extend `%grid`, there would be no CSS generated because `%column` can only exist within `%grid`. 

### Advanced Example

Obviously the examples above are completely useless but it demonstrates the possibilities in a simple way. A more useful/complex example of extending (with placeholders) might be if you wanted to create a grid with 2 levels of nesting:

	%grid {
	  margin-left: -2em;
	  overflow: hidden;
	  
	  %column {
        float: left;
	  	padding-left: 2em;
	    min-height: 1px;
	    
	    -webkit-box-sizing: border-box;
	    -moz-box-sizing: border-box;
	    box-sizing: border-box;
	  }
	  
	  @for $i from 1 through 12 {
	    // top level column spanning
	    %columns-#{$i} {
	      width: ((100 / 12) * $i) * 1%;
	      
	      @for $j from 1 through 12 {
	        // second level (nested) column spanning
	        %columns-#{$j} {
              @if $j != $i {
                $width: ((100 / $i) * $j) * 1%;
          
                @if $width <= 100% {
	              width: $width;
                }
              }
	        }
	      }
	    }
	  }
	}

	.my-grid {
	  @extend %grid;
	  
	  [class^="cols-"], 
	  [class*=" cols-"] {
	    @extend %column;
	  }
	  
	  @for $i from 1 through 12 {
	    .cols-#{$i} {
	      @extend %columns-#{$i};
	    }
	  }
	}
	
The output is too big to paste here so I've put it on [pastie][6] but you can see how I only needed to extend `%columns-#{$i}` **once**, yet it generated the necessary column styles for both top level and nested columns.

In theory, this would all make it very easy to create a framework that consisted entirely of placeholders. Then for each project that uses the framework you could extend the modules you need, meaning your resulting CSS would only ever contain the styles you actually use, with the class names of your choice. Yum.

### Extending Into Media Queries

I mentioned earlier that it wasn't possible to extend placeholders defined outside of a media query from within a media query but it is perfectly possible to extend **into** a media query. This might be easier to understand in code. The following is an example of extending from within a media query which **will not** work:

    %warning {
      background-color: red;
    }    
    
    @media (min-width: 31em) {
      .error {
        @extend %warning;
      }
    }

The extend directive will try to find the `%warning` placeholder inside the media query, and since it does not exist in that scope, Sass will error[^1]. However, you can do this:

    %column {
      width: 100%;
    }
    
    @media (min-width: 31em) {
      %column {
        width: 50%
      }
    }
    
    @media (min-width: 75em) {
      %column {
        width: 25%;
      }
    }
    
    .feature__item {
      @extend %column;
    }
    
Notice how I've defined the same placeholder within the two media queries and extended them from within a declaration outside of the media queries. This will result in:

    .feature__item {
      width: 100%;
    }
    
    @media (min-width: 31em) {
      .feature__item {
        width: 50%
      }
    }
    
    @media (min-width: 75em) {
      .feature__item {
        width: 25%;
      }
    }

## API Functions

There are many [functions][7] provided with Sass that can aid development and you have probably already used a few, such as [`lighten()`][8], [`darken()`][9], [`rgba()`][10], but I recently came across some that I wasn't already aware of that have been a great help when I started building a framework.

### if()

Not to be confused with the `@if` directive, this function can be used similar to the ternary operator provided in most languages. In Javascript you might do this:

    var foo = bar ? 'red' : 'green';    

… and in Sass you can do this:

	$foo: if($bar, 'red', 'green');
	
So instead of doing:

	.foo {
	  @if $bar {
	    width: 100%;
	  } @else {
	    width: 50%;
	  }
	}
   
You can do:

    .foo {
      width: if($bar, 100%, 50%);
    } 
  
Much tidier :)

### length()

In Sass, variables can become lists simply by adding more than one value to a variable and separating them with a space or a comma. In fact, Sass variables are just lists with one value but what if you need to know how many items are in a list? You can do just that by calling `length()` on the list.

    $colours: red green blue;
    $colourCount: length($colours); // 3
    
This is really handy when working with loops and we will see an example of this in the next section.

### nth()

The `nth()` function enables you to grab a specific value from a list by it's position in the list (from 1 to list length). It takes two parameters - the list that you wish to search and the position of the item you want to retrieve.

	$colours: red green blue;
	
	.rgb {
      float: left;
	  width: 100px;
	  height: 100px;
	}
	
	@for $i from 1 through length($colours) {
	  .rgb:nth-of-type(#{$i}) {
	    background-color: nth($colours, $i);
	  }
	}
	
<pre class="codepen" data-height="150" data-type="result" data-href="vAzaq" data-user="jjenzz" data-safe="true"><code></code><a href="http://codepen.io/jjenzz/pen/vAzaq">Check out this Pen!</a></pre>
<script async src="http://codepen.io/assets/embed/ei.js"></script>

We've looped from 1 to 3 and set the background colour on each element to whichever colour is at position 1 to 3 in the $colours variable. Doing it this way keeps things a lot [DRY][11]er and easier to maintain.

### index()

The `index()` function returns the position (from 1 to list length) of a value within a list and will return `false` if it cannot find the value. It also takes two parameters - the list to search and the value whose position you want to return. So, we could rewrite our above example to the following:

    $colours: red green blue;
  
	.rgb {
      float: left;
	  width: 100px;
	  height: 100px;
	}
	
	 @each $colour in $colours {
	   .rgb:nth-of-type(#{index($colours, $colour)}) {
	     background-color: $colour;
	   }
	 }

It's up to you which you prefer but `index()` and `nth()` can be really handy when working with multiple lists. Stay tuned as I will be demonstrating this next!

### Using `nth()` and `index()` to Refactor the Media Query Example

There are such things as multi-dimensional lists in Sass which can either be separated with braces or commas, such as:
	
	// both of these are the same
    $breakpoints: (medium 31em) (large 75em) (x-large 75em) !default;
    // or: 
    $breakpoints: medium 31em, large 75em, x-large 75em !default;
    
With the use of a multi-dimensional list, we could improve our media query mixin so that you never have to touch the mixin when adding a new breakpoint.

	// breakpoints defined in settings
	$breakpoints: medium 31em, large 75em, x-large 75em !default;

	// mixin defined elsewhere and shouldn't be touched
    @mixin breakpoint($type, $fallback:false, $parent:true) {
    
      @each $breakpoint in $breakpoints {
      	$alias: nth($breakpoint, 1);
      	$bp: nth($breakpoint, 2);
      	
      	@if $alias == $type {
	      @media (min-width: $bp) {
	        @content;
	      }
	      
	   	  @if $fallback {
	   	  	@if $parent {
	   	  	  .#{$fallback} & {
	   	  	    @content;
	   	  	  }
	   	  	} @else {
	   	  	  .#{$fallback} {
	   	  	    @content;
	   	  	  }
	   	  	}
	      }
      	} 
      }
      
    }

This is a huge step forward since having to update the mixin every time would make it prone to human error which could easily break it. However, we could refactor further if we separate the settings variables into `$breakpoints` and `$breakpoint-aliases`. 


	// breakpoints defined in settings
	$breakpoints: 31em 60em 75em !default;
	$breakpoint-aliases: medium large x-large !default;
	
	// mixin defined elsewhere and shouldn't be touched
    @mixin breakpoint($type, $fallback:false, $parent:true) {
      $pos: index($breakpoint-aliases, $type);
      $bp: nth($breakpoints, $pos);
      
      @media (min-width: $bp) {
        @content;
      }
      	
   	  @if $fallback {
   	  	@if $parent {
   	  	  .#{$fallback} & {
   	  	    @content;
   	  	  }
   	  	} @else {
   	  	  .#{$fallback} {
   	  	    @content;
   	  	  }
   	  	}
      }
    }
    
Here we use `index()` to find the position of the `$type` within the aliases variable and then use that position to get the breakpoint for that alias. We've managed to remove a loop and an if statement making it a much more legible mixin.

## Conclusion

I have only just touched on some of the more advanced features of Sass but hopefully you can see how powerful they can be. 

I would highly recommend having a read through the [Sass Functions documentation][7] to see if there is anything there that might aid your productivity further. Already using some? Please do share any examples you use that have not been mentioned here as we are all still learning, me included.

Happy Sassing :)

[1]: http://www.sass-lang.com
[2]: http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#variables_
[3]: http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#control_directives
[4]: http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixin-content
[5]: http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#placeholder_selectors_
[6]: http://pastie.org/private/bphyg3q6xqatizmcptmdw
[7]: http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html
[8]: http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#lighten-instance_method
[9]: http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#darken-instance_method
[10]: http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#rgba-instance_method
[11]: http://en.wikipedia.org/wiki/Don't_repeat_yourself

[^1]: Currently Sass will display a warning but there are plans to make this error in future versions.
