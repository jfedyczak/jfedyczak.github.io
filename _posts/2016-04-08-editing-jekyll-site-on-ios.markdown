---
layout: post
title: "Editing Jekyll site on iOS"
date: 2016-04-08
tags: ios, jekyll, github
permalink: /post/editing-jekyll-site-on-ios/
uid: 1dc8e451-6859-4cae-9bb0-742109b2883b
---
I like using Panic's Coda for writing code on iOS but as it doesn't support git natively and I wanted to be able to edit my Jekyll-based site hosted on Github Pages on my phone, I came up with following workaround.

Used components:

 - [GitHub Pages](https://pages.github.com/) account
 - VPS with SSH access (I used $5 per month [droplet on DigitalOcean](https://m.do.co/c/b0e45ee1b3ec))
 - [Coda](https://panic.com/coda-ios/) for accessing and editing files via SFTP on iOS
 - [Jekyll](https://jekyllrb.com/) as content generator. It is natively supported by GitHub pages.

## GitHub Pages

To use GitHub pages you need Github account. To create personal page, you have to create repository named

    githubusername.github.io

When you commit anything into this repository, GitHub will automatically publish it as a site at `http://githubusername.github.io`. It is also possible to [publish site at different domain](https://help.github.com/articles/using-a-custom-domain-with-github-pages/) you own. In short:

 1. Add file named `CNAME` containing only `your.domain.com` to the repository.
 2. Add `CNAME` record pointing to `githubusername.github.io` in your domain's DNS.

## VPS

I chose Ubuntu on the droplet. As DigitalOcean allows adding SSH key during installation, first thing I did is disabling password login in `/etc/ssh/sshd_server`:

    PasswordAuthentication no

Then packages update:

    apt-get update

Required packet installation:

    apt-get install git jekyll jekyll-from-redirect ruby ruby-dev make gcc nodejs

Create a user:

    useradd -d /home/youruser -s /bin/bash -m youruser

Login and generate keys:

    su - youruser
    ssh-keygen

Accept all the defaults. After generating the key, display its public part with:

    cat .ssh/id_rsa.pub

And add it as an SSH key in your [GitHub keys settings](https://github.com/settings/keys). From now on you will be able to push to your repositories from `youruser` username on the VPS.

To allow Coda access to the account, generate new SSH key in the app on your iOS device, copy the public key to `.ssh/authorized_keys` in `yourusers`'s home folder.

You should now be able to connect to the account from Coda.

## Jekyll setup

SSH to your `youruser` account and setup git:

    git config --global user.name "Your Name"
    git config --global user.email your@email

Clone the repository:

    git clone git@github.com:githubusername/githubusername.github.io

Initialize new Jekyll project as per [documentation](https://jekyllrb.com/docs/quickstart/):

    cd githubusername.github.io
    jekyll new .

Make some changes. You can then run `jekyll serve` to see them on `http://your.vps.ip:4000/`.

To publish changes, commit and push them to GitHub:

    git add -A
    git commit -m "Initial commit"
    git push origin master

Changes will be automatically visible at `githubusername.github.io`.

This post was written entirely on an iPhone using Coda and a VPS.