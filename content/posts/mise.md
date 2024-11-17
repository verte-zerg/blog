---
title: "Replace your *env tools with Mise"
date: 2024-11-17T16:40:28+01:00
draft: false
---

I've noticed that starting a new fish shell takes at least 2 seconds, and it's really starting to annoy me.

# Profiling

I decided to profile my `config.fish` script. Since I couldn't find any profilers, I resorted to the oldest method: putting print and time statements everywhere in my fish script. I quickly found that the main bottleneck was *the section initializing my env tools*, so I wrapped it in the `time` command to measure the execution time.

```sh
time pyenv init - | source
# _______________________________________________________
# Executed in  264.81 millis    fish           external
#    usr time   80.20 millis    1.47 millis   78.73 millis
#    sys time  140.85 millis    1.82 millis  139.04 millis

time goenv init - | source
# _______________________________________________________
# Executed in    2.96 secs    fish           external
#    usr time    0.70 secs    1.65 millis    0.70 secs
#    sys time    1.83 secs    2.09 millis    1.83 secs

time direnv hook fish | source
# _______________________________________________________
# Executed in   15.11 millis    fish           external
#    usr time    2.68 millis   83.00 micros    2.60 millis
#    sys time    5.30 millis  172.00 micros    5.13 millis

time direnv export fish | source
# _______________________________________________________
# Executed in   11.63 millis    fish           external
#    usr time    2.64 millis   25.00 micros    2.62 millis
#    sys time    3.57 millis  360.00 micros    3.21 millis


time rbenv init - --no-rehash fish | source
# _______________________________________________________
# Executed in   51.44 millis    fish           external
#    usr time   16.64 millis  217.00 micros   16.42 millis
#    sys time   29.18 millis  387.00 micros   28.79 millis

```

What the hell: `goenv` takes about 3 seconds to start. I'm not sure what it's doing, but it's too long to continue using it. By the way, `pyenv` also doesn't look great - 0.3 seconds.

# Searching for alternatives

After researching alternatives to `goenv`, I came across [`asdf`](https://asdf-vm.com/) and [`mise`](https://mise.jdx.dev/), which caught my attention. These are tool version managers that can install and switch between multiple versions of different languages and tools. They can replace not only `goenv`, but **all** env tools.

A quick comparison showed that `mise` is a modern alternative to `asdf`, written in Rust, and it seems to have much better performance. At least, that's what the developers [claim](https://mise.jdx.dev/dev-tools/comparison-to-asdf.html#performance). Their documentation even has a [section](https://mise.jdx.dev/tips-and-tricks.html#macos-rosetta) describing how to install Rosetta versions of tools, so they sold me on it.

# Integration

Step by step, I replaced `goenv`, `pyenv`, `direnv`, and `rbenv` with `mise`. The process was very simple:
- install `mise`: `brew install mise`
- delete initialization lines from `config.fish` of all *env tools
- put `mise activate fish | source` in the `config.fish` file

That's it. After that, just go to your project folder and type: `mise install` and you will be happy.

# Conclusion

I cleaned up my fish startup script by replacing all these tools with one line that takes only 0.05 seconds to execute. And completely moved to `mise`, as it's a drop-in replacement for all *env tools. It even detects and respects `.<language>-version` files, and it just works seamlessly.

```sh
time mise activate fish | source
# ________________________________________________________
# Executed in   44.73 millis    fish           external
#    usr time    8.40 millis    0.61 millis    7.79 millis
#    sys time   10.83 millis    1.53 millis    9.30 millis
```

I can't even recall a case when one tool replaced a bunch of others and did it in a much more intuitive and faster way. Really great job!
