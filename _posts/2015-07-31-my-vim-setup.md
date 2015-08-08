---
layout:     post
title:      My VIM Setup
date:       2015-08-07
summary:    Using VIM for software development
categories: blog vim environment 
---

As a full stack software developer, architect, database administrator, and systems administrator I wear many hats at the office. I often find myself going back and forth between many different servers and environments. One minute I'm troubleshooting a production issue in Postgres, the next I'm wiring up an NPM and webpack environment for a new project. And when I'm finally left alone I will be in my wheelhouse writing code in one of several languages like Go, PHP, JavaScript, HTML, etc. To maximize effeciency I've dialed in what I consider an effecient ecosystem of VIM, tmux and iTerm2 tools. Today I'd like to share some of my favorite configurations and plugins. 

<div style="text-align: center">
<a href="/images/MyWorkDesktop-1080.png"><img src="/images/MyWorkDesktop-720.png"  width="720" height="298"></a>
</div>

## Buffer and File Mangement
This is the most important part of my setup. My workflow is based around tmux for screen and server management, vim-airline for displaying open files and open buffers as "tabs", buffergator for viewing open buffers not in view, and git-gutter for showing changed lines. As you can see I typically keep an open window to my left for git commands and database console. When doing front end I keep my npm or webpack watcher active.

## Code Editing
For actual code editing I rely on YouCompleteme with exuberant ctags for autocomplete, autopairs for closing braces, UltiSnips for adding snippets like a Symfony2 class or action, and SuperTab to allow me to use the tab key for autocomplete or snippets depending which has precedence. 

## My .vimrc

Here is a gist of my .vimrc file. Leave a comment if you have a specific question and I'll try my best to answer. The documenation of my vim plugins is pretty decent and I recommend trying things out first to see if it fits your worflow. 

<a href="https://gist.github.com/tmpjr/11381706">Link To Gist</a>

{% gist tmpjr/11381706 %}

