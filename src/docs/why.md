The biggest reason is developer happiness from not having to manage too many files that are intrinsically linked to each other and not having to use different tools to get the *single* job done 🌹.

But of course, there are lots of other reasons as well and we present a few below.

# Comparison to other options

There are a lot of other CSS in JS frameworks out there. In the past we used and experimented with quite a few. Some quick reasons why we wrote our own.

* We are focused on Autocomplete / *Compile* time error analysis
  * None of them had this out of the box.
  * Not all APIs are statically analyzable e.g. if the API is powered by template strings, the CSS portion is essentially not analyzed at all.
* Some forced you to use a custom AST transforms 
  * Would be fine if custom ASTs came with IDE support + static analysis. It doesn't.
* Many others are framework specific e.g. React specific 
  * Some force you to rethink / wrap your component. Didn't want that.
  * They make upgrading your frontend framework harder as you need for them to update their wrapper first. 
* Many others try to solve problem with JS events instead of generating CSS.
  * This can result in issues when a JS event is absent, e.g. [a stuck `:hover`](https://goo.gl/e5tUyt)
  * Can be significantly slower in real world usage as CSS does a faster job of changing quick styles than events.
  * The number of issues reported on libraries that use JS events is generally too high for comfort.
  * Generally framework specific and that has problems we've mentioned before.
  * They also tend to change `style` instead of writing CSS which makes using devtools with immediate feedback (e.g. chrome dev tools) harder.
* CSS Modules : Not CSS in JS. Just solves namespacing.
  * Most the other CSS managment problems still exist. We are essentially CSS modules, if CSS modules were written in JS.
* Super small core size (~6k gz). 
  * We are just putting a type system + encapsulation / maintainability (no globals) on CSS.

Of course we would not exist without their previous and continued hard work and public idea exchange 🌹 

## Concept: Hashed className

CSS class names (e.g. `.btn`) suffer from global conflicts. 

* This makes integrating with third party libraries hard. Each library must follow a convention to prevent mistakes e.g. `bootstrap-btn` but *most libraries* don't and camp on global names.
* This makes even writing maintainable CSS *in your project* hard and you need to treat it as *global* e.g. if you want to create a class for buttons in two components e.g. date picker and a file dropzone you must have `.mine-datepicker-button` and `.mine-dropzone-button` to prevent mistaken conflicts. With JS/TS you would probably have seperate `.js` files (e.g. `datePicker.js` and `dropzone.js`) and each can have a local variable `buttonClass` without any conflicts. You don't even need to export these to the outside world.

However due to limitations of CSS we still need to generate a CSS file to allow you to use features like hover states and media queries. Hence we generate a className based on the content for you: 

```play
import {style} from 'typestyle';

const myColorClass = style({color:'red'});

<div>The generated class name: {myColorClass}</div>;
```

This gives you: 

* Isolation: No globals!

## Concept: Dead CSS 

Determining the impact of a CSS className on your codebase is one of the hardest problems facing frontend maintainability. 

Having the styles managed in JS (especially with TypeScript) gives you the following immediate gains:

* You can immediately see all the places where a class is used (e.g find references in TypeScript). This also gives you the *impact* footprint of a CSS class.
* Refactor class names easily, especially true with TypeScript (e.g. from `fooClass` to `barClass`. You no longer need to be afraid to touch your CSS class names).
* Remove CSS classes that are no longer used easily (e.g. switch on `noUnusedLocals` in TypeScript).
* Delete a TS file containing CSS classNames. If it's used, you get a nice compiler error which you can fix easily (same way you fix / remove unused JS code). Next go out and party 🎉.
* Based on how *all module loaders work* (including webpack/tsify/rollup) if a file isn't *required*, it doesn't become a part of the bundle. So their CSS also goes away *automatically*. 
* With fancy tree shaking module loaders (like rollup/webpack2) if a variable isn't used, it's removed from the bundle. So even without `noUnusedLocals`, the CSS bound to these variables (e.g. `const fooUnused = style({color:'red'})`) goes away.

## Vendor Prefixing

Note: We don't do *automatic* vendor prefixing for a few reasons:

* Code bloat, runtime performance, you might want more control (we don't make choices that you might need to undo).
* Vendor prefixing has no future: https://webkit.org/blog/6131/updating-our-prefixing-policy/

> Protip: Big fan of flexbox? Use `csstips` as it provides the necessary vendor prefixes so you don't need to worry about them.

## More boring reasons 

Beyond that here is a boring list of additional reasons to use TypeStyle.

* No context switching your brain (think it's worth mentioning again).
* Built in dependency system same as the rest of your JS. No special code for CSS needed.
* Ship CSS in the same channel that you ship JS, with no special configuration being required by your library user.
* Minification (Minify JS with existing tools). The CSS we generate is already nearly whitespace free.
* Shared constants and reusable styles (Using variables and objects)
* Extensible (Just use JavaScript with all its power)
* Your components are still free to have class names that you can give to external people to further style your stuff (better still take `clasName` as a property and let them use *typestyle* too!).
* Develop components alongside the style (No more hunting CSS files for estranged `ul > li > a`)
* Create isomorphic applications (easy export to a CSS file is supported)
* All the power of CSS without compromise e.g. pseudo states (e.g. `&:hover`)
* Better than CSS, management of media queries i.e. *nested class driven* media quries
* Fallback CSS properties using arrays (`{ backgroundColor: ['red', 'linear-gradient(to right, red 0%, blue 100%)'] }`)
* Super small core size (~1k).
* Extremely small and powerful API that works with any ecosystem.
* Provides great TypeScript developer autocomplete experience.
* No custom AST transform or module loader support needed.
* Works with any framework (react, angular2, cyclejs, whatever, doesn't matter).
* Zero config. Just use.

> Note: Many of these are truly the advantages of using FreeStyle. The additional features by typestyle are *autoinjection*, *`types`* (for autocomplete and errors), `csx` (CSSFunctions and Typed CSS value creators) and *csstips* (a great set of CSS mixins / page setup helpers to give a smooth learning curve for even new CSS devs). 
