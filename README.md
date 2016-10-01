# zprint

__zprint__ is a library providing a pretty printing capability for
both Clojure code and Clojure/EDN structures.  It can be used as a library,
either embedded in a larger codebase, or as a useful utility at the
repl.  See [lein-zprint][leinzprint] to use the zprint library to
reformat your source files.

------------------------------
### For the present, please consider this ALPHA quality software.

It needs a few more people to beat on it before it is ready for
prime time.

------------------------------

Zprint is designed to be a single pretty printer to use for code
and data structures.

#### What, really?  Another pretty printer?

Yes, I know we already have several.  See [here](#another-pretty-printer)
for an explanation.

One of the things I like the most about Clojure (and any Lisp) is that 
the logical structure of a function has a visual representation -- if
the function is pretty printed in a known way.  Zprint exists in part to take
any Clojure code, and pretty print so that you can visually
grasp its underlying structure.

You can see the features available in zprint below, but the major
goals for zprint are:

* Reformat (pretty print) Clojure code, completely ignoring any
  existing white space.  Fit the result as strictly as possible within 
  a specified margin, while using the vertical space most 
  efficiently.   

  For example, here is a before and after:

```clojure
(defn apply-style-x
  "Given an existing-map and a new-map, if the new-map specifies a 
  style, apply it if it exists.  Otherwise do nothing. Return 
  [updated-map new-doc-map error-string]"
  [doc-string doc-map existing-map new-map] (let [style-name (get new-map :style :not-specified)]
  (if (= style-name :not-specified) [existing-map doc-map nil] (let [style-map ( if (= style-name :default) 
  (get-default-options) (get-in existing-map [:style-map style-name]))] (cond (nil? style-name) 
  [exist ing-map doc-map "Can't specify a style of nil!"] style-map [(merge-deep existing-map style-map) 
  (when doc-map (diff-deep-doc (str doc-string " specified :style " style-name)
  doc-map existing-map style-map)) nil] :else [existing-map doc-map (str "Style '" style-name "' not found!")])))))

(zprint-fn apply-style-x 90)

(defn apply-style-x
  "Given an existing-map and a new-map, if the new-map specifies a
  style, apply it if it exists.  Otherwise do nothing. Return
  [updated-map new-doc-map error-string]"
  [doc-string doc-map existing-map new-map]
  (let [style-name (get new-map :style :not-specified)]
    (if (= style-name :not-specified)
      [existing-map doc-map nil]
      (let [style-map (if (= style-name :default)
                        (get-default-options)
                        (get-in existing-map [:style-map style-name]))]
        (cond (nil? style-name) [existing-map doc-map "Can't specify a style of nil!"]
              style-map [(merge-deep existing-map style-map)
                         (when doc-map
                           (diff-deep-doc (str doc-string " specified :style " style-name)
                                          doc-map
                                          existing-map
                                          style-map)) nil]
              :else [existing-map doc-map (str "Style '" style-name "' not found!")])))))
```
 
* Do a great job, "out of the box" on formatting code at the repl, code in
files, and EDN data structures.  Also be highly configurable, so you can
adapt zprint to print things the way you like them without having to dive
into the internals of zprint.

* Handle comments while printing Clojure source.  Some folks eschew
comments, which is fine (unless I have to maintain their code, of course).
For these folks, zprint's comment handling will provide no value.
For everyone else, it is nice that comments don't disappear
when pretty printing source code.

* Do all of this with excellent (and competitive) performance.

Zprint itself doesn't have anything terribly high-tech in it, and isn't
itself either short or particularly beautiful.  But it will bring out the beauty
in __your__ code and data structures better than anything else of which I
am aware.

[leinzprint]: https://github.com/kkinnear/lein-zprint

## Usage

__Leiningen ([via Clojars](http://clojars.org/zprint))__

[![Clojars Project](http://clojars.org/zprint/latest-version.svg)](http://clojars.org/zprint)

## Features

In addition to meeting the goals listed above, zprint has the 
following specific features:

* Prints function definitions at the repl (including clojure.core functions)
* Prints s-expressions (EDN structures) at the repl
* Processes Clojure source files through lein-zprint
* Competitive performance
* Highly configurable, with an intuitive function classification scheme
* Respects the right margin specification
* Handles comments
* Optionally will indent right hand element of a pair (see below)
* Maximize screen utilization 
* Sort and order map keys
* Constant pairing (for keyword argument functions)
* Justify paired output (maps, binding forms, cond clauses, etc.) if desired
* Syntax coloring

All of this is just so many words, of course.  Give zprint a try on
your code or data structures, and see what you think!

### API

The API for zprint is small.  A simple example:

```clojure
(require '[zprint.core :as zp])

(zp/zprint {:a "this is a pretty long value" 
 :b "a shorter value" :c '(a pretty long list of symbols)})

{:a "this is a pretty long value",
 :b "a shorter value",
 :c (a pretty long list of symbols)}
```

The basic API is:

```clojure
;; The basic call uses defaults, prints to stdout
(zprint x)

;; All zprint- functions also allow the following arguments:

(zprint x <width>)
(zprint x <width> <options>)
(zprint x <options>)

;; Format a function to stdout (accepts arguments as above)
(zprint-fn <fn-name>)

;; Output to a string instead of stdout
(zprint-str x)
(zprint-fn-str <fn-name>)

;; Colorize output for an ANSI terminal
;;
;;   None of the syntax coloring in this readme is from zprint, it is 
;;   all due to the github flavored markdown.
;;
(czprint x)
(czprint-fn <fn-name>)
(czprint-str x)
(czprint-fn-str <fn-name>)
```

If `<width>` is an integer, it is assumed to be a the width.  If it
is a map, it is assumed to be an options map.  You can have both,
either, or neither in any zprint or czprint call.

If you need to refresh your memory for the API while at the repl, try:

```clojure
(zprint nil :help)
```

Note that zprint completely ignores all whitespace and line breaks
in the function definition -- the formatting above is entirely
independent of the source of the function.

Zprint has two fundemental regimes -- formatting s-expressions, or parsing
a string and formatting the results of the parsing.  When the `-fn` versions
of the API are used, zprint acquires the source of the function, parses it,
and formats the result.  

### Support

If you have a problem, file an issue with an example of what doesn't
work, and please also include the output from this call:

```clojure
(require '[zprint.core :as zp])

(zp/zprint nil :support)
```

This will assist me a great deal in reproducing and working on the issue.  Thanks!

### Acknowledgements

At the core of `zprint` is the `rewrite-clj` library by Yannick Scherer, 
which will parse Clojure source into a zipper.  It works great!  I would
not have attempted `zprint` if `rewrite-clj` didn't exist to build upon.

### Another pretty printer

Aren't there enough pretty printers already?  What about:

* [clojure.pprint](https://clojure.github.io/clojure/clojure.pprint-api.html) which 
is built into Clojure, and does both s-expressions as well as code.
This features a port of the redoubtable Common Lisp formatter.

* [fipp](https://github.com/brandonbloom/fipp) "Fast Idiomatic Pretty Printer" and 
[puget](https://github.com/greglook/puget), a useful
library that adds significant capability to fipp.
Fipp features a very cool formatting engine, and puget adds some 
great features on top of fipp (in particular color and sorted keys
in maps).

* [cljfmt](https://github.com/weavejester/cljfmt) Which will pretty print your source files for you.
Cljfmt is truly beautiful code, crazy short and very neat inside.

These are all great packages, they have been around for a while,
and are quite useful.  Moreover, they each have some really high-tech
elements as I mentioned above.

I've been using these packages for years, and have even hacked both
clojure.pprint and fipp/puget to print things in a way a bit more
to my liking.  That said, there were a number of things that I
wanted a pretty printing package to do that none of these presently
do or that I could modify them to do in a straightforward
manner.

# Configuration

## Quick Start

The basic API is:
```clojure
(zprint x <width> <options>)
;or
(zprint x <options>)
```
If the third parameter is a number, it is used as the width of the
output.  Default is 80.  Zprint works hard to fit things into the
width, though strings will cause it to fail, as will very small widths.

`<options>` is a map of options.

Zprint prints out code and s-expressions the
way that I think it looks "best", which is very much like how most
people write Clojure code.  It does many specific things not covered
by the "community" coding standards.  In addition, it does some
things slightly differently than the community standards.  If what
you see as the default formatting doesn't please you, you could try
specifying the `:style` as `:community`, for example:

```clojure
; the default way:

(czprint-fn defn)

; the community way:

(czprint-fn defn {:style :community})
```
If this is more pleasing, you might read below as to how you could
configure this as your personal default.  You can, of course, also
configure loads specific parameters to tune the formatting
to your particular taste.  You can also
direct zprint to format any function (including functions you have
defined) in a wide variety of ways using several different paths to
get zprint to understand your desired configuration.

## Introduction

Part of the reason that zprint exists is because I find that the visual
structure of pretty printed Clojure code tells me a lot about the semantics
of the code.  I also have a few areas where my preferred formatting differs
from the the current community standards.  By default zprint will format
Clojure code the way I think it makes the most sense, but it is very easy for
you to configure it to output code and data in a way more to your liking.
You don't need to be an expert in pretty printer engines to figure out
how to alter its configuration.

Since I created zprint to be easily configured, there are *lots* of
configuration options as well as several different ways to configure zprint.

You mostly don't have to care about any of this unless you want to change
the way that zprint outputs your code or data.   If you do, read on...

[Overview](#overview)  
[How to Configure zprint](#how-to-configure-zprint)  
[Configuration Interface](#configuration-interface)  
[Option Validation](#option-validation)  
[What is Configurable](#what-is-configurable)  
[Generalized Capabilities](#generalized-capabilities)  
[Syntax Coloring](#syntax-coloring)  
[Function Classification for Pretty Printing](#function-classification-for-pretty-printing)  
[Changing or Adding Function Classifications](#changing-or-adding-function-classifications)  
[A note about two-up printing](#a-note-about-two-up-printing)  
[Widely Used Configuration Parameters](#widely-used-configuration-parameters)  


## Overview

The formatting done by zprint is driven off of an options map.
Zprint is built with an internal, default, options map.  This
internal options map is updated at the time that zprint is first
called by examining the .zprintrc file, environment variables, and
Java system properties.  You can update this internal options map
at any time by calling `set-options!` with a map containing the
parts of the options map you with to alter.  You can specify an
options map on any individual zprint or czprint call, and it will
only be used for the duration of that call.

When altering zprint's configuration by using a .zprintrc file,
calling `set-options!`, or specifying an options map on an individual call,
the things that you specify in the options map replace the current values
in the internal options map.  Only the specific values you specify
are changed -- you don't have to specify an entire sub-map when
configuring zprint.  When altering zprint's configuration using
environment variables or Java system properties, you specify a single
options map key path and value in each environment variable or Java
system property.

You can always see the current internal configuration of zprint by typing 
`(zprint nil :explain)` or `(czprint nil :explain)`.  In addition,
the `:explain` output will also show you how each element in the internal
options map received its current value.  Given the number of ways that
zprint can be configured, `(zprint nil :explain)` can be very useful
to sort out how a particular configuration element was configured with
its current value.  

The options map has a few top level configuration options, but
much of the configuration is grouped into sub-maps.  The top level
of the options map looks like this:

```clojure

{:agent {...},
 :array {...},
 :atom {...},
 :auto-width? false,
 :binding {...},
 :color-map {...},
 :comment {...},
 :delay {...},
 :extend {...},
 :fn-map {...},
 :fn-obj {...},
 :future {...},
 :list {...},
 :map {...},
 :max-depth 1000,
 :max-length 1000,
 :object {...},
 :pair {...},
 :pair-fn {...},
 :parse-string? false,
 :promise {...},
 :reader-cond {...},
 :record {...},
 :set {...},
 :style nil,
 :style-map {...},
 :tab {...},
 :uneval {...},
 :user-fn-map {...},
 :vector {...},
 :width 80,
 :zipper? false}
```

## How to Configure zprint 

When zprint is called for the first time it will configure itself
from all of the information that it has available at that time.
It will examine the following information in order to configure
itself:

* The file `$HOME/.zprintrc` for an options map in EDN format
* Environment variables for individual option map values
* Java properties for individual option map values

You can invoke the function `(configure-all!)` at any time to
cause zprint to re-examine the above information.  It will delete
any current configuration and rebuild it from the information
available at that time.

You can add configuration information by:

* Calling `set-options!` with an options map, which is saved in the internal 
options map across calls
* Specifing an options map on any call to zprint, which only affects that call
to `zprint` or `czprint`

## Configuration Interface

### .zprintrc

The `.zprintrc` file is simply a sparse options map in EDN format (see below).
That is to say, you only specify the elements that you wish to alter in
the `.zprintrc` file.  Thus, to change the indent for a map to be 0, you
would have a `.zprintrc` file as follows:

```clojure
{:map {:ident 0}}
```

Note that this map is only read and converted when zprint initially
configures itself, which is at the first use of a zprint or czprint
function.  You can invoke `configure-all!` at any later time which
will cause all of the external forms of configuration (e.g. .zprintrc,
environment variables, and Java system properties) to be read
and converted again.

#### Environment variables

You can specify an individual configuration element by specifying the
key as the environment variable name and the value as the value of the
environment variable.  For example, to specify the indent for a map
to be 0, you would say

`export zprint__map__indent=0`

which yields this change to the options map:

`{:map {:indent 0}}`

where a double underline `__` translates to a level in the map, and
a single underline `_` translates into a dash.  Thus

`export zprint__pair_fn__hang?=true`

will yield

`{:pair-fn {:hang? true}}`

Note that the strings `true` and `false` will be converted into their
boolean equivalents.

You can add (or change) functions in the :fn-map with environment variables.
Special processing is invoked when something inside the :fn-map is specified,
causing the final token of the environment variable to become a string and
the value of the environment variable is coerced to a keyword.  Thus, to
add the function `new-fn` as an `:arg1` function, use

`export zprint__fn_map__new_fn=arg1`

which will merge this map

`{:fn-map {"new-fn" :arg1}}`

into the existing `:fn-map`.
Alas, there is no way to use environment variables to affect a function
with an underscore in its function name.

The values of the environment variable are converted into EDN data.
Numbers become actual numbers, data structures become data structures, and
everything else becomes a string.  There are not a lot of data structures
in the options map, but the `:map :key-order` key takes a vector of
keys to place first when formatting a map.  You would specify the
following `:key-order`

`{:map {:key-order [:name "important"]}}`

like this

`export zprint__map__key_order='[:name "important"]'`

Note that these values are only read and converted when zprint initially
configures itself, which is at the first use of a zprint or czprint
function.  You can invoke `(configure-all!)` at any later time which
will cause all of the external forms of configuration (e.g. .zprintrc,
environment variables, and Java system properties) to be read and
converted again.  

#### Java Properties

You can also specify individual configuration elements using Java properties.
In Java properties, a dot is turned into a hypen (dash), and an underscore
represents a level in the map.
Thus,

`System.setProperty("zprint_map_indent" "0")`

will create the map

`{:map {:indent 0}}`
 
to be merged into the internal options map.
Similar to environment variables, numeric strings become numbers, strings
with the value `true` and `false` are turned into booleans, and 
you can specify a structure, so to set the `:key-order` 
to `["first" :second]` you would specify the Java property as:

`System.setProperty("zprint_map_key.order" "[\"first\" :second]")`

or in Clojure

`(System/setProperty "zprint_map_key.order" "[\"first\" :second]")`

Note that these values are only read and converted when zprint initially
configures itself, which is at the first use of a zprint or czprint
function.  You can invoke `(configure-all!)` at any later time which
will cause all of the external forms of configuration (e.g. .zprintrc,
environment variables, and Java system properties) to be read and
converted again.

#### set-options!

You call set-options! with an EDN map of the specific key-value pairs
that you want changed from the current values.  This is useful both
when using zprint at the REPL, as well as when you are using to output
information from a program that wants to configure zprint
in some particular way.  For example:

```clojure
(require '[zprint.core :as zp])

(zp/set-options! {:map {:indent 0}})
```

#### Options map on an individual call

You simply specify the options map on the call itself:

```clojure
(require '[zprint.core :as zp])

(def my-map {:stuff "a fairly long value" 
   :bother 
   "A much longer value, which makes this certainly not fit in 80 columns"})

(zp/zprint my-map {:map {:indent 0}})

{:bother
 "A much longer value, which makes this certainly not fit in 80 columns",
 :stuff "a fairly long value"}

```

### Option Validation

All changes to the options map are validated to some degree for
correctness.  When the change to the internal options map is itself
a map, when using the `.zprintrc` file, calling `(set-options! ...)`,
or specifying an options map on an individual call, every key in
the options map is validated, and some level of at least type 
validation on the values is also performed.  Thus:

```clojure
(czprint nil {:map {:hang false}})

Exception Option errors in this call: Value does not match schema: {:map {:hang disallowed-key}}  
zprint.core/zprint* (core.clj:241)

```

This call will fail validation because there is no `:hang` key in `:map`.  The
"?" is missing from `:hang?`.  My initial motivation for adding options
map validation was forgetting to type the "?" at the end of boolean valued
options and wondering why nothing changed.

There is no key validation performed for environment variables or
Java system properties -- invalid keys are simply ignored.  However, 
value type validation is performed.  Thus:

```clojure
(System/setProperty "zprint_map_indent" "true")

(configure-all!)

"In System property: Value does not match schema: {:map {:indent (not (instance? java.lang.Number true))}}"
```
whichs says that "true" is not an instance of java.lang.Number, and tells
you that any value for `{:map {:indent <value>}}` needs to be a number.

All option validation errors must be fixed, or zprint will not operate.

You can sidestep configuration issues by specifying `:default` as an
options map, which will cause zprint to use the built-in default options
map and not attempt any configuration.

## What is Configurable

The following categores of information are configurable:

* generalized capabilities
* syntax coloring
* function classification for pretty printing
* specific option values for maps, lists, vectors, pairs, bindings, arrays, etc.

### Generalized capabilites

#### :width

An integer width into which the formatted output should fit.  zprint
will work very hard to fit the formatted output into this width,
though there are limits to its effort.  For instance, it will not
reduce the minimum indents in order to satisfy a particular width
requirement.  This will be most obvious when widths are small, in
the 15 to 30 range.  Normally you might never notice this with the
default 80 column width.

Long strings will also cause zprint to exceed the requested width.
Comments will be wrapped by default so as not to exceed the width,
though you can disable comment wrapping.  See the `:comment` section.

The `:width` specification in the options map is most useful for
specifying the default width, as you can also give a width specification
as the second argument of any of the zprint functions.

#### :parse-string?

By default, zprint expects an s-expression and will format it.  If you
specify `:parse-string? true` in an options map, then the first argument
must be a string, and zprint will parse the string and format the output.

### Syntax coloring

Zprint will colorize the output when the czprint and czprint-fn calls
are used.  It is limited to the colors available on an ANSI terminal.

The key :color-map contains by default:

```clojure
 :color-map {:brace :red,
             :bracket :purple,
             :comment :green,
             :fn :blue,
             :hash-brace :red,
             :hash-paren :green,
             :keyword :magenta,
             :nil :yellow,
             :none :black,
             :number :purple,
             :paren :green,
             :quote :red,
             :string :red,
             :uneval :magenta,
             :user-fn :black},
```
You can change any of these to any other available value.  The
available values are:

* `:red`
* `:blue`
* `:green`
* `:magenta` (or `:purple`)
* `:yellow`
* `:cyan`
* `:black`

There is also a different color map for unevaluated items,
i.e. those prefaced with #_ and ignored by the Clojure reader.
This is the default :uneval color map:

```clojure
 :uneval {:color-map {:brace :yellow,
                      :bracket :yellow,
                      :comment :green,
                      :fn :cyan,
                      :hash-brace :yellow,
                      :hash-paren :yellow,
                      :keyword :yellow,
                      :nil :yellow,
                      :none :yellow,
                      :number :yellow,
                      :paren :yellow,
                      :quote :yellow,
                      :string :red,
                      :user-fn :cyan}},
```

You can also change these to any of the colors specified above.

Note that in this readme, the syntax coloring of Clojure code is 
that provided by the github flavored markdown, and not zprint.

### Function Classification for Pretty Printing

While most functions will pretty print without special processing,
some functions are more clearly comprehended when processed specially for
pretty printing.  Generally, if a function call fits on the current
line, none of these classifications matter.  These only come into play
when the function call doesn't fit on the current line.  The following
examples are shown with an implied width of well less than 80 columns
in order to demonstrate the function style in a concise manner.

Note that the 
[community style guide](https://github.com/bbatsov/clojure-style-guide)
specifies different indentation amounts for functions (forms) that have
"body" parameters, and functions that just have arguments.  Personally,
I've never really distinguished between these different types of functions
(which is why the default indent for both is 2).  But I've created
classifications so that you can class some functions as having body
arguments instead of just plain arguments, so that if you specify a
different indent for arg-type functions than body-type functions, the
right things will happen.

A function that is not classified explicitly by appearing in the
`:fn-map` is considered an "arg" function as opposed to "body" function,
and the indent for its arguments is controlled by `:list {:indent-arg n}` 
if it appears, and `:list {:indent n}` if it does not. 


The available classifications are:

#### :arg1

Print the first argument on the same line as the function, if possible.  
Later arguments are indented the amount specified by `:list {:indent-arg n}`, 
or `:list {:indent n}` if `:indent-arg` is not specified.

```clojure
 (apply str
   "prepend this one"
   (generate-strings from arguments))
```

#### :arg1-body

Print the first argument on the same line as the function, if possible.  
Later arguments are indented the amount specified by `:list {:indent n}`.

```clojure
 (if (= a 1)
   (map inc coll)
   (map dec coll))
```
#### :arg1-pair

The function has an important first argument, then the rest of the 
arguments are paired up. Leftmost part of the pair is indented
by `:list {:indent-arg n}` if it is specified, and `:list {:indent n}` 
if it is not.

```clojure
 (assoc my-map
   :key1 :val1
   :key2 :val2)
```
#### :arg1-pair-body

The function has an important first argument, then the rest of the 
arguments are paired up.  The leftmost part of the pair is indented
by the amount specified by `:list {:indent n}`.

```clojure
 (case fn-style
   :arg1 nil
   :arg1-pair :pair
   :arg1-extend :extend
   :arg2 :arg1
   :arg2-pair :arg1-pair
   fn-style)
```

#### :arg2
 
Print the first argument on the same line as the function name if it will
fit on the same line. If it does, print the second argument
on the same line as the first argument if it fits. Indentation of
later arguments is controlled by `:list {:indent n}`

```clojure
  (as-> initial-value tag
    (process stuff tag bother)
    (more-process tag foo bar))
```

#### :arg2-pair

Just like :arg2, but prints the third through last arguments as pairs.
Indentation of the leftmost elements of the pairs is controlled by
`:list {:indent n}`.  If any of the rightmost elements end up not fitting
or not hanging well, the flow indent is controlled by `:pair {:indent n}`.

```clojure
  (condp = stuff
    :bother "bother"
    :foo "foo"
    :bar "bar"
    "baz")
```

#### :binding

The function has a binding clause as its first argument.  
Print the binding clause two-up (as pairs)  The indent for any wrapped
binding element is :binding `{:indent n}`, the indent for the functions
executed after the binding is `:list {:indent n}`.

```clojure
 (let [first val1 
       second
         (calculate second using a lot of arguments)
       c d]
   (+ a c))
```

#### :pair-fn

The function has a series of clauses which are paired.  Whether or
not the paired clauses use hang or flow is controlled by
`:pair-fn {:hang? boolen}` and the indent of the leftmost
element is controlled by `:pair-fn {:indent n}`.  The indent of any
of the rightmost elements of the pair if they don't fit on the
same line or don't hang well is `:pair {:indent n}`.

```clojure
 (cond
   (and (= a 1) (> b 3)) (vector c d e)
   (= d 4) (inc a))
```

#### :hang

The function has a series of arguments where it would be nice
to put the first on the same line as the function and then
indent the rest to that level.  This would usually always be nice,
but zprint tries extra hard for these.  The indent when the arguments
don't hang well is `:list {:indent n}`.

```clojure
 (and (= i 1)
      (> (inc j) (stuff k)))
```

#### :extend

The s-expression has a series of symbols with one or more forms 
following each.  The level of indent is configurable by `:extend {:indent n}`.

```clojure
  (reify
    stuff
      (bother [] (println))
    morestuff
      (really [] (print x))
      (sure [] (print y))
      (more-even [] (print z)))
```

#### :arg1-extend

For the several functions which have an single argument
prior to the :extend syntax.  They must have one argument,
and if the second argument is a vector, it is also handled
separately from the :extend syntax.  The level of indent is controlled
by `:extend {:indent n}`

```clojure
  (extend-protocol ZprintProtocol
    ZprintType
      (more-stuff [x] (str x))
      (more-bother [y] (list y))
      (more-foo [z] (nil? z))))

  (deftype ZprintType
    [a b c]
    ZprintProtocol
      (stuff [this x y] a)
      (bother [this] b)
      (bother [this x] (list x c))
      (bother [this x y] (list x y a b)))
```

#### :fn

Print the first argument on the same line as the `(fn ...)` if it will
fit on the same line. If it does, and the second argument is a vector, 
print it on the same line as the first argument if it fits.  Indentation
is controlled by `:list {:indent n}`.

```clojure
  (fn [a b c]
    (let [d c]
      (inc d)))

  (fn myfunc [a b c]
    (let [d c]
      (inc d)))
```

#### :none

This is for things like special forms that need to be in this
map to show up as functions for syntax coloring, but don't actually 
trigger the function recognition logic to represent them as such.
Also, `:none` used to remove the default classification for functions
by specifying it in an option map.  The indent for arguments that
don't hang or fit on the same line is `:list {:indent-arg n}`
if it is specified, and `:list {:indent n}` if it is not.

#### :none-body

Like none, but the indent for arguments that don't hang or fit
on the same is always `:list {:indent n}`.

### Changing or Adding Function Classifications

You can change the classification of an existing function or add
a new one by changing the map at key :fn-map.  A fragment of the existing
map is shown below:

```clojure
:fn-map {"!=" :hang,
          "->" :arg1,
          "->>" :arg1,
          "=" :hang,
          "and" :hang,
          "apply" :arg1,
          "assoc" :arg1-pair,
          "binding" :binding,
          "case" :arg1-pair,
          "catch" :none,
          "cond" :pair,
	  ...}
```

Note that the function names are strings.  You can add any function
you wish to the :fn-map, and it will be interpreted as described above.

Often the :fn-map is configured by changing the `.zprintrc` file so 
that functions are formattted the way you prefer.  You can change the
default formatting of functions as well as configure formatting for
your own functions.  To remove formating for a function which has
previously been configured, set the formatting to `:none`.

### Detailed Configuration for maps, lists, vectors, etc.

Internally, there are several formatting capabilities that are
used in slightly different ways to format a wide variety of syntactic
elements.  These basic capabilities are parameterized, and the
parameters are varied based on the syntactic element.  Before going
into detail about the individual elements, let's look at the overview
of the capabilities:

* two-up (pairs (or more) of things that go together)
    * :binding
    * :map
    * :pair
    * :extend
    * :reader-cond
* vector (wrap things out to the margin)
    * :vector
    * :set
    * :array
* list (things that might be code)
    * :list
* objects with values (format nicely or print as object)
    * :agent
    * :atom
    * :delay
    * :fn (function objects, not functions in code)
    * :promise
    * :object 
* misc
    * :record

#### A note about two-up printing

Part of the reason for zprint's existence revolves around the
current approach to indenting used for cond clauses, binding vectors,
and maps and other things with pairs (extend and reader conditionals).

Back in the day some of the key functions that include pairs, e.g.
cond and let, had their pairs nested in parentheses.  Clojure doesn't
follow this convention, which does create cleaner looking code in
the usual case, when the second part of the pair is short and fits
on the same line or when the second part of the pair can be represented
in a hang.  In those cases when the second part of the pair ends
up on the next line (as a flow), it can sometimes become a bit
tricky to separate the test and expr pair in a cond, or a destructured
binding-form from the init-expr, as they will start in the same
column.

While the cases where it is a bit confusing are rather rare, I
find them bothersome, so by default zprint will indent the
second part of these pairs by 2 columns (controlled by `:pair {:indent 2}`
for `cond` and `:binding {:indent 2}` for binding functions).

Maps also have pairs, and perhaps suffer from the potential
for confusion a bit more then binding-forms and cond functions.
By default then, the map indent for the an item that placed on the
next line (i.e., in a flow) is 2 (controlled by `:map {:indent 2}`).  
The default is 2 for
extend and reader-conditionals as well.

Is this perfect?  No, there are opportunities for confusion here
too, but it works considerably better for me, and it might for
you too. I find this particularly useful for :binding and :map
formatting.

Should you not like what this does to your code or your s-expressions,
the simple answer is to use {:style :community} as an options-map
when calling zprint (specify that in your .zprintrc file, perhaps).

You can change the indent from the default of 2 to 0 individually
in :binding, :map, or :pair if you want to tune it in more detail.

#### A note on justifying two-up printing

I have seen some code where people justify the second element of their
pairs to all line up in the same column.  I call this justifying for
lack of a better term.  Here is an example in code:

```clojure
; Regular formatting

(zprint-fn compare-ordered-keys {:pair {:justify? true}})

(defn compare-ordered-keys
  "Do a key comparison that places ordered keys first."
  [key-value zdotdotdot x y]
  (cond (and (key-value x) (key-value y)) (compare (key-value x) (key-value y))
        (key-value x) -1
        (key-value y) +1
        (= zdotdotdot x) +1
        (= zdotdotdot y) -1
        :else (compare-keys x y)))

; Justified formatting

(zprint-fn compare-ordered-keys {:pair {:justify? true}})

(defn compare-ordered-keys
  "Do a key comparison that places ordered keys first."
  [key-value zdotdotdot x y]
  (cond (and (key-value x) (key-value y)) (compare (key-value x) (key-value y))
        (key-value x)                     -1
        (key-value y)                     +1
        (= zdotdotdot x)                  +1
        (= zdotdotdot y)                  -1
        :else                             (compare-keys x y)))
```
Zprint will optionally justify `:map`, `:binding`, and `:pair` elements.
There are several detailed configuration parameters used to control the
justification.  Obviously this works best if the keys in a map are
all about the same length (and relatively short), and the test expressions
in a code are about the same length, and the local in a binding are
about the same length.

I don't personally find the justified approach my favorite in code,
though there are some functions where it looks good.  They are a good
candidate for a directive to lein-zprint:

```clojure
;!zprint {:format :next {:style :justified}}
```

As you might gather, there is a `:style :justified` which you can use
to turn this on for maps, pairs, and bindings.

I was surprised what justification could do for some maps, however.
You can see it for yourself if you enter:

```clojure
(czprint nil :explain-justified)
```

See what you think.

__NOTE:__ Justification involves extra processing, and because of the way
that zprint tries to do the best job possible, it can cause a bit of a
combinatorial explosion that can make formatting some functions and
structures take a very long time.  I have put scant effort into optimizing
this capability, as I have no idea how interesting it is to people in
general.  If you are using it and like it, and you have situations where
it seems to be particularly slow for you, please enter an issue to let
me know.

## Widely Used Configuration Parameters

There are a several  configuration parameters that are meaningful
across a number of formatting types.  

### :indent

The value for indent is how far to indent the second through nth of 
something if it doesn't all fit on one line (and becomes a flow, see
immediately below).

### hang and flow

zprint uses two concepts: hang and flow, to describe how something is
to be printed.

This is a hang:

```clojure
(symbol "string"
        :keyword
        5
        {:map-key :value})
```

This is the same information in a flow:

```clojure
(symbol
  "string"
  :keyword
  5
  {:map-key :value})
```
zprint will try (by default) to use the *hang* approach when it
will use the same or fewer lines than a *flow*.  Unless the hang takes
too much vertical space (which makes things less clear, instead of more
clear).  There are several values which will tune the output for
hang and flow.

#### :hang?

If :hang? is true, zprint will attempt to hang.  If it is false, it won't
even try.

#### :hang-expand

`:hang-expand` one control used to decide whether or not to do a hang.  It
relates the number of lines in the hang to the number of elements
in the hang thus: `(/ (dec hang-lines) hang-element-count)`.  If every
element in the hang fits on one line, then this ratio will be < 1.
If every element in the hang takes two lines, then this ratio will
be close to 2.  If this ratio is > `:hang-expand`, then the hang
is rejected.  The idea is that hangs that run on and on down the
right side of the page are not ideal, even when they don't take
more lines than a flow.  Unless, in some cases, they are ok -- for
instance for maps.  The `:hang-expand` for :map is 1000.0, since
we expect maps to have large hangs that expand a lot.

#### :hang-diff

The value of `:hang-diff` (an integer) is related to the indent for
a hang and a flow.  Clearly, if the indent for a hang and a flow are
the same, you might as well do a hang, since a flow buys you nothing.
The difference in these indents `(- hang-indent flow-indent)` is compared
to the value of `:hang-diff`, and if this difference is <= then it 
just does the hang regardless.  `:hang-diff` is by default 1, since even if a
flow buys you one more space to the left, it often looks kind of odd.
You could set `:hang-diff` to 0 if you wanted to be more "strict", and
see if you like the results better.  Probably you won't want to deal
with this level of control.

#### :justify?

Turn on [justification](#a-note-on-justifying-two-up-printing).
Default is nil (justification off).

_____
## :map

Maps support both the __indent__ and __hang__ values, above.  The default
`:hang-expand` value is `1000.0` because maps  don't look bad with a large
hangs.

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>1000.0</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>
##### :justify? <text style="color:#A4A4A4;"><small>false</small></text>

####  :comma?  <text style="color:#A4A4A4;"><small>true</small></text>

Put a comma after the value in a key-value pair, if it is not the
last pair in a map.  

#### :force-nl? <text style="color:#A4A4A4;"><small>false</small></text>

Force new-lines between each key and value in a map.  Will never print
the value on the same line as the key. 

#### :sort? <text style="color:#A4A4A4;"><small>true</small></text>

Sort the key-value pairs in a map prior to output.  Alternatively, simply output
them in the order in which they come out of the map. 

#### :sort-in-code? <text style="color:#A4A4A4;"><small>false</small></text>

If the map appears inside of a list that seems to be code, should it
be sorted.  

#### :key-order <text style="color:#A4A4A4;"><small>nil</small></text>

Accepts a vector which contains keys which should sort before all
other keys.  Typically these keys would be keywords, strings, or
integers.  The value of this capability is to bring one or more
key-value pairs to the top of a map when it is output, in order to
aid in visually distinguishing one map from the other.  This can
be a significant help in debugging, when looking a lot of maps at
the repl.  Note that `:key-order` only affects the key order when
keys are sorted.

Here is a vector of maps formatted with just sorting. 

```clojure
[{:code "58601",
  :connection "2795",
  :detail {:alternate "64:c1:2f:34",
           :ident "64:c1:2f:34",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :received,
  :reference 14873,
  :time 1425704001,
  :type "get"}
 {:code "0xe4e9",
  :connection "X13404",
  :detail
    {:code "0xe4e9", :ident "64:c1:2f:34", :ip4 "3.13.168.151", :time "30m"},
  :direction :transmitted,
  :reference 14133,
  :time 1425704001,
  :type "post"}
 {:code "58601",
  :connection "X13404",
  :direction :transmitted,
  :reference 14227,
  :time 1425704001,
  :type "post"}
 {:code "0x1344a676",
  :connection "2796",
  :detail {:code "0x1344a676", :ident "50:56:a5:1d:61", :ip4 "3.13.171.81"},
  :direction :received,
  :reference 14133,
  :time 1425704003,
  :type "error"}
 {:code "323266166",
  :connection "2796",
  :detail {:alternate "01:50:56:a5:1d:61",
           :ident "50:56:a5:1d:61",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :transmitted,
  :reference 14873,
  :time 1425704003,
  :type "error"}]
```

Lots of information -- at least it is sorted.
But the type and the direction are the important parts if you
were scanning this at the repl, and they are buried pretty deep.

If you were looking for received errors, and needed to see them
in context, you might prefer the following presentation...

Using the following options map:

```clojure
{:map {:key-order [:type :direction]}})
```
yields:

```clojure
[{:type "get",
  :direction :received,
  :code "58601",
  :connection "2795",
  :detail {:alternate "64:c1:2f:34",
           :ident "64:c1:2f:34",
           :interface "3.13.168.35",
           :string "datacenter"},
  :reference 14873,
  :time 1425704001}
 {:type "post",
  :direction :transmitted,
  :code "0xe4e9",
  :connection "X13404",
  :detail
    {:code "0xe4e9", :ident "64:c1:2f:34", :ip4 "3.13.168.151", :time "30m"},
  :reference 14133,
  :time 1425704001}
 {:type "post",
  :direction :transmitted,
  :code "58601",
  :connection "X13404",
  :reference 14227,
  :time 1425704001}
 {:type "error",
  :direction :received,
  :code "0x1344a676",
  :connection "2796",
  :detail {:code "0x1344a676", :ident "50:56:a5:1d:61", :ip4 "3.13.171.81"},
  :reference 14133,
  :time 1425704003}
 {:type "error",
  :direction :transmitted,
  :code "323266166",
  :connection "2796",
  :detail {:alternate "01:50:56:a5:1d:61",
           :ident "50:56:a5:1d:61",
           :interface "3.13.168.35",
           :string "datacenter"},
  :reference 14873,
  :time 1425704003}]
```

When working with hundreds of maps, even the tiny improvement 
made by ordering a few keys in a better way can reduce the cognitive
load, particularly when debugging.

#### :key-ignore <text style="color:#A4A4A4;"><small>nil</small></text>
#### :key-ignore-silent <text style="color:#A4A4A4;"><small>nil</small></text>

You can also ignore keys (or key sequences) in maps when formatting
them.  There are two basic approaches.  `:key-ignore` will replace
the value of the key(s) to be ignored with `:zprint-ignored`, where
`:key-ignore-silent` will simply remove them from the formatted output.

You might use this to remove sensitive information from output, or
to remove elements that have more data than you wish to display.

You can also supply key-sequences, in addition to single keys, to
either configuration parameter.

An example of the basic approach.  First the unmodified data:

```clojure
zprint.core=> (czprint sort-demo)

[{:code "58601",
  :connection "2795",
  :detail {:alternate "64:c1:2f:34",
           :ident "64:c1:2f:34",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :received,
  :reference 14873,
  :time 1425704001,
  :type "get"}
 {:code "0xe4e9",
  :connection "X13404",
  :detail
    {:code "0xe4e9", :ident "64:c1:2f:34", :ip4 "3.13.168.151", :time "30m"},
  :direction :transmitted,
  :reference 14133,
  :time 1425704001,
  :type "post"}
 {:code "58601",
  :connection "X13404",
  :direction :transmitted,
  :reference 14227,
  :time 1425704001,
  :type "post"}
 {:code "0x1344a676",
  :connection "2796",
  :detail {:code "0x1344a676", :ident "50:56:a5:1d:61", :ip4 "3.13.171.81"},
  :direction :received,
  :reference 14133,
  :time 1425704003,
  :type "error"}
 {:code "323266166",
  :connection "2796",
  :detail {:alternate "01:50:56:a5:1d:61",
           :ident "50:56:a5:1d:61",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :transmitted,
  :reference 14873,
  :time 1425704003,
  :type "error"}]
```

Here is the data with the `:detail` key ignored:

```clojure
zprint.core=> (czprint sort-demo {:map {:key-ignore [:detail]}})

[{:code "58601",
  :connection "2795",
  :detail :zprint-ignored,
  :direction :received,
  :reference 14873,
  :time 1425704001,
  :type "get"}
 {:code "0xe4e9",
  :connection "X13404",
  :detail :zprint-ignored,
  :direction :transmitted,
  :reference 14133,
  :time 1425704001,
  :type "post"}
 {:code "58601",
  :connection "X13404",
  :direction :transmitted,
  :reference 14227,
  :time 1425704001,
  :type "post"}
 {:code "0x1344a676",
  :connection "2796",
  :detail :zprint-ignored,
  :direction :received,
  :reference 14133,
  :time 1425704003,
  :type "error"}
 {:code "323266166",
  :connection "2796",
  :detail :zprint-ignored,
  :direction :transmitted,
  :reference 14873,
  :time 1425704003,
  :type "error"}]
```

The same as above, with `:key-ignore-silent` instead of `key-ignore`:

```clojure
zprint.core=> (czprint sort-demo {:map {:key-ignore-silent [:detail]}})

[{:code "58601",
  :connection "2795",
  :direction :received,
  :reference 14873,
  :time 1425704001,
  :type "get"}
 {:code "0xe4e9",
  :connection "X13404",
  :direction :transmitted,
  :reference 14133,
  :time 1425704001,
  :type "post"}
 {:code "58601",
  :connection "X13404",
  :direction :transmitted,
  :reference 14227,
  :time 1425704001,
  :type "post"}
 {:code "0x1344a676",
  :connection "2796",
  :direction :received,
  :reference 14133,
  :time 1425704003,
  :type "error"}
 {:code "323266166",
  :connection "2796",
  :direction :transmitted,
  :reference 14873,
  :time 1425704003,
  :type "error"}]
```

An example of the key-sequence approach.  This will remove all
of the elements with key `:code` inside of the maps with the key
`:detail`, but not the elements with the key `:code` elsewhere.
This example uses `:key-ignore` so you can see where it removed
values.  

```clojure
zprint.core=> (czprint sort-demo {:map {:key-ignore [[:detail :code]]}})

[{:code "58601",
  :connection "2795",
  :detail {:alternate "64:c1:2f:34",
           :ident "64:c1:2f:34",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :received,
  :reference 14873,
  :time 1425704001,
  :type "get"}
 {:code "0xe4e9",
  :connection "X13404",
  :detail {:code :zprint-ignored,
           :ident "64:c1:2f:34",
           :ip4 "3.13.168.151",
           :time "30m"},
  :direction :transmitted,
  :reference 14133,
  :time 1425704001,
  :type "post"}
 {:code "58601",
  :connection "X13404",
  :direction :transmitted,
  :reference 14227,
  :time 1425704001,
  :type "post"}
 {:code "0x1344a676",
  :connection "2796",
  :detail {:code :zprint-ignored, :ident "50:56:a5:1d:61", :ip4 "3.13.171.81"},
  :direction :received,
  :reference 14133,
  :time 1425704003,
  :type "error"}
 {:code "323266166",
  :connection "2796",
  :detail {:alternate "01:50:56:a5:1d:61",
           :ident "50:56:a5:1d:61",
           :interface "3.13.168.35",
           :string "datacenter"},
  :direction :transmitted,
  :reference 14873,
  :time 1425704003,
  :type "error"}]
```



_____
## :reader-cond

Reader conditionals are controlled by the `:reader-cond` key.  All
of the keys which are supported for `:map` are supported for `:reader-cond`
(except `:comma?`), albeit with different defaults.  By default,
`:sort?` is nil, so the elements are not reordered.  You could enable
`:sort?` and specify a `:key-order` vector to order the elements of a
reader conditional.

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>1000.0</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>
##### :force-nl? <text style="color:#A4A4A4;"><small>false</small></text>
##### :sort? <text style="color:#A4A4A4;"><small>false</small></text>
##### :sort-in-code? <text style="color:#A4A4A4;"><small>false</small></text>
##### :key-order <text style="color:#A4A4A4;"><small>nil</small></text>

_____
## :extend

When formatting functions which have extend in their function types.
Supports __indent__ as described above.  

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>

#### :force-nl?  <text style="color:#A4A4A4;"><small>true</small></text>

Forces a new line between the elements of the extend. 

_____
## :vector

Supports __indent__ as described above.  

##### :indent <text style="color:#A4A4A4;"><small>1</small></text>

Vectors wrap their contents, as distinct from maps and lists,
which use hang or flow.  Wrapping means that they will fill out
a line and then continue on the next line.

```clojure
(require '[zprint.core :as zp])

(zp/zprint (vec (range 60)) 70)

[0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25
 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48
 49 50 51 52 53 54 55 56 57 58 59]
```

#### :wrap-coll? <text style="color:#A4A4A4;"><small>true</small></text>

If there is a collection in the vector, should it wrap? 

```clojure
(require '[zprint.core :as zp])

(def v (vec (concat (range 10) '({:key :value "key" "value"}) (range 10 20))))

(zp/zprint v 60)

[0 1 2 3 4 5 6 7 8 9 {:key :value, "key" "value"} 10 11 12
 13 14 15 16 17 18 19]

(zp/zprint v 60 {:vector {:wrap-coll? nil}})

[0
 1
 2
 3
 4
 5
 6
 7
 8
 9
 {:key :value, "key" "value"}
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19]
```
#### :wrap-after-multi? <text style="color:#A4A4A4;"><small>true</small></text>

Should a vector continue to wrap after a multi-line element is 
printed? 

```clojure
(require '[zprint.core :as zp])

(def u (vec (concat (range 10) '({:key :value "key" "value" :stuff 
"the value of stuff is hard to quantify" :bother 
"few people enjoy being bothered"}) (range 10 20))))

(zp/zprint u)

[0 1 2 3 4 5 6 7 8 9
 {:bother "few people enjoy being bothered",
  :key :value,
  :stuff "the value of stuff is hard to quantify",
  "key" "value"} 10 11 12 13 14 15 16 17 18 19]

(zp/zprint u {:vector {:wrap-after-multi? nil}})

[0 1 2 3 4 5 6 7 8 9
 {:bother "few people enjoy being bothered",
  :key :value,
  :stuff "the value of stuff is hard to quantify",
  "key" "value"}
 10 11 12 13 14 15 16 17 18 19
```
_____
## :set

`:set` supports exactly the same keys as does vector.

##### :indent <text style="color:#A4A4A4;"><small>1</small></text>
##### :wrap-coll? <text style="color:#A4A4A4;"><small>true</small></text>
##### :wrap-after-multi? <text style="color:#A4A4A4;"><small>true</small></text>

_____
## :list

:list supports __indent__ and __hang__ described above.

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>2.0</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>

#### :indent-arg <text style="color:#A4A4A4;"><small>nil</small></text>

The amount to indent the arguments of a function whose arguments do
not contain "body" forms.
See [here](#function-classification-for-pretty-printing)
for an explanation of what this means.  If this is nil, then the value
configured for `:indent` is used for the arguments of functions that
are not "body" functions.  You would configure this value only if
you wanted "arg" type functions to have a different indent from
"body" type functions.  It is configured by `:style :community`.

#### :hang-size <text style="color:#A4A4A4;"><small>100</small></text>

The maximum number of lines that are allowed in a hang.  If the number
of lines in the hang is greater than the `:hang-size`, it will not do
the hang but instead will format this as a flow.  Together with
`:hang-expand` this will keep hangs from getting too long so that
code (typically) doesn't get very distorted.

#### :constant-pair <text style="color:#A4A4A4;"><small>true</small></text>

Lists (which are frequently code) support something called _**constant
pairing**_.  This capability looks at the end of a list, and if the
end of the list appears to contain pairs of constants followed by
anything, it will print them paired up.  A constant in this context
is a keyword, string, or number.  An example will best illustrate
this.

We will use a feature of zprint, where it will parse a string prior to
formatting, so that the anonymous functions show up right.

```clojure
(require '[zprint.core :as zp])

(def x "(s/fdef spec-test\n :args (s/and (s/cat :start integer? :end integer?)\n #(< (:start %) (:end %)))\n :ret integer?\n :fn (s/and #(>= (:ret %) (-> % :args :start))\n #(< (:ret %) (-> % :args :end))))\n")

;;
;; Without constant pairing, it is ok...
;; 

(zp/zprint x 60 {:parse-string? true :list {:constant-pair? nil}})

(s/fdef spec-test
        :args
        (s/and (s/cat :start integer? :end integer?)
               #(< (:start %) (:end %)))
        :ret
        integer?
        :fn
        (s/and #(>= (:ret %) (-> % :args :start))
               #(< (:ret %) (-> % :args :end))))

;;
;; With constant pairing it is nicer
;;

(zp/zprint x 60 {:parse-string? true :list {:constant-pair true}})

(s/fdef spec-test
        :args (s/and (s/cat :start integer? :end integer?)
                     #(< (:start %) (:end %)))
        :ret integer?
        :fn (s/and #(>= (:ret %) (-> % :args :start))
                   #(< (:ret %) (-> % :args :end))))

;;
;; We can demonstrate another configuration capability here.
;; If we tell zprint that s/fdef is an :arg1 style function, it is better
;; (note that :constant-pair? true is the default).
;;

(zp/zprint x 60 {:parse-string? true :fn-map {"s/fdef" :arg1}})

(s/fdef spec-test
  :args (s/and (s/cat :start integer? :end integer?)
               #(< (:start %) (:end %)))
  :ret integer?
  :fn (s/and #(>= (:ret %) (-> % :args :start))
             #(< (:ret %) (-> % :args :end))))
```
Constant pairing tends to make keyword style arguments come out
looking rather better than they would otherwise.  This feature was added
to handle what I believed was a very narrow use case, but it has shown
suprising generality, making unexpected things look much better.

In particular, try it on your specs!

#### :constant-pair-min <text style="color:#A4A4A4;"><small>4</small></text>
 
An integer specifying the minimum number of required elements capable of being
constant paired before constant pairing is used.  Note that constant
pairing works from the end of the list back toward the front (not illustrated
in these examples).  

Using our previous example again:

```clojure
(require '[zprint.core :as zp])

(def x "(s/fdef spec-test\n :args (s/and (s/cat :start integer? :end integer?)\n #(< (:start %) (:end %)))\n :ret integer?\n :fn (s/and #(>= (:ret %) (-> % :args :start))\n #(< (:ret %) (-> % :args :end))))\n")

;;
;; There are 6 elements that can be constant paired
;;

(zp/zprint x 60 {:parse-string? true :list {:constant-pair-min 6}})

(s/fdef spec-test
        :args (s/and (s/cat :start integer? :end integer?)
                     #(< (:start %) (:end %)))
        :ret integer?
        :fn (s/and #(>= (:ret %) (-> % :args :start))
                   #(< (:ret %) (-> % :args :end))))

;;
;; So, if we change the requirements to 8, it won't constant-pair
;;

(zp/zprint x 60 {:parse-string? true :list {:constant-pair-min 8}})

(s/fdef spec-test
        :args
        (s/and (s/cat :start integer? :end integer?)
               #(< (:start %) (:end %)))
        :ret
        integer?
        :fn
        (s/and #(>= (:ret %) (-> % :args :start))
               #(< (:ret %) (-> % :args :end))))
```
_____
## :binding

Controls the formatting of the first argument of
any function which has `:binding` as its function type.  `let` is, of
course, the canonical example.  :binding supports __indent__ and
__hang__ described above.

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>
##### :justify? <text style="color:#A4A4A4;"><small>false</small></text>

_____
## :pair

The :pair key controls the printing of the arguments of a function
which has -pair in its function type (e.g. :arg1-pair).  `:pair` 
supports __indent__ and __hang__ described above. 

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>
##### :justify? <text style="color:#A4A4A4;"><small>false</small></text>

#### :force-nl? <text style="color:#A4A4A4;"><small>false</small></text>

If you wish to force a newline between the things that are paired.

_____
## :pair-fn

The :pair key controls the printing of the arguments of a function
which has :pair-fn as its function type (e.g. `cond`).  `:pair-fn` 
supports __indent__ and __hang__ described above. 

##### :indent <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang? <text style="color:#A4A4A4;"><small>true</small></text>
##### :hang-expand <text style="color:#A4A4A4;"><small>2</small></text>
##### :hang-diff <text style="color:#A4A4A4;"><small>1</small></text>

#### :force-nl? <text style="color:#A4A4A4;"><small>false</small></text>

If you wish to force a newline between the things that are paired in a
pair-fn.

_____
## :record

Records are printed with the record-type and value of the record
shown with map syntax, or by calling their `toString()` method.

#### :to-string? <text style="color:#A4A4A4;"><small>false</small></text>

This will output a record by calling its `toString()` java method, which
can be useful for some records. If the record contains a lot of information
that you didn't want to print, for instance. If `:to-string?` is true, 
it overrides the other `:record` configuration options.

#### :hang? <text style="color:#A4A4A4;"><small>true</small></text>

Should a hang be attempted?  See example below. 

#### :record-type? <text style="color:#A4A4A4;"><small>true</small></text>

Should the record type be output?  

An example of `:hang?`, `:record-type?`, and `:to-string?`

```clojure
(require '[zprint.core :as zp])

(defrecord myrecord [left right])

(def x (new myrecord ["a" "lot" "of" "stuff" "so" "that" "it" "doesn't" "fit" "all" "on" "one" "line"] [:more :stuff :but :not :quite :as :much]))

(zp/zprint x 75)

#zprint.core/myrecord {:left ["a" "lot" "of" "stuff" "so" "that" "it"
                              "doesn't" "fit" "all" "on" "one" "line"],
                      :right [:more :stuff :but :not :quite :as :much]}

(zp/zprint x 75 {:record {:hang? nil}})

#zprint.core/myrecord
 {:left ["a" "lot" "of" "stuff" "so" "that" "it" "doesn't" "fit" "all" "on"
         "one" "line"],
  :right [:more :stuff :but :not :quite :as :much]}

(zp/zprint x 75 {:record {:record-type? nil}})

{:left ["a" "lot" "of" "stuff" "so" "that" "it" "doesn't" "fit" "all" "on"
        "one" "line"],
 :right [:more :stuff :but :not :quite :as :much]}

(zprint x {:record {:to-string? true}})

"zprint.core.myrecord@682a5f6b"
```
_____
## :array

Arrays are formatted by default with the values of their elements.

#### :hex? <text style="color:#A4A4A4;"><small>false</small></text>

If the elements are numeric, format them in hex. Useful if you are 
doing networking.  See below for an example. 

#### :object? <text style="color:#A4A4A4;"><small>false</small></text>

Don't print the elements of the array, just print it as an 
object. 

A simple example:

```clojure
(require '[zprint.core :as zp])

(def ba (byte-array (range 50)))

(zp/zprint ba 75)

[0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27
 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49]

(zp/zprint ba 75 {:array {:hex? true}})

[00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f 10 11 12 13 14 15 16 17 18
 19 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30
 31]

;; As an aside, notice that the 8 in 18 was in column 75, and so while the
;; 31 would have fit, the ] would not, so they go on the next line.

(zp/zprint ba 75 {:array {:object? true}})

#object["[B" "0x31ef8e0b" "[B@31ef8e0b"]
```

______
## :agent, :atom, :delay, :fn, :future, :promise 

All of these elements are formatted in a readable manner by default,
which shows their current value and minimizes extra information.

#### :object? <text style="color:#A4A4A4;"><small>false</small></text>

All of these elements can be formatted more as Clojure represents
Java objects by setting `:object?` to true.  

_____
## :object

When elements are formatted with `:object?` `true`, then the output
if formatted using the information specified in the `:object` 
information.  :object supports __indent__ as described above.  

##### :indent <text style="color:#A4A4A4;"><small>1</small></text>

_____
## :comment

zprint has two fundemental regimes -- printing s-expressions and
parsing a string and printing the result.  There are no comments
in s-expressions, except in the `comment` function, which is handled
normally. When parsing a string, zprint will deal with comments.
Comments are dealt with in one of two ways -- either they are ignored
from a width standpoint while formatting, or their width is taken
into account when formatting.  In addition, comments can be
word-wrapped if they don't fit the width, or not.  These are
indpendent capabilities.

#### :wrap? <text style="color:#A4A4A4;"><small>true</small></text>

Wrap a comment if it doesn't fit within the width.  Works hard to preserve
the initial part of the line.  

#### :count? <text style="color:#A4A4A4;"><small>false</small></text>

Count the length of the comment when ensuring that things fit within the
width.  Tends to mess up the code more than helping, in my view.  

An example (using :parse-string? true to include the comment):

```clojure
(require '[zprint.core :as zp])

(def cd "(let [a (stuff with arguments)] (list (or foo bar baz) (format output now) (and a b c (bother this)) ;; Comment that doesn't fit real well, but is almost a fit to see how it works\n (format other stuff))(list a :b :c \"d\"))")

(zp/zprint cd 75 {:parse-string? true :comment {:count? nil}})

(let [a (stuff with arguments)]
  (list (or foo bar baz)
        (format output now)
        (and a b c (bother this))
        ;; Comment that doesn't fit real well, but is almost a fit to see
        ;; how it works
        (format other stuff))
  (list a :b :c "d"))

zprint.core=> (czprint cd 75 {:parse-string? true :comment {:count? true}})

(let [a (stuff with arguments)]
  (list
    (or foo bar baz)
    (format output now)
    (and a b c (bother this))
    ;; Comment that doesn't fit real well, but is almost a fit to see how
    ;; it works
    (format other stuff))
  (list a :b :c "d"))

(zp/zprint cd 75 {:parse-string? true :comment {:count? nil :wrap? nil}})

(let [a (stuff with arguments)]
  (list (or foo bar baz)
        (format output now)
        (and a b c (bother this))
        ;; Comment that doesn't fit real well, but is almost a fit to see how it works
        (format other stuff))
  (list a :b :c "d"))
```
______
## :tab

zprint will expand tabs by default when parsing a string, largely in order
to properly size comments.  You can disable tab expansion and you can
set the tab size.

#### :expand? <text style="color:#A4A4A4;"><small>true</small></text>

Expand tabs.  

#### :size <text style="color:#A4A4A4;"><small>8</small></text>

An integer for the tab size for tab expansion.  

______
## :style and :style-map

You can define your own styles, by adding elements to the `:style-map`.
You can do this the same way you make other configuration changes,
but probably you want to define a style in the .zprintrc file, and
then use it elsewhere.

You can see the existing styles in the `:style-map`, and you would
just add your own along the same lines.  The map associated with a
style must validate successfully just as if you used it as an options
map in an individual call to zprint.

You might wish to define several styles with different color-maps,
perhaps, allowing you to alter the colors more easily.

______
______
## Debugging the configuration

When something is configurable in a lot of different ways, it can
sometimes be challenging to determine just *how* it got configured.
To aid in figuring out how zprint got configured in a particular way,
you can use the special call:

```clojure
(zprint nil :explain)
```

which will output the entire current configuration, and indicate
exactly where each value came from.  If there is no information
about a configuration value, it is the default.  For all values
that have been changed from the default value, the `:explain` output
will show the current value and indicate how this value was set.
Calls to set-options! are numbered.  For example, if the call is
made:

```clojure
(require '[zprint.core :as zp])

(zp/set-options! {:map {:indent 0}})

(zp/czprint nil :explain)

...
:map {:comma? true,
      :force-nl? nil,
      :hang-diff 1,
      :hang-expand 1000.0,
      :hang? true,
      :indent {:set-by "set-options! call 3", :value 0},
      :key-order nil,
      :sort-in-code? nil,
      :sort? true},
...
```

You can see from this fragment of the output that the indent for
a map has been changed to `0` by a call to `set-options!`.

This will distinguish values set by the `.zprintrc` from values set
by environment variables from values set by Java properties, so you
can more easily determine where a particular value came from if you
wish.

At any time, the `(zprint nil :explain)` or `(czprint nil :explain)`
will show you the entire current configuration of zprint, allowing
you to see all of the default values or any changes that have been
made to them.  Anything you can see with the `:explain` option can
be changed by set-options! or by any of the other configuration
approaches.

## License

Copyright © 2016 Kim Kinnear

Distributed under the MIT License.  See the file LICENSE for details.

