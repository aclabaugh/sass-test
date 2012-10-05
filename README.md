# bundler + guard Test for SASS and LiveReload

I've never been a fan of doing development on my local machine. For whatever reason it seems like you should be doing dev in the same environment you will deploy to. Because of that my typical development process happens on the server with the real text editor, VIM. Recently, I've been experimenting with tools like [SASS](http://sass-lang.com/) and [LiveReload](http://livereload.com/).

LiveReload has a built-in SASS compiler that will automatically compile SASS and reload the browser on file change. The result is an experience like making changes in the web inspector. Move an element over 5px, it moves over 5 pixels. Change the color, it changes. No switching applications, no clearing the cache, no reloading. Web development for the twenty-first century.

These tools are nice and neat, but working in a team requires a workflow that works for everyone. And consequently the majority of our team dev process follows my initial process. Development, post inital CSS and HTML work, always happens on a dev server. Local tools like the LiveReload app don't fit this process. However, there are ruby tools that can replicate this functionality server-side.

SASS itself is packaged as a ruby gem and I've describe the [process of getting it installed](http://thoughts.andyclabaugh.com/first-experiments-with-sass) previously. In my initial experimentation, I was just looking to get it compiling files. This time I was looking to set it up in a way that is least intrusive to our normal dev process. Manually processing a file each time you make a change can add up to a lot of lost time. Ideally we were looking for a tool that would monitor all SASS directories and compile whenever a change is detected in a file. While there doesn't appear to be a good tool to do this server-wide, there are a number of good tools to accomplish this per-site.

First is the simple folder watch for SASS. Also of note, our typical convention of adding a underscore prefix to global css files clashes with SASS convention. Files that start with an underscore are not processed by SASS unless called explicitly.

    sass --watch sass:css

In search of some more comprehensive solutions, I ran across bundler and guard. Doug Avery over at Viget has a nice [write up](http://viget.com/extend/case-study-ruby-tools-for-non-ruby-projects) of how they use these two ruby gems in their dev process, even for projects outside ruby. Bundler allows you to maintain a set of ruby gems required for a project. Guard is a ruby gem that monitors files and reacts on changes.

To get started, we need to pull down the required gems.

    gem install bundler
    gem install guard

Once we have these two installed we can setup the project bundle.

    bundle init

This will create a Gemfile in the project directory. Edit that file and include the needed gems. On my system SASS keeps complaining about rb-inotify, so I include it as well.

    source 'http://rubygems.org'

    gem 'guard'
    gem 'guard-sass'
    gem 'guard-livereload'
    gem 'rb-inotify', '~&gt; 0.8.8'

Next, we need to enabled guard for the project.

    guard init

This will create a Guardfile for the project. This is a ruby file where we can define actions to occur on file change.

    guard 'sass', :input =&gt; 'sass', :output =&gt; 'css'
    
    guard :livereload do
      watch(%r{.+\.(css|js)$})
    end

That first line is the short hand convention for a guard statement. The second is the full syntax. There are a [number](https://github.com/hawx/guard-sass) [of](https://github.com/guard/guard-livereload) [resources](https://gist.github.com/1610551) that provide examples of different guard definitions.

Once your Gemfile and Guardfile are defined you can install the bundle and initialize the guard.

    bundle install
    bundle exec guard

Since we are running this on a server, I will typically add a '-n f' flag to the exec statement to stop it from trying to use X11 notifications.

The exec statement will start monitoring the project, so you are free to edit the project files in another command prompt and the SASS will be auto compiled on save. In order to hook into LiveReload, you must install a [browser extension](http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions-). From the options page of the extension you can set the server name and port (35729 by default). Load your page and click the LiveReload button. If it works, you will see the following in your guard command prompt.

    &gt; LiveReload 1.6 is waiting for a browser to connect.
    &gt; Browser connected.

And there you go. LiveReload, with SASS compiling, from the dev server to your browser. Pretty awesome.

