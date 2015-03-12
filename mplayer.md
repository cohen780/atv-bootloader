# MPlayer on the AppleTV #

MPlayer is the de-facto standard for video content playback on Linux. It support numerous rendering and decode options and as such can be used for testing various SD and HD content playback on the AppleTV. I test using the standard installed MPlayer under Ubuntu, sometimes I pull from svn and build from source as H.264 decode/render is a moving target right now.

MPlayer has no problems with SD content. That's 480i/p mpeg2 or 480i/p mpeg4/H.264 content. It can play back using GL, Xv or XvMC (mpeg2 only).

Mplayer has no problems with mpeg2 HD content using Xv or XvMC. 1080i using Xv can put the cpu usage very close to max so I suggest using XvMC for 1080i mpeg2 HD content. GL rendering is not possible.

MPlayer has no problems with 720p mpeg4/H.264 HD content using Xv. Some very high bitrate content will require use of the "-lavdopts fast:skiploopfilter=all" option. GL rendering is lower bitrate content is possible but it's best to stick with Xv.

MPlayer cannot play 1080p mpeg4/H.264 HD content even with "-lavdopts fast:skiploopfilter=all" option. As 1080p mpeg4/H.264 HD content is extremely cpu intensive, this is no surprise.

MPlayer, with [this overlay patch](http://urandom.ca/mebox/downloads.php), supports an externally rendered overlay (vf\_overlay) with minimal cpu overhead. This means that one could develop a simple application that handles navigation, EPG and OSD while using MPlayer to decode and render SD and HD content. Development versions of [SMPlayer](http://smplayer.sourceforge.net/) and [Freevo](http://freevo.sourceforge.net/) seem to be using or thinking about using this feature.