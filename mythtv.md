![http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/mythtv-logo.jpg](http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/mythtv-logo.jpg)

# MythTV #

under construction



The new playback settings are great but really confusing to understand at first. To top it off, none of the default settings are suitable for the AppleTV. Go to the playback settings and change "CPU--" to the following settings. This sets XvMC for decode resolutions greater than 1280 x 720 (1080i mpeg2) and ffmeg for everything else. I prefer "linearblend" over "kerneldeint". "kerneldeint" does "linearblend" for most frames and switches to something else for high motion. I can see the toggling between the two different algorithms and it annoys my viewing.


| **CPU --** | **Profile 1** | **Profile 2** |
|:-----------|:--------------|:--------------|
| Description | XvMC for higher resolutions | ffmpeg fallback |
| Resolution | >= 1280 720 | > 0 0 |
| Decoder | XvMC | ffmpeg |
| Video Renderer | xvmc-blit | xv-blit |
| OSD Renderer | ia44blend | chromakey |
| OSD Fade | Off | Off |
| Deinterlacer | one field | linearblend |
| Secondary Deinterlacer | none | one field |
| Filters | none | none |
