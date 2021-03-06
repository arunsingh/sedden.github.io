---
layout: post
title:  "Publish to GitHub User Pages with Jekyll and Travis-CI"
date:   2015-11-08 23:38:31
categories: travis ci github access token push
---

According GitHub's Help on [User, Organization, and Project Pages][gh-pages], there
are few important differences on how those kind of pages are served over HTTP(S).

* **User and Organization Pages** will be published with content of the *master*-Branch, while the repository must match the schema *\<username\>.github.io*.
* **Project Pages** are published with content of the *gh-pages*-Branch of a certain project repository.

This short guide will show you how to automate publishing for **User and Organization Pages** with [Travis-CI][travis] and [Jekyll][jekyll].
The topic has been discussed on [stackoverflow][stackoverflow] [many](http://stackoverflow.com/a/33125422) times.
This is kind of a best-of using an HTTPS API Token (via environment variable) and keeping the history of the published *master*-branch.

#### It's only five steps

1. Create a repository *\<username\>.github.io* with an empty commit on master.
2. Create a *jekyll*-Branch based on the *master*-Branch and put your [Jekyll][jekyll]-site into it.
3. Connect [Travis-CI][travis] with GitHub and create an access token to allow `git push` for the build job.
4. Go to the Travis-CI Dashboard, activate the build job for your repository and add an *Environment Variable* with the name `GITHUB_ACCESS_TOKEN` and the newly generated token.
5. Last but not least, add the `.travis.yml` file below to your `jekyll` branch, adjust it, commit and push.

{% highlight yaml %}
sudo: false
language: ruby
branches:
  only:
    - jekyll
script:
  - "bundle exec jekyll build"
after_success: |
  if [ -n "${GITHUB_ACCESS_TOKEN}" ] && [ "${TRAVIS_PULL_REQUEST}" = 'false' ]
  then
    cd "${TRAVIS_BUILD_DIR}/_site"
    git init
    git remote add origin "https://sedden:${GITHUB_ACCESS_TOKEN}@github.com/sedden/sedden.github.io"
    git config user.name "Stefan Jenkner"
    git config user.email "stefan@jenkner.org"
    git fetch
    git reset origin/master
    git add .
    git commit -a -m "Travis CI build ${TRAVIS_BUILD_NUMBER}."
    git push origin HEAD:master >/dev/null 2>&1
    cd "${TRAVIS_BUILD_DIR}"
  fi
{% endhighlight %}

Ensure to redirect any output of `git push` to `/dev/null`! Otherwise it will leak your access token!

[gh-pages]: https://help.github.com/articles/user-organization-and-project-pages/
[travis]: https://www.travis-ci.org/
[jekyll]: http://jekyllrb.com/
[stackoverflow]: http://stackoverflow.com/

