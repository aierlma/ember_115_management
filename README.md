# ember_115_management

This repo stored corresponding dockerfile and compose.yaml of this tutorial [【Nas】告别 alist，三步实现 emby 115 直链播放](https://hicane.com/archives/nas-gao-bie-alist-san-bu-shi-xian-emby-115-zhi-lian-bo-fang)

And the images on dockerhub are arm64 v8 architecture, if you want amd64, use the original [emby_nginx](https://hub.docker.com/r/hicane/emby_nginx) and [emby_115](https://hub.docker.com/r/hicane/emby_115), the dockerfiles should be the same, I just built it on arm machine and that's all.

Basically, you need to have a clouddrive2 first, and then use the compose.yaml

If you want to built your own image, note to give executive permission to sh files under emby_nginx, and change the base image from arm architecture to your targetd one
