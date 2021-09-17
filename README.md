This repo is a fork of CruelKernel, a customised Android/Linux kernel based on Samsung kernel sources for S10 and Note10 devices.


[CruelKernel](https://github.com/CruelKernel/samsung-exynos9820)  
[Samsung Open Source](https://opensource.samsung.com/main)  
[Linux Kernel](https://www.kernel.org/)

[CruelKernel README.md](https://github.com/lilboatclub/samsung-exynos9820/blob/cruel-DTI8-v3.8/CruelKernel%20README.md)  
[Linux Kernel README](https://github.com/lilboatclub/samsung-exynos9820/blob/cruel-DTI8-v3.8/README_linux)

# Modifying the Displayport Driver

To add custom resolutions we can hard code them into the Displaydriver. You can find the drivers in ```drivers/video/fbdev/exynos/dpu20/```

We need to edit two files, those are ```displayport.h``` and ```cal_9820/displayport_reg.c```  
For this example I will be adding the resolution 3440x1440p24.
### displayport_reg.c
```
...
	{V3840X1080P60,	V4L2_DV_BT_CEA_3840X1080P60_ADDED,	60, SYNC_NEGATIVE, SYNC_POSITIVE, 0, RATIO_ETC, "V3840X1080P60"},
	{V3840X1200P60,	V4L2_DV_BT_CEA_3840X1200P60_ADDED,	60, SYNC_NEGATIVE,	SYNC_POSITIVE, 0, RATIO_ETC, "V3840X1200P60"},
	{V3440X1440P60,	V4L2_DV_BT_CVT_3440X1440P60_ADDED, 60, SYNC_NEGATIVE, SYNC_POSITIVE, 0, RATIO_21_9, "V3440X1440P60", DEX_WQHD_SUPPORT},

--->	{V3440X1440P24, V4L2_DV_BT_CVT_3440X1440P24_ADDED, 24, SYNC_NEGATIVE, SYNC_POSITIVE, 0, RATIO_21_9, "V3440X1440P24", DEX_WQHD_SUPPORT},
/*	{V3440X1440P100, V4L2_DV_BT_CVT_3440X1440P100_ADDED, 100, SYNC_NEGATIVE, SYNC_POSITIVE, 0, RATIO_21_9, "V3440X1440P100"}, */
	{V3840X2160P24,	V4L2_DV_BT_CEA_3840X2160P24,	24, SYNC_POSITIVE, SYNC_POSITIVE, 93, RATIO_16_9, "V3840X2160P24"},
	{V3840X2160P25,	V4L2_DV_BT_CEA_3840X2160P25,	25, SYNC_POSITIVE, SYNC_POSITIVE, 94, RATIO_16_9, "V3840X2160P25"},
...
```
### displayport.h

```
typedef enum {
        V640X480P60,
        V720X480P60,
        V720X576P50,
        V1280X800P60RB,
        V1280X720P50,
        V1280X720P60EXT,
...
        V3440X1440P60,
--->    V3440X1440P24,
/*      V3440X1440P100,*/
...
        V4096X2160P60,
        V640X10P60SACRC,
        VDUMMYTIMING,
} videoformat;
```

and also

```
#define V4L2_DV_BT_CVT_3840X2160P24_ADDED { \
        .type = V4L2_DV_BT_656_1120, \
        V4L2_INIT_BT_TIMINGS(3440, 1440, 0, V4L2_DV_HSYNC_POS_POL, \
                533250000, 48, 32, 80, 3, 10, 6, 0, 0, 0, \
                V4L2_DV_BT_STD_DMT | V4L2_DV_BT_STD_CVT, \
                V4L2_DV_FL_REDUCED_BLANKING) \
}

/* # This bit doesn't get added, it's just to show what the numbers above mean
pixelclock 	= 533250000
H Front Porch 	= 48
H Sync 		= 32
H Back Porch 	= 80
V Front Porch 	= 3
V Sync 		= 10
V Back Porch 	= 6 */
```

Timings were calculated using [https://tomverbeure.github.io/video_timings_calculator](https://tomverbeure.github.io/video_timings_calculator)

That's all that's needed to add custom timings. See [cruel kernel](https://github.com/CruelKernel/samsung-exynos9820) for info on how to build the kernel

To use your new timings, ``cat`` the sysfs file ``/sys/class/dp_sec/forced_resolution`` to get the number for your resolution, then ``echo`` that number to ``forced_resolution``

e.g.

```
cat /sys/class/dp_sec/forced_resolution

... (list of resolutions here)
28 : V2560X1440P60
...

echo 28 > /sys/class/dp_sec/forced_resolution
```

If on stock samsung rom you may need to ``echo 0`` to ``/sys/class/dp_sec/dex`` first. Phone may crash if monitor doesn't support the added resolution/doesn't communicate that it does. Adapter needs to have enough bandwidth also. check ``dmesg Displayport:`` for debugging info or ``cat /proc/dplog``
