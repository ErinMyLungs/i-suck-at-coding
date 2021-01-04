# Creating a Dev Tool Module for DPG

## Overview

**This is a 2 part series:**
1. [Setting up live code execution in a live DPG program]({{<relref development_environment.md>}})
2. Set up a general development UI kit that can be re-used easily for each component and project (You are here)
------
In the prior tutorial we put together a window that lets us directly run code and interact with a DPG program. Here we take that and expand it into a small but reusable development kit.

## Approach
{{<columns>}}<!-- begin columns block -->
### Design
-----
We're going to keep it simple and small:

1. A code input with execution ability
2. A logger with convenience methods
3. A Debugger Window

This is meant to be modular and easy to hack on so that we can build quick windows and functions without having to build up the basics each time. 
<--->
![Dev Kit Diagram](/development_module/kit_diagram.png)
{{</columns>}}


## Code

The components we need are:
* A logger
* An execution window
* A debug window


Let's Define Our Class:
{{<columns>}}<!-- begin columns block -->
### Dev Kit Class

Here we do these steps:
1. Define our `__init__` method
   * the `logger` var will define the logger name
2. Create an `init_windows` method
   * This creates all windows and updates main window params
3. Create a `run` method
   * I tend to make run be a simple initialize all windows and start DPG

#### This is how I structure all my UI components in DPG:
* `__init__` with any state variables that I want to consistently track
* A method (`init_windows` here) to create all defined windows in the component
* A `run` method that allows running just that component solo

In general this seems to work pretty well and I follow this pattern frequently.
<--->
```python
import dearpygui.core as c
import dearpygui.simple as s

class DevKit:
    def __init__(
            self,
            logger: str = "",
        ):
            self.logger = logger
    def init_windows(self):
        """
        Creates and arranges our windows for the kit
        """
        c.set_main_window_size(1000, 850)
        c.set_main_window_pos(x=280, y=0)

    def run(self):
        """
        Simple run method
        """
        self.init_windows()
        c.start_dearpygui()

dev = DevKit("An Example Logger")
if __name__ == "__main__":
    dev.run()
```
{{</columns>}}
{{<hint info>}}
**Importing dearpygui and star imports**

Importing via `from dearpygui.core import *` or any modules generally is discouraged.

For example if in dearpygui.core there is a function
```python
def print(val):
    return
```
Then on import any time you called `print('some value')` instead of it going to your console it'll call the print function you accidentally imported.

By doing `import dearpygui.core as c` you don't overwrite the default print function and can still access it by calling `c.print`.

I also find it much easier to track where things come from.
{{</hint>}}

Okay that's super basic, let's add a logger window.