# AppleTV Video Playback #

The quality of video playback is dependent on numerous factors. Quality of your video content, output selection, X11 setup and display device all contribute to good or bad video playback. Make the wrong choices and you can get terrible video playback quality. This section is not a recipe for obtaining good quality video playback but rather a guide to help understand the dependencies.

Video Content
> Obviously, poor video content is going to give poor video playback. There's no amount of filters or display tricks that will fix poor content. Compressed formats (mpeg2, mpeg4 and H.264) tend to be more sensitive to poor content as one can control the amount of compression the is applied during encoding. This is reflected in a quantity know as bit-rate. Generally larger bit-rates are less compressed than lower bit-rates.

> I capture from two sources. The first is for SD content and I use a PVR-150 via s-video to capture the video output from a Dish Network 311 receiver. I also use the IR blaster from the PVR-150 to change channels.  I have two identical setups for a "dual tuner" capture device under MythTV. First thing to note is Dish Network uses mpeg2 as a compression method for the 311 receiver. So even though I'm capturing from an analog s-video output, the SD content will have varying amounts of compression artifacts due to the nature of the original video source. Second, the PVR-150 converts the analog video to digital mpeg2 in hardware. So to get the best quality, the PVR-150 need to be setup to capture/convert at a high bit-rate. I use 9Mb/sec. Maybe a little excessive but that yields about 2.2GB per 1 hour recording. I can live with that since I have about 3TB of storage for captured video.

> The second source is a HDHomeRun connected to an external antenna for off-the-air ATSC HD content. This gives me two independent tuners for recording local channels in 480p, 720p and 1080i. Since it is capturing ATSC HD content, this means mpeg2 and I have no control over the compression as the local station/network controls this function. Generally any 720p or 1080i ATSC HD content will be good quality. The only factor that might cause problems is the reception. I'm about 50 miles away from the local stations so I use a big directional antenna to obtain good reception.

Video Output
> The AppleTV has both component (analog) and HDMI (digital) output. Both outputs are capable of good video quality. For most people the video output choice is more dependend on their video display device. I use several display monitors depending on where my AppleTV is located as it moves around over time.

> a) 4:3 ratio 1024 x 768 LCD via a HDMI -> DVI adapter. This is ok but suffers from a beat pattern between the displayed content and the LCD retrace frequency. So decoded content looks good but every few seconds, fast moving objects get a wavy edge to them. This is not a de-interlace problem but a artifact of running the content display at one frequency and the actual display at another. Lesson here is always have the video output device define the retrace frequency rather than pick some odd ball retrace frequency. Since most video content is 24, 30 or 60 frames per second, you really want something like 60 (really 59.99) frames per second.  This way you get nice up and down frequency conversions without funny beat patterns. I could fix this issue for this display monitor but I really don't care. This monitor is only used for atv-bootloader development as it does not have an issue with overscan masking the command-line output.

> b) 4:3 ratio NTSC TV with 480i component inputs. This is a standard NTSC TV. This is in my "office", so when I'm doing AppleTV development, I use this as the video output monitor. I can even test HD content as this has the same decode cpu footprint as a real HDTV because the scaling from HD content (1080i or 720P) to NTSC (720 x 480) is done using the hardware scaler on the nvidia chipset. You want the display device (nvida chipset) to define the display output characteristics. The nvidia binary module supports HD480i, HD480p, HD720p, HD1080i and HD1080p as "TVStandard". For more selections, see the nvidia binary readme. Here's an xorg section for SD content over component. Change the "TVStandard" to a HD format for a HDTV using component.
```
Section "Monitor"
    Identifier     "Generic Monitor"
    #Option        "DPMS"
EndSection

Section "Device"
    Identifier     "Generic Video Card"
    Driver         "nvidia"
EndSection

Section "Screen"
    Identifier     "Default Screen"
    Device         "Generic Video Card"
    Monitor        "Generic Monitor"
    DefaultDepth    24
    Option         "DPI" "100x100"
    Option         "UseEvents" "1"
    Option         "AddARGBVisuals" "1"
    Option         "AddARGBGLXVisuals" "1"
    Option         "UseDisplayDevice" "TV"
    Option         "TVOutFormat" "COMPONENT"
    Option         "TVStandard" "HD480i"
    Option         "TVOverScan" "0.80
    Option         "NoLogo" "True"
    Option         "Coolbits" "1"
    SubSection     "Display"
        Depth       24
        Modes      "1920x1080" "1280x720" "1024x768" "720x480" "800x600" "640x480"
    EndSubSection
EndSection

Section "Extensions"
    Option         "Composite" "Disable"
EndSection
```


> c) 16:9 ratio Westinghouse TX-42F430S HDTV display that has HDMI, component, s-video and VGA inputs in addition to an ATSC tuner. This is primary "watch tv" or "watch movies" display device. It's very nice. It has a native 1920 x 1080 pixel LCD panel so true 1080p is possible. One thing that I have found is that the apparent difference between 1080i/p and 720p is very hard to see. Yes, 1080p is a higher resolution than 720p but the question is "can you really tell the difference". To me, this becomes a content driven answer. With the proper content, yes.

> But there's another issue to remember. The AppleTV has limited cpu resources. Viewing 1080i HD content (interlaced) on a non-interlaced display requires the use of de-interlacing or the edges of fast moving objects will look feathered (jagged). So if you have a 720p native display, and have xorg setup to drive 720p, you need to uses some type of de-interlacing on the AppleTV side. If you configure xorg.conf to drive a 1080i signal, you get free de-interlacing by the HDTV itself and it looks great. My Westinghouse does an interesting thing. When xorg is configured as 1080p, 1080i video content is also de-interlaced. Perfect. Here's an xorg section for HD content over HDMI. Since HDMI supports DPMS, X11 will "ask" the HDTV what resolutions it supports and default to the highest or 1080p for me.
```
Section "Monitor"
    Identifier     "Generic Monitor"
    Option         "DPMS"
EndSection

Section "Device"
    Identifier     "Generic Video Card"
    Driver         "nvidia"
EndSection

Section "Screen"
    Identifier     "Default Screen"
    Device         "Generic Video Card"
    Monitor        "Generic Monitor"
    DefaultDepth    24
    Option         "DPI" "100x100"
    Option         "UseEvents" "1"
    Option         "AddARGBVisuals" "1"
    Option         "AddARGBGLXVisuals" "1"
    Option         "UseDisplayDevice" "DFP"
    Option         "NoLogo" "True"
    Option         "Coolbits" "1"
    SubSection     "Display"
        Depth       24
        Modes      "1920x1080" "1280x720" "1024x768" "720x480" "800x600" "640x480"
    EndSubSection
EndSection

Section "Extensions"
    Option         "Composite" "Disable"
EndSection
```


> For composite output, Connect a cable from the green (luma) component output on the AppleTV to the yellow composite input on the TV. xorg.conf need the following settings for "Section Screen"

> For s-video output you need to make or buy an s-video to dual RCA cable (http://www.l-com.com/productfamily.aspx?id=596) See http://en.wikipedia.org/wiki/S-video for pin outs. Connect the s-video (luma - pin3, gnd pin 1) to the green (luma/Y) component output and s-video (chroma - pin 4, gnd pin 2) to the red (Cr/Pr) component outputs on the AppleTV. xorg.conf need the following settings for "Section Screen"


> For PAL output, change TVStandard to one of the nvidia PAL settings and I assume that the resolution size would change to a PAL size.

# Other xorg.conf setups #
Help out others by posting your " device" and "screen" sections in the comments if you figure out settings for other displays. Make sure you include the model and manufacture of your display for reference.