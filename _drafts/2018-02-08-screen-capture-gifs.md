---
layout:     post
title:      Screen capture gifs
date:       2018-02-09
summary:    
categories: [how-to]
---

On macOS

1. Get the Giphy Capture app
1. Use it to record your screen. You can adjust some quality settings here to keep the size down. Capturing a smaller portion of your screen will also lead to a smaller file size.
1. Use the `gifsicle` command line tool to optimise the gif:

   ```
   $ gifsicle -U original.gif -O2 -o compressed.gif
   ```
   
   It can also drop frames for greater ensmallment (in this example, we drop every second frame):
   
   ```
   $ gifsicle --delay $DELAY -U original.gif `seq -f "#%g" 0 2 $NUM_FRAMES` -O2 -o smaller.gif
   ```
   
   Without the `--delay` option the gif will appear go faster because there are fewer frames. You can get the values for `DELAY` and `NUM_FRAMES` by running
   
   ```
   $ gifsicle --info original.gif
   ```

   (you'll have to figure out how much the new delay will be based on the original delay and how many frames you're dropping).
   
   According to the man page you can use the `--colors` flag to reduce the number of distinct colours in the gif to make it even smaller, but I haven't tried this.
