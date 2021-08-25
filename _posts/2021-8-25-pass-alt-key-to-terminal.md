---
layout: post
title: Pass ALT(meta) key to putty
---

## Putty 里正确设置 ALT(meta) 键

此处感谢此帖
[终端里正确设置 ALT 键和 BS 键](http://www.skywind.me/blog/archives/2021)
的作者，同时也在此作一下整理与备份。

### 背景

本人工作环境使用 windows+putty 通过 ssh 连接服务器，在配置 vim 的 Auto-pairs 插件的时候，无意中被 \<M-p\> 这类快捷键无效的问题难住。经过一些分析与对比，发现是 ALT(meta) 键无法正确发送给终端造成的，试过很多种方法并未能解决此问题。

### 调查

网上找了一些文件后得知，不管你在终端下使用 vim/neovim, emacs, nano 或者 zsh，你都会碰到使用 ALT 键的情况（终端下叫做 meta键），而由于历史原因，大部分终端软件的默认设置都无法正确使用 ALT 键。

要在终端下正确使用 ALT键最简单的做法是：首先将终端软件的 “使用 Alt键作为 Meta键” 的功能打开，意思是如果你在终端下按下 ALT+X，那么终端软件将会发送 \<ESC\>x 两个字节过去，字节码为：0x27, 0x78。

使用如下命令可以察看发送的字节码：

```shell
showkey -a
```

试了一下ALT+X，果然是 0x27, 0x78。

### 各终端解决

SecureCRT 和 Xshell 都有设置解决此类问题，如 Use ALT key as meta key，勾上就行。

其他终端软件里，可以按如下的方法来尝试

Putty/MinTTY 默认ALT+X 就是发送 \<ESC\>x 过去

Mac下面的 iTerm2/Terminal.app 需要跟 XShell / SecureCRT一样设置一下

Ubuntu 下面的 GnomeTerminal 默认也是发送 \<ESC\>x 过去的

任意平台下面的 xterm 可以配置 ~/.Xdefaults 来设置这个行为。

唯有 vim ，由于历史原因，需要在你的 vimrc 里加一段键盘码配置：

```shell
function! Terminal_MetaMode(mode)
    set ttimeout
    if $TMUX != ''
        set ttimeoutlen=30
    elseif &ttimeoutlen > 80 || &ttimeoutlen <= 0
        set ttimeoutlen=80
    endif
    if has('nvim') || has('gui_running')
        return
    endif
    function! s:metacode(mode, key)
        if a:mode == 0
            exec "set <M-".a:key.">=\e".a:key
        else
            exec "set <M-".a:key.">=\e]{0}".a:key."~"
        endif
    endfunc
    for i in range(10)
        call s:metacode(a:mode, nr2char(char2nr('0') + i))
    endfor
    for i in range(26)
        call s:metacode(a:mode, nr2char(char2nr('a') + i))
        call s:metacode(a:mode, nr2char(char2nr('A') + i))
    endfor
    if a:mode != 0
        for c in [',', '.', '/', ';', '[', ']', '{', '}']
            call s:metacode(a:mode, c)
        endfor
        for c in ['?', ':', '-', '_']
            call s:metacode(a:mode, c)
        endfor
    else
        for c in [',', '.', '/', ';', '{', '}']
            call s:metacode(a:mode, c)
        endfor
        for c in ['?', ':', '-', '_']
            call s:metacode(a:mode, c)
        endfor
    endif
endfunc

call Terminal_MetaMode(0)
```

到此，vim 中 ALT 组合键 \<M-X\> 或 \<A-X\> 就可以工作了。
