---
title: Going Static
date: 2015-02-08
description: By now you may have noticed the new décor of my site - the even more observant of you may have also noticed that my blog has migrated Wordpressto Jekyll.
draft: false
type: post
---

By now you may have noticed the new décor of my site - the even more observant of you may have also noticed that my blog has migrated from [Wordpress](https://wordpress.org/) to [Jekyll](http://jekyllrb.com/).

Besides the fact that all the cool kids have been moving their blogs away from Wordpress, there were three main reasons for the switch:

- The plethora of [Wordpress](http://www.cvedetails.com/vulnerability-list/vendor_id-2337/product_id-4096/Wordpress-Wordpress.html) and PHP related [security](http://blog.sucuri.net/2014/05/vulnerability-found-in-the-all-in-one-seo-pack-wordpress-plugin.html) [vulnerabilities](http://blog.sucuri.net/2015/01/critical-ghost-vulnerability-released.html) that are floating around these days.
- Speed. Nothing beats serving static HTML.
- I don't have to worry about a Ubuntu update breaking anything in my LEMP/LAMP stack - which makes patching security updates a nightmare.

Happy days.

## Game of Themes

I've adapted the great theme [Hyde](http://hyde.getpoole.com/) made by [Mark Otto](https://twitter.com/mdo) and added a few things like [SASS](http://sass-lang.com/) via [jekyll-assets](https://github.com/ixti/jekyll-assets) and changed a few other things - some small like the fonts, others slightly bigger like removing the sidebar. I'm also proud to say that my site does *NOT* use jQuery. Yes, no jQuery!

Feel free to grab the theme [on Github](https://github.com/aidenhaak/aidenhaak.com) and try it yourself.

## Deploying Jekyll

Deployment is easily done via the use of [rsync](http://en.wikipedia.org/wiki/Rsync) and adding Rakefile that looks like the following code to your blog's directory.

{{< highlight ruby >}}

#rake generate
task :generate do
puts '* updating jekyll site'
system 'jekyll b'
end

# rake rsync
desc 'Uses rsync to push the contents of ./_site to the server.'
task :rsync do
puts '* rsyncing the contents of ./_site to the server'
system 'rsync -acvz -e "ssh -p $PORT_NUMBER" _site/ user@example.com:/dir/to/website/root/www/example.com/'
end

# rake deploy
desc 'Deploy the content to the server.'
task :deploy => [:generate, :rsync] do
end

{{< / highlight >}}

Then it's just a matter of typing *rake deploy* in your blog's directory to push your new contents to your web server. That's one less excuse I now have for my dreadfully lacklustre rate of posting. 
