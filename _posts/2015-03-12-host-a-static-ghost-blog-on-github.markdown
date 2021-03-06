---
title: Host a static Ghost blog on GitHub using Buster
date: '2015-03-12 19:30:00'
tags: [ghost, buster]
---

I have really enjoyed the simplicity of the [Ghost](https://github.com/tryghost/Ghost" target="_blank) blogging system and wanted to use it for my personal site.  But I also enjoyed the ease of hosting with [GitHub Pages](https://pages.github.com" target="_blank) so I needed to find a way to generate a static version of a Ghost blog.  Luckily someone already had a handy little tool for the job called [Buster](https://github.com/axitkhurana/buster" target="_blank).


### Installing Buster

Details on Buster can be found [here](https://github.com/axitkhurana/buster" target="_blank). Basically, you need to install **wget** and **git** with [brew](http://brew.sh/" target="_blank) (if they are not already installed on your system) then install Buster with [pip](https://pip.pypa.io" target="_blank).  Just run the following commands:

``` bash
brew install wget
brew install git
pip install buster
```

To test the install of Buster run the following:

``` bash
buster --version
```
You should see current version display (i.e. ```0.1.3```).


### Setting up Ghost

First off you need to download of a copy of the latest version of Ghost found [here](https://github.com/TryGhost/Ghost/releases" target="_blank). Extract the archive and open a console and go to the newly created directory and run the following:

``` bash
npm install --production
npm start
```

Then visit http://127.0.0.1:2368/ghost in a browser and go through the setup screens to create an admin account and start using ghost.


### Setting up Buster

In a new console window go to the directory where ghost is installed and type the following to setup buster:

``` bash
buster setup
```

Just follow the prompts and provide the data needed. Please view the README on the [Buster GitHub page](https://github.com/axitkhurana/buster" target="_blank) for more details on how to use Buster.


### Generating static pages with Buster

At the time of writing this Buster had some issues with missing the generation of some needed files and creating some files with incorrect URL reference in page. I created this script as a workaround for those issue.  So create a file (I called it **build-static.sh**) and be sure and change out the references to http://joshgerdes.com and replace them with your remote site URL.

``` bash
#!/bin/bash

# Generate static files with buster
buster generate --domain=http://127.0.0.1:2368  

# Copy sitemap files
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://127.0.0.1:2368/sitemap.xsl
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://127.0.0.1:2368/sitemap.xml
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://localhost:2368/sitemap-pages.xml
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://localhost:2368/sitemap-posts.xml
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://localhost:2368/sitemap-authors.xml
wget --convert-links --page-requisites --no-parent --directory-prefix static --no-host-directories --restrict-file-name=unix http://localhost:2368/sitemap-tags.xml

# Replace urls that were missed by buster
find static/* -name robots.txt -type f -exec sed -i '' 's#http://localhost:2368#http://joshgerdes.com#g' {} \;
find static/* -name *.xsl -type f -exec sed -i '' 's#http://localhost:2368#http://joshgerdes.com#g' {} \;
find static/* -name *.xml -type f -exec sed -i '' 's#loc>http://localhost:2368#loc>http://joshgerdes.com#g' {} \;
find static/* -name *.html -type f -exec sed -i '' 's#u=http://localhost:2368#u=http://joshgerdes.com#g' {} \;  
find static/* -name *.html -type f -exec sed -i '' 's#url=http://localhost:2368#url=http://joshgerdes.com#g' {} \;  
find static/* -name *.html -type f -exec sed -i '' 's#href="http://localhost:2368#href="http://joshgerdes.com#g' {} \;  
find static/* -name *.html -type f -exec sed -i '' 's#src="http://localhost:2368#src="http://joshgerdes.com#g' {} \;  
find static/* -name *.html -type f -exec sed -i '' 's#link>http://localhost:2368#link>http://joshgerdes.com#g' {} \; 

# Add CNAME file for github pages
buster add-domain joshgerdes.com

# Copy files that were missed by buster
cp humans.txt static/humans.txt
cp -R content/images static/content
```

Now give the newly created file execute permissions 

``` bash
chmod +x ./build-static.sh
```

Then just run in the directory where you installed ghost:

``` bash
./build-static.sh
```

This should generate static files under a subdirectory called ```/static``` which should be connected to your GitHub repository.

***NOTE**: I also forked the Buster repository and made changes to the project to fix the broken URLs and add the sitemap files to the static output.  So if you want to use my version of Buster it can be found [here](https://github.com/joshgerdes/buster" target="_blank). Hopefully my pull request will be accepted shortly and it will become part of the main script.*


### Previewing static pages

You can preview the static pages by running the following command:

``` bash
buster preview
```

Then visit http://localhost:9000 in a browser to view the static pages locally.


### Deploying static pages to GitHub

If you like what you see, you can push those changes to your GitHub repository using normal commands or type the following command:

``` bash
buster deploy
```

With luck you should have a nice static version of your site published to GitHub to enjoy.