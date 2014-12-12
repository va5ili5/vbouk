---
layout: post
title:  "Tags in Jekyll"
description: "A way to add tags to your posts without plugins"
date:   2014-12-12
author: Boukonis Vasileios
tags:
- jekyll
- tags
- howto
categories: jekyll
---

If you use Jekyll for first time or you have recently migrated to Jekyll from another blogging system you probably have already noticed that the standard Jekyll installation doesn't provide any build in mechanism to generate tag pages that contains list of posts with specified tag. You can solve this problem by writing your own custom plugin but if you want to host your blog on GitHub pages then, for security reasons, GitHub will disable it. A workaround as referenced in <a href="http://jekyllrb.com/docs/plugins/" target="_blank">Jekyll's documentation</a> is to build your website locally and push the generated files included in `_site` folder to your repository. Searching the internet I found that there are other workarounds to the problem without using plugins. Im my blog I wanted for each tag to create a separate page to display all the post matcing the selected tag. To do this we need to create a new folder called tags in the site's root directory. 

{% highlight ruby %}
root
	tags	
{% endhighlight %}

Inside this folder we add new folders for the tags. So for instance if we want to have a `jekyll` and `howto` tag we add two new folders inside the `tags` folder with the names `jekyll` and `howto` like so

{% highlight ruby %}
root
	tags
		jekyll
		howto	
{% endhighlight %}

Now that we have our folders set, we add a page in each folder. This page will be used to display the posts. For instance if we want to display all the posts tagged by `jekyll`we will use this page to display them. We name this page `index.html` and in the YAML front-matter we declare two variables. The layout and the tag. The layout variable contains the name of the layout to be used and the tag variable contains the tag value for this spacific post. For the `jekyll` tag the page will be:

{% highlight ruby %}
---
layout: tagtemplate
tag: jekyll
---
{% endhighlight %}

and for the `howto` tag:
{% highlight ruby %}
---
layout: tagtemplate
tag: howto
---
{% endhighlight %}

So now we have the required files the folders ready we can continue to implement the layout. In `_layouts` folder we create a new layout page called e.g. `tagtemplate.html`. For my blog this is the layout:

{% highlight html %}
{% raw %}
---
layout: default
---
<div class="page-content">
	<h1 class="page-heading">Posts tagged by "{{ page.tag }}"</h1>
	<ul class="post-list">
	{% for post in site.tags[page.tag] %}
		<li>
	    	<h2>
	      		<a class="post-link" href="{{ post.url | prepend: site.baseurl }}">  {{ post.title }}</a>
	        </h2>
	        <span class="page-post-meta">{{ post.date | date: "%B %d, %Y" }}</span>
		</li>
	{% endfor %}
	</ul>
</div>
{% endraw %}
{% endhighlight %}

First we display the name of the tag in `<h1></h1>` tags. The for loop iterates all the posts with that tags and prints a list of posts. Each post in this list contains the post title and the meta-data. We make the post title a link so we can easily go back to the original post.

Now we are ready to create a post using tags. In the YAML front-matter we add a tags variable and we asign values. For example a post tagged by `jekyll` and `howto` the front-matter will be:
{% highlight html %}
{% raw %}
tags:
- jekyll
- tags
{% endraw %}
{% endhighlight %}

The last think we need to do is to include the tags (as links) in our `index.htm` file and in the posts layout. For this we need another for loop to loop through the post tags and display the tag name.

{% highlight html %}
{% raw %}
/* index.html */
{% for tags in post.tags %}
    <a class="tags" href="{{base.url}}/tags/{{tags}}/">{{tags}}</a>
  {% endfor %}
{% endraw %}
{% endhighlight %}

{% highlight html %}
{% raw %}
/* post.html */
{% for tags in page.tags %}
    <a class="tags" href="{{site.baseurl}}/tags/{{tags}}">{{tags}}</a>
  {% endfor %}
  {% endraw %}
{% endhighlight %}

That's it now we have for each tag a page to display a list with all the posts tagged by that tag. You can check my <a href="//github.com/va5ili5/vbouk" target="_blank">GitHub repository</a>.