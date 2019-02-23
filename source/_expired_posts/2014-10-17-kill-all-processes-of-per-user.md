---
layout: post
title: 'Kill all processes of per user'
date: 2014-10-17 02:47
comments: true
categories: 
---

針對某位 user 砍它所有 processes


- ```pgrep -u pichuang | sudo xargs kill -9```

- ```pkill -u pichuang```

- ```ps aux | grep pichuang | awk '{ print $2 }' | sudo xargs kill -9```

通常是遇到 fork bomb 才會用到 XD