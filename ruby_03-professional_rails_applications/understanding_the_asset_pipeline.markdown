---
title: Understanding the Asset Pipeline
length: 180
tags: ruby, rails, asset_pipeline
---

## Learning Goals

* Understand where Rails looks for client-side assets in your application
* Add assets to manifests to create asset bundles
* View and modify the asset pipeline load path
* Learn when not to use asset pipeline manifests

## Warm Up

When building a Rails application, we tend to just assume that jQuery is readily available for us. If we open up the Chrome Developer Tools, we'll see that our Rails application is loading jQuery.

Where is it coming from?

## Lecture

Source files go in one end; if necessary, they get processed and compiled (think SASS or CoffeeScript); they get concatenated and compressed and spit out the other end as bundles.

The asset pipeline relies on a few technologies:

* **Sprockets** is a Rack-based asset packaging for compiling and serving web assets. It handles dependency management and preprocesses CoffeeScript, SASS, et cetera on your behalf.
* **Tilt** is a wrapper around a number of Ruby template engines, giving them a common interface. You can take a look at all of the formats Tilt can handle by [checking out the documentation][tilt].

[tilt]: https://github.com/rtomayko/tilt/blob/master/README.md

So, why should you use the asset pipeline? Inline JavaScript (mixed in your HTML/ERB code) blocks loading and rendering the page—which is not something we usually want to block. Plus it is messy to mix JavaScript, Ruby, and HTML in a view template. Let's keep JavaScript in its own files in the Rails assets directories.

Rails will pick up new files in your `app/assets` directory, but you have to reset the server if you add a new *directory* to the `app/assets`.

Rails pulls in assets from the following locations:

* `app/assets`
* `lib/assets`
* `vendor/assets`
* Gems with a `vendor/assets` directory.

`vendor/assets` is a good place for third-party JavaScript libraries that aren't yours (e.g. Underscore.js, D3). `lib/assets` is typically used for assets that are created by your team but used by multiple applications. `app/assets` are your application-specific assets.

By default, Rails places three sub-directories in your `app/assets` directory. These are completely arbitrary. You can name these directories whatever you want or add other directories to your heart's content.

Anything in the pipeline will be available at the `/assets` URL. So, the `app/assets/javascripts/application.js` in your asset pipeline will be available in development at `http://localhost:3000/assets/application.js`. `app/assets/stylesheets/application.css` will also be available at the root of your asset directory. The asset pipeline will completely flatten your directory structure when you spin up your development server or precompile your assets.

**Your Turn**—try this out for yourself:

* Clone [turingschool-examples/storedom](https://github.com/turingschool-examples/storedom) (too soon?) and do the necessary prep work (`bundle`, the requisite `rake` tasks).
* Create a directory in `app/assets` called `texts`.
* Add a text file—let's call it `hello.txt`—to `app/assets/texts` and give it some contents.
* Fire up the server and visit `http://localhost:3000/assets/hello.txt`.
* Move it to `app/assets/javascripts` and refresh the page.
* Create a file called `hello.js` in `app/assets/javascripts`.

At it's core, the asset pipeline is a list of load paths. You can see these load paths by firing up the Rails console.

```ruby
y Rails.application.config.assets.paths
```

(The `y` command just formats the hash as YAML.)

You'll typically see something like this:

```yaml
---
- "/Users/stevekinney/Projects/storedom/app/assets/images"
- "/Users/stevekinney/Projects/storedom/app/assets/javascripts"
- "/Users/stevekinney/Projects/storedom/app/assets/stylesheets"
- "/Users/stevekinney/Projects/storedom/vendor/assets/javascripts"
- "/Users/stevekinney/Projects/storedom/vendor/assets/stylesheets"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/less-rails-bootstrap-3.2.0/app/assets/fonts"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/less-rails-bootstrap-3.2.0/app/assets/javascripts"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/less-rails-bootstrap-3.2.0/app/assets/stylesheets"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/turbolinks-2.2.2/lib/assets/javascripts"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/jquery-rails-3.1.1/vendor/assets/javascripts"
- "/Users/stevekinney/.rvm/gems/ruby-2.1.3/gems/coffee-rails-4.0.1/lib/assets/javascripts"
```

The asset pipeline works its way through your load path. The first asset with a given name wins. If you had an asset named `app/assets/stylesheet.css.scss` and another called `vendor/assets/stylesheet.css.scss`, the asset in your `app/assets` directory would win because it occurs first in the load path.

#### Adding to the Search Path

By default, Rails will search the first set of directories directly under `app/assets`, `lib/assets`, `vendor/assets`. You can add additional paths to the asset pipeline, if your heart desires.

Let's say you're living in the future and you want to include some Adobe Flash. And you want to store your flashy Flash apps in `app/flash/assets`—in an effort from infecting your other assets with a case of the early 2000s. You can add that path to the asset pipeline in `config/initializers/assets.rb`.

```rb
Rails.application.config.assets.precompile << Rails.root.join("app", "flash", "assets")
```

**Your Turn**: Add an additional path somewhere in your application to the asset pipeline. Pop a file into your new directory and make sure it works. Rename it to `hello.txt` or `hello.js` and see what happens.

### Manifests

Manifests are a way to pull in other, related files. So, if I request `application.js` and it requires `another.js` in it's manifest, I will get both of them.

* `// require_tree .` requires all of the files in that directory and all subdirectories.
* `// require_directory .` loads all of the files in the directory but *not* the subdirectories.
* Alternatively, you can just take matters into you own hands and manually define the files you want to include.

In this example, we're looking at `application.js`; so, we're using JavaScript comments. If you're in `application.css` then it would be in CSS comments. 

By default, the asset pipeline concatenates all of assets into one file (using `require_tree .`). Browsers can only make a limited number of requests in parallel. This technique allows you to get all of your assets with one request.

Here's an example of an `application.js` manifest from Storedom:

```js
//= require jquery
//= require jquery_ujs
//= require twitter/bootstrap
//= require turbolinks
//= require_tree .
```

#### Manifest Directives

* `require` grabs an asset and puts it in our bundle once.
* `include` works a lot like `require`, but it will allow you to include a file more than once. (I have yet to find a practical use for this directive.)
* `require_self` tells Sprockets to load the body of the current file before loading any of the dependencies. You would use this if you wrote any styles or JavaScript in `application.css` or `application.js` respectively and you wanted Sprockets to load that code before loading any of the required assets.
* `require_directory` requires all of the source files of the same format in a given directory. It only goes one level deep.
* `require_tree` works like `require_directory`, but it also traverses subdirectories.
* `depend_on` announces that you depend on a file, but does not include it in the asset bundle.
* `stub` blacklists a dependency from the bundle.

#### A Quick Note on SASS/SCSS

Sure, you could require and concatenate multiple `.scss` files using manifests, but you probably shouldn't. The asset pipeline is fairly agnostic to the special features of the assets you're working with. SASS has an `@import` directive that works better for our purposes. In this case, it's better to just include `@import` directives in your `application.css.scss` than it is to use manifest directives.

You can also do something similar with JavaScript, but it's a little more intensive, so for now, we'll use manifests to concatenate our scripts.

#### MD5 Fingerprints

If you look at a Rails application, you'll notice that Rails has appended a string of letters and numbers onto your filename. The main idea here is make sure you browser isn't referencing an out-of-date cached version of the file. This MD5 fingerprint will change every time you modify an included file.

### ActionView Helpers

ActionView gives you a set of helper methods that you can use in your views to include assets.

```erb
<%= stylesheet_link_tag "application", :media => "all" %>
<%= javascript_include_tag "application" %>
```

In addition to including scripts and stylesheets, you can also use ActionView helpers to include media content.

```rb
audio_path("horse.wav")   # => /audios/horse.wav
audio_tag("sound")        # => <audio src="/audios/sound" />
font_path("font.ttf")     # => /fonts/font.ttf
image_path("edit.png")    # => "/images/edit.png"
image_tag("icon.png")     # => <img src="/images/icon.png" alt="Icon" />
video_path("hd.avi")      # => /videos/hd.avi
video_tag("trailer.ogg")  # => <video src="/videos/trailer.ogg" />
```

See [ActionView::Helpers::AssetTagHelper][taghelper] documentation for more information.

The `sass-rails` gem also has [a set of helpers][sasshelpers] that allow you to reference other assets without having to know their exact location. This should help you resist the temptation of using ERB in your `.scss` assets.

[taghelper]: http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html
[sasshelpers]: https://github.com/rails/sass-rails#asset-helpers

### Differences in Development and Production

As we mentioned in the beginning, the whole idea of the asset pipeline is to concatenate everything into one file, because performance. But, we'll notice that when we spin up our application in development, we'll see many files listed in the resources tab of the Chrome Developer Tools.

This is because this functionality is disabled in `config/environments/development.rb`:

```rb
# Debug mode disables concatenation and preprocessing of assets.
# This option may cause significant delays in view rendering with a large
# number of complex assets.
config.assets.debug = true
```

This is useful for debugging JavaScript and CSS issues. It's turned off in production.

#### Precompiling Assets

If you request an asset in development, Rails will check `public/assets` first. If your asset is not there, it will hit up the asset pipeline and compile it on the fly. This is useful in development because you're likely to be making frequent changes and edits to those files. But, it would also be a performance bottleneck in production if Rails had to compile those files on every request.

If you want to use an asset in your application, it has to either be required by a file in your asset pipeline or precompiled.

Let's say you had a stylesheet called `site.css.scss`. You could simply require it in `application.css`.

```css
// = require_self
// = require 'site'
```

Alternatively, you can add it to the precompile list similar to the way we added a load path earlier.

```rb
config.assets.precompile += %w( site.css )
```

When you run `rake assets:precompile`, Rails goes through your assets and copies everything over to `public/assets`. It then creates `application.js` and `application.css` by reading the manifests. It does not look at any other file unless you explicitly tell it to.

The confusing part here is that every file in your asset pipeline that is *not* a CSS or JavaScript file will be copied over to `public/assets`. JavaScripts and stylesheets must either be included in a manifest or explicitly added to the `config.assets.precompile` directive.

It's also important to note, that when using `config.assets.precompile`, that the file extension of the target file matters. `config.assets.precompile += %w( site )` will not work.

### Gemified Assets

As we've seen with `jquery-rails`, gems can be used to bundle client-side assets.

#### Engines

In order to bundle client-side assets for inclusion in the the asset pipeline, you have to define an engine that inherits from `Rails::Engine`. Let's take a look at how the `rails-jquery` does it by checking out `lib/jquery/rails/engine.rb`.

```rb
module Jquery
  module Rails
    class Engine < ::Rails::Engine
    end
  end
end
```

There's not a lot going on in this code, but it tells Rails, "Hey! Look at me! Put by `vendor/assets` into the asset pipeline!"

If you use an `index.js` or `index.css`, then you can require the whole gem without specifying a file.

Why would you want to use an `index.js`? Well, let's say you broke your gem assets into multiple files—probably a good idea. Using an `index.js`, allows you to be explicit about the order that these files should be included in.

### Post-Processing

In production, Rails will minify your assets to help you conserve bandwidth. Rails has some sensible defaults that you're welcome to override if you'd like.

* Stylesheets are compressed with the YUI Compressor. You can also use the standard SASS compressor by by setting `config.assets.css_compressor = :sass`.
* JavaScript is compressed using Uglifier, but you can set `config.assets.js_compressor` to `:closure-compiler`, `:uglifier` or `:yui-compressor`.
