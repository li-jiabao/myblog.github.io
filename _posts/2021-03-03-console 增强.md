---
title: console 增强
author: lijiabao
date: 2021-02-16 21:49:08.904879100 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
#### tab 补全


#### [彩色输出命令设置](https://wiki.archlinux.org/index.php/Color_output_in_console)
```
alias diff='diff --color=auto'  # diff的
alias grep='grep --color=auto'
alias ip='ip -color=auto'
alias ls='ls --color=auto'

# pacman的彩色输出通过取消color行的注释即可

# less 设置好下面这些，可以让man彩色输出
export LESS=-R
export LESS\_TERMCAP\_mb=$'\\E\[1;31m'     # begin blink
export LESS\_TERMCAP\_md=$'\\E\[1;36m'     # begin bold
export LESS\_TERMCAP\_me=$'\\E\[0m'        # reset bold/blink
export LESS\_TERMCAP\_so=$'\\E\[01;44;33m' # begin reverse video
export LESS\_TERMCAP\_se=$'\\E\[0m'        # reset reverse video
export LESS\_TERMCAP\_us=$'\\E\[1;32m'     # begin underline
export LESS\_TERMCAP\_ue=$'\\E\[0m'        # reset underline
# and so on

# man
man() {
    LESS\_TERMCAP\_md=$'\\e\[01;31m' \\
    LESS\_TERMCAP\_me=$'\\e\[0m' \\
    LESS\_TERMCAP\_se=$'\\e\[0m' \\
    LESS\_TERMCAP\_so=$'\\e\[01;44;33m' \\
    LESS\_TERMCAP\_ue=$'\\e\[0m' \\
    LESS\_TERMCAP\_us=$'\\e\[01;32m' \\
    command man "$@"
}


## less的语法彩色输出
You can enable code syntax coloring in less. First, install source-highlight, then add these lines to your shell configuration file:

~/.bashrc
export LESSOPEN="| /usr/bin/source-highlight-esc.sh %s"
export LESS='-R '

```


#### [alias别名增加](https://wiki.archlinux.org/index.php/Bash#Aliases)
将常用的一些命令加参数可以绑定为别名，使用`alias ip=‘ip -color=auto`
下面是一些比较常用的alias绑定，别名绑定一般放在用户目录下的`~/.bashrc/~/.zshrc`
系统级别的alias绑定一般放在`/etc/bash.bashrc`
```
#\# Modified commands

alias diff='colordiff' # requires colordiff package

alias grep='grep --color=auto'

alias more='less'

alias df='df -h'

alias du='du -c -h'

alias mkdir='mkdir -p -v'

alias nano='nano -w'

alias ping='ping -c 5'

alias dmesg='dmesg -HL'

#\# New commands

alias da='date "+%A, %B %d, %Y \[%T\]"'

alias du1='du --max-depth=1'

alias hist='history | grep' # requires an argument

alias openports='ss --all --numeric --processes --ipv4 --ipv6'

alias pgg='ps -Af | grep' # requires an argument

alias ..='cd ..'

# Privileged access

if (( UID != 0 )); then

alias sudo='sudo '

alias scat='sudo cat'

alias svim='sudoedit'

alias root='sudo -i'

alias reboot='sudo systemctl reboot'

alias poweroff='sudo systemctl poweroff'

alias update='sudo pacman -Su'

alias netctl='sudo netctl'

fi

#\# ls

alias ls='ls -hF --color=auto'

alias lr='ls -R' # recursive ls

alias ll='ls -l'

alias la='ll -A'

alias lx='ll -BX' # sort by extension

alias lz='ll -rS' # sort by size

alias lt='ll -rt' # sort by date

alias lm='la | more'

#\# Safety features

alias cp='cp -i'

alias mv='mv -i'

alias rm='rm -I' # 'rm -i' prompts for every file

# safer alternative w/ timeout, not stored in history

alias rm=' timeout 3 rm -Iv --one-file-system'

alias ln='ln -i'

alias chown='chown --preserve-root'

alias chmod='chmod --preserve-root'

alias chgrp='chgrp --preserve-root'

alias cls=' echo -ne "\\033c"' # clear screen for real (it does not work in Terminology)

#\# Make Bash error tolerant

alias :q=' exit'

alias :Q=' exit'

alias :x=' exit'

alias cd..='cd ..'


# 有颜色的输出设置
alias ip='ip -color=auto'
alias grep='grep -color=auto'

```



#### bash function (设置函数来完成特定的命令操作也是可以的)
在上述的几个配置文件中增加函数方法，也能当成命令来使用
```
# 展示错误代码
EC() {
	echo -e '\\e\[1;33m'code $?'\\e\[m\\n'
}
trap EC ERR

# 用来提取众多类型压缩文件的通用解压命令
extract() {
    local c e i

    (($#)) || return

    for i; do
        c=''
        e=1

        if \[\[ ! -r $i \]\]; then
            echo "$0: file is unreadable: \\\`$i'" >&2
            continue
        fi

        case $i in
            \*.t@(gz|lz|xz|b@(2|z?(2))|a@(z|r?(.@(Z|bz?(2)|gz|lzma|xz)))))
                   c=(bsdtar xvf);;
            \*.7z)  c=(7z x);;
            \*.Z)   c=(uncompress);;
            \*.bz2) c=(bunzip2);;
            \*.exe) c=(cabextract);;
            \*.gz)  c=(gunzip);;
            \*.rar) c=(unrar x);;
            \*.xz)  c=(unxz);;
            \*.zip) c=(unzip);;
            \*.zst) c=(unzstd);;
            \*)     echo "$0: unrecognized file extension: \\\`$i'" >&2
                   continue;;
        esac

        command "${c\[@\]}" "$i"
        ((e = e || $?))
    done
    return "$e"
}

# 上述的类似方法定义操作可以根据自己的实际情况来设置，比如调节亮度也可以使用这个来进行定义执行
```


#### command not found命令找不到
`pkgfile`包含了一个命令找不到的钩子脚本，这个脚本可以用来搜寻标准库来寻找可能提供该命令的包，[pkgfile安装](https://wiki.archlinux.org/index.php/Pkgfile#Installation)
`pkgfile -U`同步包


##### bash的command not found设置
为了激活这个，需要在配置文件`~/.bashrc`中`source`这个hook，在配置文件中添加这一行即可
`source /usr/share/doc/pkgfile/command-not-found.bash`

##### zsh的设置
- 可以在`～/.zshrc`中添加一个`command_not_found_handler()`方法，用来指定命令找不到时的处理方法方法内容如下：
```
command\_not\_found\_handler() {
	local pkgs cmd="$1" files=()
	printf 'zsh: command not found: %s' "$cmd" # print command not found asap, then search for packages
	files=(${(f)"$(pacman -F --machinereadable -- "/usr/bin/${cmd}")"})
	if (( ${#files\[@\]} )); then
		printf '\\r%s may be found in the following packages:\\n' "$cmd"
		local res=() repo package version file
		for file in "$files\[@\]"; do
			res=("${(0)file}")
			repo="$res\[1\]"
			package="$res\[2\]"
			version="$res\[3\]"
			file="$res\[4\]"
			printf '  %s/%s %s: /%s\\n' "$repo" "$package" "$version" "$file"
		done
	else
		printf '\\n'
	fi
	return 127
}
```

- 或者利用pkgfile包提供的`/usr/share/doc/pkgfile/command-not-found.zsh`同bash的设置
