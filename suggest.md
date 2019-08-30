---
layout: page
title: Suggestions
subtitle: additions / revisions for this website
---

First, create an account on [GitHub](https://github.com/) and [create a fork](https://help.github.com/en/articles/fork-a-repo) of my [website repository](https://github.com/sburden/website/).

Then, create a branch where you will make your changes.

~~~
# create a new branch called "branchname"
git branch branchname

# "checkout" this branch to edit it
git checkout branchname
~~~

To create a "blog post" that will appear on the [home page]({{ site.url }}):

~~~
# create the post -- refer to other posts for format
vim _posts/blog/YYYY-MM-DD-new-post.md

# edit the post
...                     

# add the post to the git repository
git add _posts/blog/YYYY-MM-DD-new-post.md 
~~~

To add a citation that will appear on the [papers page]({{ site.url }}/papers):

~~~
# add the BibTeX citation to the appropriate .bib file
# * follow the format of existing entries 
# * include DOI when available
vim conferences.bib
vim journals.bib
vim working.bib

# add the changes to the git repository
git add -u

# add the corresponding .pdf's to the _papers directory
cp ~/LastnameYYYYconf.pdf _papers/
git add _papers/LastnameYYYYconf.pdf
~~~

To add a bio that will appear on the [people page]({{ site.url }}/people):

~~~
# add the bio text in the appropriate section (PhD students, etc)
# * follow the format of existing entries 
# * include links to websites etc when available
# * include enough text to wrap around the photo
vim people/index.md

# add the changes to the git repository
git add -u

# add the bio photo to the people directory
cp ~/firstname_lastname.jpg people/
git add people/firstname_lastname.jpg
~~~

Finally, commit your changes, push them to GitHub, and [create a pull request](https://help.github.com/en/articles/creating-a-pull-request).  Then ping me to merge your edits.

~~~
# commit changes to your branch
git commit -m "explanation of edits"

# push edited branch to GitHub
git push origin branchname
~~~

