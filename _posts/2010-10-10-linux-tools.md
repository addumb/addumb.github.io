---
layout: post
title: Linux Tools
date: 2010-10-10 14:48:33.000000000 -07:00
type: post
published: true
status: publish
categories:
- Tech
tags: []
author:
  login: addumb
  email: adam@addumb.com
  display_name: addumb
  first_name: ''
  last_name: ''
---
[http://github.com/addumb/tools](http://github.com/addumb/tools)

I've gotten tired of re-writing small command-line tools that are missing from most Linux distributions. Too often I've found myself trying to pipe through a command that calculates the average or standard deviation of a text file. I re-write the same awk lines over and over to get a distribution of certain columns of log files (like nginx HTTP status codes or of the document size sent).

I know these tools have been written over and over by many many people. The ones I've found have been either subject specific (e.g. generate a sideways ASCII histogram from a log file, which is really super sweet and useful, but not always what I'm looking for) or too all-in-one to satisfy my need for a more tool-chainy way to do things. This is 90% for myself, so I don't have to re-write these over and over. I put them up on github so I can just clone the tools wherever I want to use them. It'd be great if other people found these tools useful, and maybe even wrote some more :)

DISCLAIMER: So far these are basically wrappers around awk. That's mostly because awk is the awesomest tool in the whole world. Also, it's because I've been dealing with a lot of text processing lately at work. I have dreams of many other tools, but none concrete enough to start writing them.
Cross-post from [http://github.com/addumb/tools/blob/master/README.md](http://github.com/addumb/tools/blob/master/README.md):

## avg
Spit out the average of a column in a text file

## stddev
Spit out the standard deviation of a column in a text file

## column-bin
bin the columns of a text file (VERY useful for quick summaries of apache HTTP status codes and the like)
Usage:

    $ cat test.csv
    log HTTP status: 200 body_size: 1024
    log HTTP status: 200 body_size: 1024
    log HTTP status: 500 body_size: 0
    log HTTP status: 200 body_size: 10000
    log HTTP status: 400 body_size: 10
    log HTTP status: 400 body_size: 10
    log HTTP status: 400 body_size: 10
    log HTTP status: 409 body_size: 10
    $ column-bin -c 4 > test.csv
    409	1
    400	3
    200	3
    500	1

## column-histogram
Similar to column-bin, but for continuous variables. Bin the column into bins of a fixed size OR bin them into log-based sizes to make either a histogram or a semi-log histogram.
Usage:

    $ cat test.csv
    log HTTP status: 200 body_size: 1024
    log HTTP status: 200 body_size: 1024
    log HTTP status: 500 body_size: 0
    log HTTP status: 200 body_size: 10000
    log HTTP status: 400 body_size: 10
    log HTTP status: 400 body_size: 10
    log HTTP status: 400 body_size: 10
    log HTTP status: 409 body_size: 10
    $ column-histogram -c 6 -b 1000 > test.csv
    10000	1
    0	5
    1000	2
    $ column-histogram -c 6 -L 10 > test.csv
    10000	1
    0	1
    10	4
    1000	2
