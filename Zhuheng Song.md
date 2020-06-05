### Table of Content

1. [2020.03.11 System Selection](#20200311-System-Selection)
2. [2020.03.20 Environment Selection](#20200320-Environment-Selection)
3. [2020.03.20 Environment Setup](#20200320-Environment-Setup)
4. [2020.04.21 Framework Setup](#20200421-Framework-Setup)
5. [2020.04.22 Windows Only Bug Fixing](#20200422-Windows-Only-Bug-Fixing)
6. [2020.04.28 Multiprocessing Webots Controller](#20200428-Multiprocessing-Webots-Controller)
7. [2020.04.29 Plain Output in Webots on Windows Fixing](#20200429-Plain-Output-in-Webots-on-Windows-Fixing)
8. [2020.04.30 Real Rover & Webots Rover Compatibility](#20200430-Real-Rover--Webots-Rover-Compatibility)
9. [2020.04.30 Keyboard Event Support](#20200430-Keyboard-Event-Support)
10. [2020.05.13 Webots Devices Access Fixing](#20200513-Webots-Devices-Access-Fixing)
11. [2020.05.17 Finish Sign Output](#20200517-Finish-Sign-Output)
12. [2020.05.22 Data Delay in Queue Fixing](#20200522-Data-Delay-in-Queue-Fixing)
13. [2020.05.23 Force Termination Fixing](#20200523-Force-Termination-Fixing)
14. [2020.05.25 Data Delay in Queue Fixing2](#20200525-Data-Delay-in-Queue-Fixing2)
15. [2020.05.27 Signal Inexistence Fixing](#20200527-Signal-Inexistence-Fixing)
16. [2020.05.30 Data Delay in Queue Fixing3](#20200530-Data-Delay-in-Queue-Fixing3)
17. [2020.06.02 Refactor and Performance](#20200602-Refactor-and-Performance)

---

## 2020.03.11 System Selection

| Experiment Title   | System Selection                                |
| ------------------ | ----------------------------------------------- |
| Experiment Purpose | Decide architecture of the system               |
| Keyword            | Multi-processing, Multi-threading, Architecture |
| Conclusion         | Decide to design a multiprocessing system.      |
| Issues             | /                                               |

![img](Zhuheng Song/regular_development.svg)

There is no doubt that a whole control process goes like this:

1. Abstract data from sensors into signals to show state of the rover

2. Make decision of following behavior depending on current state

3. control the rover to do the required behavior

Since here we do not have very complex situations to make decisions and we do not have to very complex tasks to do, it is obvious that **each stage takes different amount of time**. I suppose **Perception** takes a lot time, while time cost by **Decision** stage may be really short that we could even ignore it. In addition, **Control** stage actually need an **separate**, **high frequency** loop to keep high precision control of the rover.

Therefore, we need three separate loops:

- A perception loop keeps processing sensors data and transmit signals to decision loop. It should send out signals every time a signal is updated, but not send once when all signals are updated. In this way, we could always do decisions with the latest signals.
- A decision loop keeps sending commands to control loop depending on signals received from perception loop.
- A control loop keeps controlling the rover to act like what the decision loop asks to do, for example apply a PID control on velocity of four wheels.

Then we found two normal ways to allow three separate loops in one program:

1. multi-processing
2. multi-threading

After some research we found that:

> If your program is a **CPU intensive** application, multi-processing is recommended, While multi-threading is more suitable for **I/O intensive** applications.

- A CPU intensive application is: for example, which has lots of loops or calculating steps.
- A I/O intensive application is: for example, mainly deal with files, or a web crawler.

Since multi-threading is running concurrently, most implements use a counter to decide whether switch to another thread. Therefore a CPU intensive application could reaches the counter threshold very soon, and need to compete for time to execute again.

On the other hand, for I/O intensive applications which have lots of situations where has to wait for **external** events, multi-threading could save a lot of resource, as for these situations, we could switch to another thread.

Given this, we decide to design a **multiprocessing** system.

## 2020.03.20 Environment Selection

| Experiment Title   | Environment Selection                               |
| ------------------ | --------------------------------------------------- |
| Experiment Purpose | Decide the development tool chain and their version |
| Keyword            | Python, OpenCV, Raspberry Pi                        |
| Conclusion         | Python is selected as the main programming language |
| Issues             | Performance of RPi4 need to be tested               |

Since our program does not have strict requirement on speed, factors of programming language selection of our program are:

- easy to get start with, because we could not find a language that everyone in the team are familiar with
- easy to configure the tool chain
- easy to develop and debug
- has powerful library to process images, so we do not have to start from very beginning
- allow running the program just on our own computers but not only on the rover, so that everyone could get involved in

Considering all factors above, we decide to select **Python** as programming language of our program.

So the configuration is as follow:

| Item     | Argument                                                     |
| -------- | ------------------------------------------------------------ |
| Board    | [Raspberry Pi 4b](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/specifications/) |
| System   | [Raspbian Buster](https://www.raspberrypi.org/downloads/raspberry-pi-os/) |
| Language | [Python](https://www.python.org/) 3.7.3                      |
| OpenCV   | [opencv-contrib-python](https://pypi.org/project/opencv-contrib-python/) 4.2.0.34 |

## 2020.03.20 Environment Setup

| Experiment Title   | Framework Setup                                              |
| ------------------ | ------------------------------------------------------------ |
| Experiment Purpose | setup specified environment and write a shell script to do that, then test performance of RPi4 |
| Keyword            | shell, configuration, Linux, RPi4                            |
| Conclusion         | Performance of RPi4 is totally wonderful                     |
| Issues             | /                                                            |


A bash script to auto setup the environment on RPi4 is written:

```bash
#! /bin/bash
# ------------------------------------------------------------------------------
# Install essential softwares and packages.
#
# To descrease download speed apt source and Raspbian source are changed to
# tsinghua. Another side effect is that default pip and python are changed to
# pip3 and python3.
#
# ❗Notice that you should download the spefified opencv-contrib-python wheel
# file to **Desktop** before executing it. (you can get download link in README)
# ------------------------------------------------------------------------------

# backup apt source list and Raspbian source list and change them to tsinghua mirror
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo cp /etc/apt/sources.list.d/raspi.list /etc/apt/sources.list.d/raspi.list.bak
sudo sh -c 'echo "deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib" > /etc/apt/sources.list'
sudo sh -c 'echo "deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib" >> /etc/apt/sources.list'
sudo sh -c 'echo "deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui" > /etc/apt/sources.list.d/raspi.list'
sudo apt update

# remove the soft links to pip2 and python2
sudo rm /usr/bin/pip
sudo rm /usr/bin/python
# then install update-alternatives
sudo update-alternatives --install /usr/bin/pip pip /usr/bin/pip3 1
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1

# change to tsinghua pip mirror and upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip install pip -U

# install OpenCV and its dependencies
sudo apt-get install -y libhdf5-dev libhdf5-103
sudo apt-get install -y libqtgui4 libqtwebkit4 libqt4-test python3-pyqt5
sudo apt-get install -y libatlas-base-dev
sudo apt-get install -y libjasper-dev
pip install ~/Desktop/opencv_contrib_python-4.1.0.25-cp37-cp37m-linux_armv7l.whl  # ❗this wheel file has to be there first
```

The performance of RPi4 is proved to be very high, as the CPU usage of RPi4 is only about 25% when I watch 1080P video in browser on it. Its performance is far higher than RPi3b+. We do not need to worry about this any more.

## 2020.04.21 Framework Setup

| Experiment Title   | Environment Selection                      |
| ------------------ | ------------------------------------------ |
| Experiment Purpose | work out the basic framework of the system |
| Keyword            | Python, Module, Class, Queue               |
| Conclusion         | My design seems feasible                   |
| Issues             | /                                          |

To make it easy for subgroups to develop their code separately, and also make it easier to call in the main process, we decide to abstract code of each stage into a class, and put them into different modules, that is to say, different files.

That is:

- **chassis** module: `Controller` class
- **decision** module: `Decider` class
- **detection** module: `Detector` class

[multiprocessing.Queue()](https://docs.python.org/3/library/multiprocessing.html#multiprocessing.Queue) is used to communicate between processes. Although according to a [Stack Overflow answser](https://stackoverflow.com/a/8463046/10088906), **`multiprocessing.Pipe()` performs much faster**, but `Queue()` seems more easy to write, so use `Queue()` to communicate between processes for the present. There seems to be no delay in the queues so far.

Code in the main process looks like this:

```python
# define the processes
def move(movement_queue):
    rover = chassis.Chassis()
    while True:
        if not movement_queue.empty():
            rover.set_state(movement_queue.get())


def sense(sense_queue):
    detector = sensor.Sensor()
    detector.run(sense_queue)


def decide(sense_queue, move_queue):
    decider = decision.Decision()
    decider.run(sense_queue, move_queue)


start = time.time()
info('start at:' + str(start))
# create pipes
move_queue = multiprocessing.Queue()
sense_queue = multiprocessing.Queue()

# create processes and start them
move_process = multiprocessing.Process(target=move, args=(move_queue, ))
sense_process = multiprocessing.Process(target=sense, args=(sense_queue, ))
decide_process = multiprocessing.Process(target=decide, args=(sense_queue, move_queue,))
move_process.daemon = True
sense_process.daemon = True
decide_process.daemon = True
move_process.start()
sense_process.start()
decide_process.start()

while True:
    # now use time to be finish flag signal
    if time.time() - start > 1:  # 1s
        flag_finish_patio = 1
    if flag_finish_patio:
        break
```

## 2020.04.22 Windows Only Bug Fixing

| Experiment Title   | Framework Setup                                             |
| ------------------ | ----------------------------------------------------------- |
| Experiment Purpose | fix a Windows only bug related to multiprocessing in Python |
| Keyword            | Multi-processing, Windows, Module                           |
| Conclusion         | It seems Windows does not support multiprocessing that well |
| Issues             | /                                                           |

https://github.com/TDPS-Mihotel/Mihotel/commit/f596f7a00628964994f570b3bd3b7b968d8b1e78

When running the code above on Windows, got following error message:

``` shell
RuntimeError:
An attempt has been made to start a new process before the current
process has finished its bootstrapping phase.
```

After some research found out that lines related to multiprocessing need to be under `if __name__ == "__main__"`

## 2020.04.28 Multiprocessing Webots Controller

| Experiment Title   | Multiprocessing Webots Controller                            |
| ------------------ | ------------------------------------------------------------ |
| Experiment Purpose | try to merge our multiprocessing framework into webots controller |
| Keyword            | Webots, Multiprocessing                                      |
| Conclusion         |                                                              |
| Issues             |                                                              |

https://github.com/TDPS-Mihotel/Mihotel/commit/25b28aacbf9244f37ab6ee26aa66ec87367cc803

https://github.com/TDPS-Mihotel/Mihotel/commit/7c99397c3791545c5a65ccc0bc75749dcffb5cbd

## 2020.04.29 Plain Output in Webots on Windows Fixing

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/8771398c14971b905962861cb1de8c545d6c52fc

## 2020.04.30 Real Rover & Webots Rover Compatibility

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/c18485fcd669e015734d5b8a1d6ba2b850770542

## 2020.04.30 Keyboard Event Support

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/fb98b8576c32fc084975f67bbba72cf72967efbe

## 2020.05.13 Webots Devices Access Fixing

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/ba8d1d9142acb6e8fdff722719e9e387dfd5453b

https://github.com/TDPS-Mihotel/Mihotel/commit/84fbc64524f766934a4cac060e2ea5d5363b4294

## 2020.05.17 Finish Sign Output

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

## 2020.05.22 Data Delay in Queue Fixing

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/cd71ba74f301a7041cd27b96ec57f1464959dcc6

## 2020.05.23 Force Termination Fixing

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/a3061c61ba421171fe6a6e9cdc1392c41560efa9

https://github.com/TDPS-Mihotel/Mihotel/commit/17d78d013bfd090386f1ca338c4bda64c6bda664

## 2020.05.25 Data Delay in Queue Fixing2

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/13247d5e7aaa6246b1399515236ce47b577edc4a

## 2020.05.27 Signal Inexistence Fixing

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/39c1ef74ffca84696e14d2fcb1adbe4239000356

## 2020.05.30 Data Delay in Queue Fixing3

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/4e77bb28b77202a564703fa336fc5e110dcac142

https://github.com/TDPS-Mihotel/Mihotel/commit/4c2b7783ddab01784b9c44c5c3a0891a6ac0e578

## 2020.06.02 Refactor and Performance

| Experiment Title   |      |
| ------------------ | ---- |
| Experiment Purpose |      |
| Keyword            |      |
| Conclusion         |      |
| Issues             |      |

https://github.com/TDPS-Mihotel/Mihotel/commit/b0b035c62b241ffe9d51122a44ee14f89aec20db