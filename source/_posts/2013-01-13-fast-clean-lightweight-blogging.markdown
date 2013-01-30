---
layout: post
title: "Fast, clean, lightweight blogging"
date: 2013-01-13 21:12
comments: true
categories: 
---

Now, I'm not exactly a shining example of actual blogging - I get distracted by real life way too easily.  However, on the technical side, I think I have a fairly good setup. Here is how you would duplicate it. I assume barebones linux knowledge and the ability to read carefully.

1.  Get a server. You can get a free account from [Amazon's AWS](http://aws.amazon.com) and even a free instance for a year (a "micro" in their parlance, which is more than sufficient for a small blogging platform). You could also get a virtual machine from [Fatboxes.com](http://fatboxes.com) - I hear they're pretty good. As it turns out, it doesn't really matter which distribution you choose - [CentOS](http://centos.org), [Debian](http://debian.org), [whatever](http://archlinux.org). If you're more comfortable with one than the other, have at.

2.  Install the software we'll be using to make things fast. [Varnish](http://varnish-cache.org) and [Lighttpd](http://lighttpd.cz). For any reasonable distro, that should be as simple as a quick:

        apt-get install varnish lighttpd

    Varnish is a very simple beast. The entirety of the config you need is:

        backend default {
            .host = "127.0.0.1";
            .port = "8080";
        }

    Put that in /etc/varnish/default.vcl and start varnish. You might also check /etc/default/varnishncsa, set `VARNISHNCSA_ENABLED` to 1, and start the varnishlog service.

    Lighttpd is almost as simple. From the default config file, /etc/lighttpd/lighttpd.conf, set server.port to 8080. Everything else should be correct out of the box. Once you change that, start the lighttpd service. At this point, you should be able to hit port 80 on your server with a web browser, and get a default page in response. Last step is to chown /var/www/ to your user:

        sudo chown user:group /var/www

3.  Fork your own copy of octopress on [github](https://github.com/imathis/octopress) and then clone a copy on your local computer somewhere:

        git clone https://github.com/[your username here]/octopress.git
        bundle install # This installs all the gems and whatnot you need
        rake install  # This creates all the config files, directories, etc., you need

    You now need to modify \_config.yml and Rakefile in the root of the new repo. They are well-documented with comments, and should be easy enough to figure out. Things you want to keep an eye out specifically are:

        # From Rakefile
        ssh_user       = "dbishop@hostname"
        document_root  = "/var/www"
        
        # From _config.yml
        url: http://gnuconsulting.com/
        title: geekery
        subtitle: of all stripes
        author: David Bishop
        simple_search: http://google.com/search
        description: General geekery, a dash of politics, and more.

4.  Add your first post by running:

        rake new_post 'Hey, I'm blogging!'

    That will create a file along the lines of `source/_posts/2013-01-13-hey-im-blogging.markdown`. Edit that in your favorite text editor ([vim](http://vim.org), natch), using [Markdown](http://daringfireball.net/projects/markdown/basics) to format your text. 

5.  Once your precious prose is how you want it, it's time to publish:

        rake gen_deploy

    And I'm sure that all went swimmingly with no errors. You're welcome!
