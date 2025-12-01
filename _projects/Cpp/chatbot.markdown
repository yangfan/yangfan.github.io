---
layout: page
title: Memory Management Chatbot
description: |
  The chatbot will talk with you about basic concepts in C++ memory management. You can ask related topic of memory management in C++, like unique pointer and get answer from chatbot. In this program, the smart pointers and move semantics are used to manage resources and optimize the code.
# img: /projects/Cpp/assets/img/chatbot/membot_demo.gif
importance: 3
category: Cpp
github: https://github.com/yangfan/Memory-Management-Chatbot
---

This project creates a simple ChatBot using C++. The ChatBot (or MemBot) will introduce some basic concepts about the memory management in C++. The ChatBot will provide you the information based on the topic you type in, e.g., pointers.

Text file [`answergraph.txt`](https://github.com/yangfan/Memory-Management-Chatbot/blob/master/src/answergraph.txt) defines a knowledge graph, where each node represents an answer and edge represents user query. The most related answer is selected based on the Levenshtein distance. In this project, smart pointers are used to manage the memory allocation.

<div class="row justify-content-center">
    <div class="col-10">
      <div class="w-50 mx-auto" style="background-color: white;">
          <img class="img-fluid z-depth-1" src="{{ '/assets/img/projects/Cpp/chatbot/membot_demo.gif' | relative_url }}" alt=""/>
      </div>
    </div>
</div>
<br/>

## Dependencies

- cmake >= 3.11
  - All OSes: [click here for installation instructions](https://cmake.org/install/)
- make >= 4.1 (Linux, Mac), 3.81 (Windows)
  - Linux: make is installed by default on most Linux distros
  - Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  - Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
- gcc/g++ >= 5.4
  - Linux: gcc / g++ is installed by default on most Linux distros
  - Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  - Windows: recommend using [MinGW](http://www.mingw.org/)
- wxWidgets >= 3.0
  - Linux: `sudo apt-get install libwxgtk3.0-dev libwxgtk3.0-0v5`. If you are facing unmet dependency issues, refer to the [official page](https://wiki.codelite.org/pmwiki.php/Main/WxWidgets30Binaries#toc2) for installing the unmet dependencies.
  - Mac: There is a [homebrew installation available](https://formulae.brew.sh/formula/wxmac).
  - Installation instructions can be found [here](https://wiki.wxwidgets.org/Install). Some version numbers may need to be changed in instructions to install v3.0 or greater.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./membot`.

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/Memory-Management-Chatbot" %}
</div>

## Credits

The starter code for this project is from the [Udacity C++ Nanodegree Program](https://www.udacity.com/course/c-plus-plus-nanodegree--nd213): Memory Management.
