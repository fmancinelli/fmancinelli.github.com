---
layout: post
title: Blogging on GitHub (GitBloggin')
---

Before starting this blog I had another attempt at blogging. What I was looking for was a system that allowed me to maintain my blog without using a web site.

Basically I wanted to keep my blog locally on my hard disk, with all the posts available in plain/text and organized using some directory structure. I was looking for a system that allowed me to write posts by using `vi` or `emacs` or whatever plain/text editor I was comfortable with, without having to use any fancy WYSIWYG editors on the web (yes, I am a pretty old-school person)

I ended up coding it by myself. The result was [FSBlogR](https://bitbucket.org/bitloom/fsblogr/wiki/Home). A small CGI script written in Ruby that was able to serve the content of a directory structure (that represented a category hierarchy) as if it was a blog with all its posts.

*FSBlogR* implemented exactly what I was looking for: my posts were stored in plain/text on my local disk, I was able to edit them using my favorite editor, and publishing a new post was just a matter of doing an `rsync` to a remote host.

However *FSBlogR* was far from being perfect. It used the *ERB* templating system but posts were still written using a mix of plain/text and HTML. I only implemented a very primitive filtering system that simply put HTML paragraph tags around plain/text paragraphs. A real markup language (e.g., [Markdown](http://daringfireball.net/projects/markdown/syntax), [Textile](http://textile.thresholdstate.com), etc.) was needed in order to keep posts content as simple as possible. 

Unfortunately the lack of time and ethusiasm decreed the death of FSBlogR and my previous blog.

At the beginning of this year I started this blog and I decided to go back using an online service. I chose [Posterous](http://www.posterous.com) because of its post-by-email feature. I told myself that even if I couldn't have my posts as regular files on my hard disk, it was nevertheless nice to have them available in the "sent" folder of my email client. 

However I started soon to feel frustrated with *Posterous*... It was quite slow and its theming system simply sucked. Basically you have to use a single file where you specify the CSS and all the possible ways of displaying information on your blog (e.g., front page, single post, static pages, etc.): imagine a programming language where you can't define functions or modules and you have to do everything in your `main`.

So I started to look around and I stumbled upon [GitHub Pages](http://pages.github.com): a way of publishing web sites by using a *Git* repository on GitHub. Basically the `master` branch of a repository called `username.github.com` becomes the content of the site located at `http://username.github.com`. Actually the `master` branch is pre-processed by [Jekyll](https://github.com/mojombo/jekyll), a static site generator that takes a set of templates and the raw content of the site written with some markup languages (i.e., Markdown or Textile), and generates the complete HTML version that is ready to be published.

Finally this is exactly what I was trying to accomplish with *FSBlogR* in the first place!

Now I have migrated my blog from Posterous to GitHub and...

* I have all my posts available on my local disk,

* I can edit them using my favorite text editors,

* I can use different markup languages to write them... Basically I can work with actual text/plain content that can be easily searched and copy/pasted almost as it is,

* the templating system allows me to build modular and easily maintainable structure for the web site,

* publishing a post is just a matter of git-pushing to the `master` branch whenever I am done with writing a post.

That's simply amazing.

If you want to have a look at how this blog looks like in its raw form you can have a peek [here](https://github.com/fmancinelli/fmancinelli.github.com). You can find this very post [here](https://github.com/fmancinelli/fmancinelli.github.com/blob/master/_posts/2011-02-27-Blogging_on_GitHub.markdown).

A very big list of sites and blogs using *GitHub Pages* is also available [here](https://github.com/mojombo/jekyll/wiki/Sites).

