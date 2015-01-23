# ngix_http_enchance_mp4_module

- This module is an enhance version of original nginx mp4 module (which shipped into nginx source code). Please note this is my own project, it has nothing related with nginx's source code except it takes all source code of ``ngx_http_mp4_module``.

- This module enable nginx mp4 module to modify mp4 file (which ``moov`` atom place in the last of file) on the fly (just one time), then deliver it to video player. 

# This is why
- In order to deliver HTTP (RTMP and other can do better) video on demand (using ``ngx_http_mp4_module``) the right way: player just has to download:
    1. the ``ftyp`` atom and check that the container format, version and branding are supported.
    2. ``moov`` atom, check that the required codec are available and use the "stco" sub-atoms to start decoding the video and audio streams.

The whole data is ``mdat`` atom, which can be buffered, decompressed and play later. 

The ``ftyp`` and ``moov`` atom are just few KB, so if these atom are placed in the begining of the file, every player can start progressive download just after download them. 
Most of video encoder/muxer/demuxer (without doing 2 pass encoding) place moov atom in the last of the file, which cause progressive playback has problem. Browser has to download the whole file in order to start playing that file, it's not progressive playback anymore.

People can fix mp4 source file using various tools, likes:
    1. https://github.com/danielgtaylor/qtfaststart
    2. Using ffmpeg with 2 pass encoding or ``movflags=+faststart``  
    3. Using my modified version took from ffmpeg project: https://github.com/whatvn/qt-faststart 
    3. Using mp4box with -inter option


These tools above require you to do this at encoding step: convert the file, standardise it. 
Using this module, you *dont have to standarlise* using above tool anymore (you still have to convert file into Quick time format). 


# Usage:

- install 
    - ``./configure --add-module=ngx_http_enhance_mp4_module``

- like mp4 module, nginx configuration file will look likes this:

``bash
  location /mp4 {
                error_log logs/mp4_debug.log debug;
                root /data2/streams/ ;
                enchance_mp4;
                fix_mp4 on;
                enchance_mp4_buffer_size 1m;
                enchance_mp4_max_buffer_size 50m;
    }
``

- which:
    - ``enchance_mp4``, ``enchance_mp4_buffer_size``, ``enchance_mp4_max_buffer_size`` are the same with ``mp4``, ``mp4_buffer_size``, and ``mp4_max_buffer_size`` directives from the original module.
    - ``fix_mp4``: when it is on, mp4 file will be fixed if needed, if ``moov`` atom is place in right order, we dont have to fix it. if it is off, this module works exactly the way nginx mp4 module do (so why you use it).


# Note:
- To make it less conflict with nginx mp4 module, I've tried to change its namespace from ``ngx_mp4`` -> ``ngx_enhance_mp4``, I don't take license of this. 

    





