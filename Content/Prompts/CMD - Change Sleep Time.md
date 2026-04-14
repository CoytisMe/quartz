---
category: script
tags:
  - cmd
  - power
  - sleep
  - settings
---

**dc = plugged in**
**ac = battery**

## This is the only one you really want
**Turns off sleep while plugged.**
~~~
powercfg /x -standby-timeout-ac 0
~~~


### The rest, work it out.

~~~
powercfg /x -hibernate-timeout-ac 0
~~~

~~~
powercfg /x -hibernate-timeout-dc 0
~~~

~~~
powercfg /x -disk-timeout-ac 0
~~~

~~~
powercfg /x -disk-timeout-dc 0
~~~

~~~
powercfg /x -monitor-timeout-ac 0
~~~

~~~
powercfg /x -monitor-timeout-dc 0
~~~

~~~
powercfg /x -standby-timeout-dc 0
~~~