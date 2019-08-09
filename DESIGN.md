## Timer, the universal task scheduler

### Design docs

#### Constraints:

* We want the Scheduler to be both able to operate on its own, and when bound to a ticking engine.

* When running bound to a ticking engine, registration methods use the amount of ticks that have to pass before the task is run. Similarly, timer tasks' period counter is also measured in ticks.

* When bound to a ticking engine, the engine has (at the end, or beginning of a tick) call .Tick() on the Scheduler - it'll decrement the counter for all registered events, and if time is up - run them.

* When running in standalone mode, registration methods use the amount of milliseconds, and the scheduler has its own thread that runs the timer and executes callbacks.


### Implementation thoughts

* We want the scheduler to be able to run in both tick-bound and standalone modes. However, making the scheduler use an interface that matches both tickbound and standalone registrations is difficult, as one operates on ticks, and the other - on miliseconds.

### Solution:
**The scheduler has a Timescale field. The timescale field reveals how many milliseconds pass between each tick. For standalone, this is always equal to one, and in any tick-bound engine, it has to be set in the constructor to the engine's tick delay.**

* The above effectively means, that with the timescale set properly (and defaulting to one for standalone) regardless of whether you run in a tick-bound engine, or in standalone, dividing the *real time* you want to pass by the timescale will result in registering the task to run in that time without knowing any of the nuances of the engine that runs the scheduler. This is a very portable and future-proof solution: If the underlying engine is modified to run at a different tickrate, all events that are supposed to run in X seconds will correctly adjust to use the new tickrate. Similarly, you can separate the scheduler to run on its own thread, and all tasks registered in that scheduler will be fired at correct times, with the exception of not being synchronised to the engine thread.

* The scheduler will have two sets of methods available: 
* * The first is the safe choice, which uses real time and does the conversion for you. This is suitable for 99% of usecases, as you mostly work with real timespans, not game ticks. 
* * The second is using "scheduler ticks" which are a time measure dependent on the specific configuration of the scheduler, which allow you to skip the calculation and to run tasks at specific tick offsets, rather than at specific real time offsets. This is useful in some situations, mostly familiar to those with Bukkit backgrounds, and those working with engine quirks (such as: simulating a game mechanic expressed in ticks)
