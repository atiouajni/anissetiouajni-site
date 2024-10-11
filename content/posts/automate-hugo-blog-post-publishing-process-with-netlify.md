---
title: "Automate Hugo Blog Post Publishing Process With Netlify"
date: 2022-02-25T18:03:38+01:00
type: 'post'
categories: ['blog']
---
When I wanted to create my personal blog, I was faced with a very wide choice of technology. Between traditional CMS like Wordpress and Drupal and static site generators like Jekyll and Hugo, there were too many choices.
So I thought about several selection criteria that will help me choose the right technology:

- a simplistic and minimalist content presentation
- Use of my usual tools as much as possible
- Avoid too much development/customization
- Automate the publication of new content
- the cost

Once it was clearer in my head, the search was much more relevant. My number 1 criterion was to find a simple theme that corresponds well to my idea. **[Hugo](https://gohugo.io)** appeared very quickly in my first choices.
In addition, Hugo is on the rise and more and more blogs are using it to develop their web content in a very minimalist way. Which is reassuring!
From now, all we have to do is take a look at the technology behind this pretty name Hugo.

Hugo is a static website generator. It supports [Markdown](https://www.markdownguide.org/) markup language ( language I often use to document my projects on Github )
The site lists a wide choice of themes to use and customize if necessary. Nevertheless a theme I really liked, that of [Gokarna](https://github.com/526avijitgupta/gokarna).
A few very minimalist modifications and here I am with an interesting base for my project.

I wanted this blog to be automated as much as possible so I could just focus on the content - writing the posts.
A couple stood out very quickly, **Hugo + Netlify**. [Netlify](https://www.netlify.com/) can host the Hugo site with CDN, has continuous deployment features and a very intuitive admin GUI. It fetches content from Github and automatically deploys Hugo-generated pages.

By default, the website is always accessible through a Netlify subdomain based on the site name. But we have the possibility through the "custom domains" feature to point a non-netlify domain to our newly generated site.

A visit to [OVHcloud](https://www.ovhcloud.com/) to buy the domain name anissetiouajni.com, a few clicks to modify the DNS to use those of netlify and here we have an online website, deployed automatically.

A picture paints a thousand words :

![Post publishing process](/img/2022-02-25/post-publishing-process.png)

Now let's get to the practical part. I will describe the entire procedure from installing Hugo on my pc to putting my site online and creating my first post!

## Install Hugo

As I have a Macbook, I will install Hugo with Homebrew. If you are on other OS, you can follow the Hugo [installation](https://gohugo.io/getting-started/installing/) page.


```shell
brew install hugo
hugo -version
```

## Create a new site

```shell
hugo new site anissetiouajni-site
```

Running this command will generate a whole directory structure. More details about this directory structure can be found on [Hugo's website](https://gohugo.io/getting-started/directory-structure/#directory-structure-explained).

```shell
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

## Install a new theme

```shell
git clone https://github.com/526avijitgupta/gokarna themes/gokarna
```
Once the theme was downloaded, I followed the basic configuration described on the [Gokarna](https://gokarna-hugo.netlify.app/posts/theme-documentation-basics/) site.

## Test locally
Let's start the Hugo server and see the magic happen.

```shell
hugo server -D
```

`-D` option mean that we want to build content even if it was marked as draft

We can add some draft content by simply typing this command :

```shell
hugo new posts/my-first-post.md
```

Once our post is written, read and re-read, we can remove the `draft` property and push the whole structure into our GitHub repository. If this property remains at `true`, the content will not be visible to the public after publication.

## Deploy static site to Netlify

The first step is to push our code into GitHub. Since I want the site to be generated in its entirety during deployment, I must therefore specify to the `git` command to ignore the `public` directory.

```shell
echo public > .gitignore
echo .hugo_build.lock >> .gitignore
git init
git add *
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/atiouajni/anissetiouajni-site.git
git push -u origin main
````

Now that we have our structure in GitHub, we can move on to the Netlify part.
To configure continuous deployment, there are 2 ways to do it on Netlify. Either through the GUI or through a configuration file. I would use this last option in order to simplify and minimize my interventions.

```shell
vi netlify.toml
````

Edit the file with this content :

```toml
[build]
  command = "hugo --gc --minify"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.92.2" // same version of Hugo I use locally
```

Then just follow the very good [step-by-step](https://www.netlify.com/blog/2016/09/29/a-step-by-step-guide-deploying-on-netlify/) tutorial provided by Netlify. 

We are able to generate our website automatically and continuously. But to access it, we must go through the subdomains of Netlify. By default, any site on Netlify is accessible from its Netlify subdomain, which has the form `[name-of-your-site].netlify.app`. In my case, it is [heuristic-lalande-14552d.netlify.app](https://heuristic-lalande-14552d.netlify.app).

## Point custom domain name to Netlify

As I said above, Netlify allows us to specify custom domains. I will set up this feature to redirect my domain name [anissetiouajni.com](anissetiouajni.com) to the Netlify servers (which hosts the content of my website)

To assign a custom domain to a site, navigate to the Netlify GUI (you must be logged) `Site settings > Domain management > Custom domains`.
Select `Add domain alias` at the bottom of the Custom domains panel, and enter the custom domain name. Once added, you will have a warning message like this :

![Check DNS configuration Netlify](/img/2022-02-25/domains-https-check-dns-configuration.png)

Now we are pretty close and all that’s left to do is configuring the DNS. Just click on the Check DNS configuration and we will get the Netlify nameservers, that we would need to update in our domain DNS settings.

My domain name is provided by OVHCloud. I connect to the admin GUI, and follow [instructions](https://docs.ovh.com/gb/en/domains/web_hosting_general_information_about_dns_servers/) to remove default DNS servers and add those of Netlify ( `dns*.p04.nsone.net` ).

Depending on the DNS provider, changes to DNS records can take several hours to propagate and take effect for the entire internet. On my side I had to wait more than 24 hours and renew the TLS certificate because it used the old domain name. 

In the netlify admin console, on `Site settings > Domain management > HTTPS`, a simple click on `Renew certificate` generates a new certificate.

## Summary

Here I am at the end of setting up my website. I'm quite happy to find a simple and effective way to share my knowledge without dwelling on the form and availability of content. The steps are quite simple and the sites provide very detailed guidelines.
I finished pushing this article in my GitHub and here it is immediately available for everyone to read!











