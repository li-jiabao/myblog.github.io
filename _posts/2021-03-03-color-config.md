---
title: 颜色配置
author: lijiabao
date: 2021-02-18 16:36:15.907131000 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
#### less颜色配置


```
LESS_TERMCAP_xx
# 下面是xx的一些特殊符号的表示
mb
Start blinking
md
Start bold mode
me
End all mode like so, us, mb, md and mr
so
Start standout mode
se
End standout mode
us
Start underlining
ue
End underlining

export LESS=-R
export LESS\_TERMCAP\_mb=$'\\E\[1;31m'     # begin blink
export LESS\_TERMCAP\_md=$'\\E\[1;36m'     # begin bold
export LESS\_TERMCAP\_me=$'\\E\[0m'        # reset bold/blink
export LESS\_TERMCAP\_so=$'\\E\[01;44;33m' # begin reverse video
export LESS\_TERMCAP\_se=$'\\E\[0m'        # reset reverse video
export LESS\_TERMCAP\_us=$'\\E\[1;32m'     # begin underline
export LESS\_TERMCAP\_ue=$'\\E\[0m'        # reset underline
# and so on

man() {
    env \
        LESS_TERMCAP_mb=$(printf "\e[1;31m") \
        LESS_TERMCAP_md=$(printf "\e[1;31m") \
        LESS_TERMCAP_me=$(printf "\e[0m") \
        LESS_TERMCAP_se=$(printf "\e[0m") \
        LESS_TERMCAP_so=$(printf "\e[1;44;33m") \
        LESS_TERMCAP_ue=$(printf "\e[0m") \
        LESS_TERMCAP_us=$(printf "\e[1;32m") \
            man "$@"
}

# 遵循的格式
\e[(one or more numbers separated by semicolons)m

# 颜色的表示
0   Reset to standard configuration
1    Bold
31  Set foreground color to red
32  Set foreground color to green
33  Set foreground color to yellow
44  Set background color to blue

# ansi标准
30–37   Foreground color, dark
90–97   Foreground color, bright
40–47  Background color, dark
100–107  Background color, bright
38;5;###  Set foreground color to color ### (256-color extension)
48;5;###   Set background color to color ### (256-color extension)
```