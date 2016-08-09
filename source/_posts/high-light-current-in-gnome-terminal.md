title: high light current in gnome terminal
date: 2016-08-09 07:06:56
tags:
---

Tested in Ubuntu 16.04:

edit  ~/.config/gtk-3.0/gtk.css

```
TerminalWindow .notebook tab:active {
        background-color: #b0c0f0;
}
```
