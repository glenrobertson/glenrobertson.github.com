---
layout: page
title: How to host a static site for free using Github Pages and Jekyll
---

My first post is about how I set up my blog: it is hosted for free on Github and consists entirely of static files.

![Yo dawg, I heard you like blogs, so I blogged about how to set up a blog so you can blog while you blog](/images/yo-dawg-i-heard-you-like-blogs.jpg)

If you:
* Have basic HTML, CSS and Git knowledge
* Prefer writing blog posts in markdown in a text editor over a web-based blog editor
* Only need to host static content such as HTML, CSS, images, etc
* Want free backups of your website (using Github)

... then you should host your website using Github Pages. The rest of the post will tell you how to set it up.

### Set up github repository
Sign up or log in to github.com, and create a repository with the name being: your username + ".github.com". For example, my username is `glenrobertson` so I created a repository called `glenrobertson.github.com` ([here](https://github.com/glenrobertson/glenrobertson.github.com)).

### Test your site
Add an `index.html` file to the root of your repository and push it (replace USERNAME in the following code with your username).

    git clone https://github.com/USERNAME/USERNAME.github.com
    cd USERNAME.github.com
    echo 'Hello world' > index.html
    git add index.html
    git commit -m "First commit"
    git remote set-url origin git@github.com:USERNAME/USERNAME.github.com
    git push origin master

At this point, you should be able to see your site at: http://USERNAME.github.com

### Add basic site layout
Github Pages uses a static site generator called [Jekyll](http://jekyllrb.com/). Using Jekyll's features, layouts can be used to avoid repeating HTML, and write blog posts in Markdown (which Jekyll converts to HTML). When the codebase contains a Jekyll configuration and is pushed to Github, Github will run Jekyll over the codebase and generate a directory of static files to host.

Create the following directory structure in your repository:

    config.yml
    _layouts/
        default.html
        page.html
    _posts/
        2013-01-13-first-post.html
    css/
        screen.css
    index.html

`_layouts/` contains files with HTML code that you want to reuse across your pages and blog posts.
`_posts/` contains files of your blog posts in Markdown.
`css/` is a directory to serve static files.
`index.html` is the root file of your site.
`config.yml` is the Jekyll configuration.

#### Configuration
Add the following content to `_config.yml`:

    markdown: rdiscount
    pygments: true

It tells Jekyll to use rdiscount as the Markdown converted, and to use pygments for syntax highlighting.

#### Layouts
The `_layouts` directory is a Jekyll convention. Any files in here contain HTML that you want to reuse across your pages.
Add the following content to `_layouts/default.html`:

    {% raw %}
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    <html xmlns="http://www.w3.org/1999/xhtml">
    <head>
       <meta http-equiv="content-type" content="text/html; charset=utf-8" />
       <title>{{ page.title }}</title>
       <link rel="stylesheet" href="/css/screen.css" type="text/css" media="screen, projection" />
    </head>
    <body>
        <div id="wrapper">
            <div id="header">
                <h1><a href="/">Home page</a></h1>
            </div>
            {{ content }}
        </div>
    </body>
    </html>
    {% endraw %}

The placeholder {% raw %}(`{{ content }}`){% endraw %} will inject the content of a page that is using the layout.


Add the following content to `_layouts/page.html`:

    {% raw %}
    ---
    layout: default
    ---

    <h2>{{ page.title }}</h2>
    <div id="content">
    {{ content }}
    </div>
    {% endraw %}

The `page` layout extends the `default.html` layout, with an added h2 element for the title of the page.

#### Posts
The `_posts` directory contains blog posts written in Markdown. They are named by date + the title of the blog. The date can be read by Jekyll, and can be used when listing the blog posts.
Create a markdown file called `year-month-day-post-title.md` (e.g. `2013-01-13-first-post.md`). The name of the file makes up the URL path to the post. Add the following content to the file:

    ---
    layout: page
    title: My first post
    ---

    Here is my first post.

The configuration has the `page` layout is being used, and the title "My first post", which the `page` layout injects as {% raw %} `{{ page.title }}` {% endraw} %}.

#### Static pages
Static pages are just html files, css files, images, etc. 
Add the following to `index.html`:

    {% raw %}
    ---
    layout: default
    title: Home
    ---

    <h2>Posts</h2>
    <ul class="posts">
        {% for post in site.posts %}
        <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
    </ul>
    {% endraw %}

The page uses the `default` layout. The is some layout syntax to list each blog post by date, and a link to them.


#### Test site with the jekyll --server
You should now have a basic site set up with a single blog post. You can test the site by using the Jekyll server, before pushing the code up to Github.

First, install jekyll and rdiscount:

    gem install rdiscount jekyll

Now, run the jekyll server:

    jekyll --server --auto

This will start the jekyll server in auto mode, which will restart the server when any files are modified. It should be running on port 4000. Go to [http://localhost:4000](http://localhost:4000) in your browser.


### Push it to Github
Once you have finished, run `git push origin master` to push your site to Github. You should then be able to view your site at http://USERNAME.github.com.

If you run into any problems, have a look at [my repository](https://github.com/glenrobertson/glenrobertson.github.com).

Now that your site is live, you may want to point your personal domain at USERNAME.github.com. This can be done by adding a `CNAME` record to your domain's zone file. To learn how to do this, follow the guide here: [setting up a custom domain with Github Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-pages). 

Thanks to [Tom Preston-Werner](https://github.com/mojombo) for creating Jekyll, and his basic repository layout ([here](https://github.com/mojombo/mojombo.github.com)).


