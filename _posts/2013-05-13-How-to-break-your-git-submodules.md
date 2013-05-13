---
layout:     post
title:      "How to break your git submodules"
---

I have been using a large variety of plugins in vim, and a few months ago I started using git submodules and pathogen to manage them. With the help of a script to take care of some of the common operations, such as updating all of the plugins at once and committing them, things worked pretty smoothly.

Then one day I cloned the repository over to another computer and told git to update the submodules, where it then halted the procedure and asked me for my github username and password. This doesn't make any sense though, since I don't own the repository it was cloning from, and when I did enter the credentials it failed predictably saying it couldn't find the repository under my username.

The problem stems from my inexperience with git submodules, particularly with the way it handles changes to the submodule repositories. 

    [galactor][allgood38]$ tree -d -L 2
    .
    ├── autoload
    ├── bundle
    │   ├── command-t.git
    │   ├── gundo.vim.git
    │   ├── LaTeX-Box.git
    │   ├── nerdcommenter.git
    │   ├── nerdtree.git
    │   ├── ScrollColors.git
    │   ├── snipmate-snippets.git
    │   ├── supertab.git
    │   ├── syntastic.git
    │   ├── YouCompleteMe.git
    │   ...
    ├── colors
    ├── ftdetect
    ├── ftplugin
    ├── plugin
    ├── snippets
    ├── spell
    └── syntax

Above is the directory structure of my vimfiles repository. Each of the folders ending in `.git` are submodules. Submodules don't need to end with this, its just the way I chose to handle it with my management script. It also makes it clear that the folder is in fact a submodule. 

The problem was created when I ran vim, and the plugin in the submodule folder altered some of the files, and when I was playing around with the plugins I think I may have committed the changes into the submodule. This is fine if you're a dev working on a plugin, but I had now just created a commit, that would only be visible to me, I didn't upload it to the remote url, which is what git uses to clone the submodule. So now any time I try to clone the repository and update the submodules, it looks for a commit the developer never made, causing it to fail.

The first approach I took was removing the submodule completely and trying to re-add it, however given that I was trying to add it with the same submodule name, git realised there was still references to the old one in its index and refused to do anything.

Eventually I learned that the `.git` folder within each of the submodule root directories is actually just a text file with a line that references where the real `.git` folder is within the parent (top-level) repository. As a side note, I noticed all my references where absolute, so if you were to try moving the entire parent repository to another location in the filesystem, submodules also fail.

For example, this is the `.git` file (not folder) within one of the git submodule working directories:

    gitdir: /home/allgood38/.git/vimfiles/.git/modules/\
        vim/bundle/tabular.git

So I went into the parent's `.git` folder and went to the path `.git/modules/.../submodule.git` which contains something very similar to a bare git repository. Once in there I moved one directory up and did:

    rm -rf submodule.git/*
    git clone --bare submodule-url.git submodule.git

And now we have a bare repository with the correct HEAD pointer and everything. Now you go into the `submodule.git` again and open `config`. Remove the line that says `bare: true`, since submodules won't actually work as bare repositories. Finally, go to the submodule path in your working directory and type:

    git pull origin master

And the submodule will use the information from the bare repository we just clone to update all of the information. Now if you run `git submodule update`, you shouldn't have any problems.
