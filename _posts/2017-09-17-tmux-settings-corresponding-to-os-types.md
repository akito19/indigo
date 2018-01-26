---
title: "Tmux Settings Corresponding to OS Types"
layout: post
date: 2017-09-17 22:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- tmux
- macOS
- Linux
category: blog
author: akito19
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

Have you ever wanted to write settings corresponding to OS types in tmux?

I use macOS and Linux(Ubuntu); the former is for my work and later is for a regular life. So I had a problem that cannot be shared with settings about clipboard in each OS.

However, the problem can be solved because `if` is introduced into tmux 2.4.

This will help us to write settings in each OS. It is true that we have not used `else` and `elseif` yet,
but [a pull request](https://github.com/tmux/tmux/issues/1071) has been submitted to enable these operators.

### Tmux `if` Syntax

In tmux, we can use `if` as `%if`. A sample has been public in tmux repository.
Everyone can see [the example](https://github.com/tmux/tmux/blob/master/example_tmux.conf#L11-L14).

`%if` supports `==` and `!=` operator.

```shell
%if #{==,#{env_variable},expectation}
set foo
%endif
%if #{!=,#{env_variable},expectation}
set bar
%endif
```

### My configuring `.tmux.conf`

I'm using Tmux v2.5.

```
% tmux -V
tmux 2.5
```

Tmux enables `%if` but the syntax is limited only environment variable now. It means that we must use variables which are defined as the variables.

I wrote settings on `.tmux.conf` like below:

```shell
%if #{==:#{OSTYPE},darwin16.0}
set-option -g status-right ‘SSID: #(get_ssid) | Battery: #(battery -c tmux) | [%Y-%m-%d %H:%M] | #H:[#P]’
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe “reattach-to-user-namespace pbcopy”
bind-key -T copy-mode-vi y send-keys -X copy-pipe “reattach-to-user-namespace pbcopy”
set-option -g default-command “reattach-to-user-namespace -l $SHELL”
%end

%if #{==:#{OSTYPE},linux-gnu}
set-option -g status-right ‘[%Y-%m-%d %H:%M] | #H:[#P]’
bind-key -T copy-mode-vi Enter send -X copy-pipe-and-cancel “xsel -ip && xsel -op | xsel -ib”
bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel “xsel -ip && xsel -op | xsel -ib”
%endif
```

And `.zshrc`:

```
% export OSTYPE=$OSTYPE
```

The reason why I wrote above is that `OSTYPE` has not been registered as environment variables.
Furthermore, I could not use `uname` command becuase `%if` has supported for only environment variables.
Also, I did not use `*` when I declared OS type in `%if` evaluation like `darwin*` and `linux*` because of not being supported `*`.
Thus I set `OSTYPE` as an environment variable in loading `.zshrc` in order to distinguish Linux from macOS.
