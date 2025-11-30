---
layout: page
title: System Monitor
description: |
  The system monitor for linux operation system created using C++. It provides information such as cpu usage, memory usage, processes information. The program reads system information from various files in directories like /proc/ and /etc/, computes parameters and shows them in a graphic interface in terminal.
img: /assets/img/projects/Cpp/system-monitor/system_monitor.png
importance: 2
category: Cpp
github: https://github.com/yangfan/System-Monitor
---

In this project, I created a simple system monitor using c++. As shown in figure below, the system monitor provides the system information and processes information.

<div class="row justify-content-center">
    <div class="col-8">
          <img class="img-fluid z-depth-1" src="{{ 'assets/img//projects/Cpp/system-monitor/system_monitor.png' | relative_url }}" alt=""/>
    </div>
</div>
<br/>

The information is obtained from files in linux system. The path of these directories are presented in the table below.

| Info              | directory path    |
| ----------------- | ----------------- |
| OS                | `/etc/os-release` |
| Kernal            | `/proc/version`   |
| CPU               | `/proc/stat`      |
| Memory            | `/proc/meminfo`   |
| Total Processes   | `/proc/stat`      |
| Running Processes | `/proc/stat`      |
| Up Time           | `/proc/uptime`    |

<br/>
Note that the memory usage shown in this system monitor is non-cache/buffer memory.

The files containing processes information are located in `/proc/` directory. The PID is shown as the directory name in `/proc/`.

| Info         | directory path                      |
| ------------ | ----------------------------------- |
| USER         | `/proc/[pid]/status`, `/etc/passwd` |
| CPU Usage    | `/proc/[pid]/stat`                  |
| Memory Usage | `/proc/[pid]/status`                |
| Up Time      | `/proc/[pid]/stat`                  |
| Command      | `/proc/[pid]/cmdline`               |

<br/>

## Compiling and Running

### Compiling

This project requires [ncurses](https://www.gnu.org/software/ncurses/) for displaying outputs. Install this library by command in terminal:  
`sudo apt install libncurses5-dev libncursesw5-dev`.

After installation of dependencies, clone this project and create build directory in root of the project.  
`mkdir build && cd build`.

Then run `cmake ..` and `make` in `build` directory.

### Running

The executable is in `results/` directory. To run the system monitor, go to root directory of the project and run command:  
`./results/monitor`.

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/System-Monitor" %}
</div>

## Credits

Starter code for System Monitor Project in the Object Oriented Programming Course of the [Udacity C++ Nanodegree Program](https://www.udacity.com/course/c-plus-plus-nanodegree--nd213).
