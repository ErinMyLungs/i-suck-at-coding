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
* `init_windows` to create all defined windows in the component
* `run` method that allows running just that component solo

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
        Creates and arranges our windows
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

For example let's pretend dearpygui.core added a new function
```python
def print(val):
    return str(val).upper()
```
Then on import any time you called `print('some value')` instead of it going to your console it'll call the print function you accidentally imported.

By doing `import dearpygui.core as c` you don't overwrite the default print function and can still access it by calling `c.print`.

I also find it much easier to track where things come from.
{{</hint>}}

Okay that's super basic, let's add a logger window!

### Creating Logger Window

Here we define window configuration. You can do a `s.show_logger()` but by defining our own window we gain a few advantages:

* We can give the logger its own name
  * This lets us set up multiple logger windows that don't collide
* We can control positioning and size


I like to define specific window creation functions as 'private' (remember that there are no truly private functions in python!) because it's semi-rare to call these methods directly. They are mostly chained into an overall `init_windows` method.

This helps autocomplete when we call other log methods.
{{<columns>}}<!-- begin columns block -->
#### Code
```python
def _logger_window(self):
    """
    Creating our logger window
    """
    window_name = (
        self.logger.replace("_", " ").title()
        + "##window"
    )
    with s.window(
        name=window_name,
        width=500,
        height=500,
        x_pos=500,
        y_pos=350,
        no_scrollbar=True,
    ):
        c.add_logger(
            name=self.logger,
            autosize_x=True,
            autosize_y=True,
        )

def init_windows(self):
    """
    Creates and arranges our windows
    """
    self._logger_window()
    c.set_main_window_size(1000, 850)
    c.set_main_window_pos(x=280, y=0)

```
<--->
#### Expected Output
![Logger window](/development_module/logger_window.png)

Note how the x_pos and y_pos kwargs lets us push this in the corner.
{{</columns>}}

### Adding logger helper methods

Because we stored the logger name in our `self.logger` variable, this makes it very easy for us to ensure our logs are getting to the right place. Here we are going to define some basic helper functions and then we'll do some advanced python wizardry using decorators.


{{<columns>}}<!--begin columns block -->
#### Code
```python
def log(self, message):
    """
    Convenience method because log_info
    is just too many letters to type.
    """
    self.log_info(message=message)

def log_info(self, message):
    """
    Logs at info level
    """
    c.log_info(message=message, logger=self.logger)

def log_debug(self, message):
    """
    Logs at debug level
    """
    c.log_debug(message=message, logger=self.logger)

def log_warning(self, message):
    """
    Logs at warning level
    """
    c.log_warning(
        message=message, logger=self.logger
    )

def log_error(self, message):
    """
    Logs at error level
    """
    c.log_error(message=message, logger=self.logger)
```
<--->
#### Expected Output
![Logger with helpers](/development_module/logger_with_helpers.png)

If you want to see each of these examples, change `run` to look like this:
```python
def run(self):
    """
    Simple run method
    """
    self.init_windows()
    self.log("Log")
    self.log_info("Log Info")
    self.log_debug("Log Debug")
    self.log_error("Log Error")
    self.log_warning("Log Warning")
    c.start_dearpygui()
```
{{</columns>}}

#### More Advanced Logger Helpers - Decorators

Okay those are pretty basic, let's make a few advanced decorators to make our lives easier.

{{<columns>}}<!--begin columns block -->
#### Code
```python
import functools #top of file

def log_return(self, func):
    """
    An external decorator for dumping a function
    return value into the logger.
    """

    @functools.wraps(func)
    def wrap(*args, **kwargs):
        result = func(*args, **kwargs)
        self.log_info(
            f"{func.__name__} return value:"
        )
        self.log_info(result)
        self.log_info("-" * 20)
        return result

    return wrap

def log_all(self, func):
    """
    A decorator for dumping a function's
    arguments and result into a logger
    """

    @functools.wraps(func)
    def wrap(*args, **kwargs):
        name = func.__name__

        # Logging Arguments
        self.log_info(f"{name} Arguments:")
        for argument in args:
            self.log(f"\t{argument}")

        # Logging kwargs if exist
        if len(kwargs) > 0:
            self.log(f"{name} Key Word Arguments:")
            for key, value in kwargs.items():
                self.log(f"\t {key} : {value}")

        # Logging Return Value
        result = func(*args, **kwargs)
        self.log(f"{name} Return Value:")
        self.log(f"\t{result}")
        self.log("-" * 20)
        return result

    return wrap
```
<--->
#### Explanation

Decorators are a bit complex but they basically allow for you to modify functions externally instead of re-writing them.

These are called via `@log_return` or `@log_all` like so:

```python
#Add this under the if __name__ == "__main__":
@dev.log_return
def external_log_example():
    """
    This picks a random number, prints it in console
    and returns the picked value.

    You should be able to see the same value in
    console and in the logger.
    """
    import random

    random_number = random.randint(0, 10)
    print(random_number)
    return random_number 
```

Here's what's happening - our function `external_log_example` is being wrapped so the name of the function, it's return value, and a horizontal line is logged before the `random_number` variable is returned.

And for `@log_all`
```python
#Add this under the if __name__ == "__main__":
@dev.log_all
def echo_scream(string_to_echo, scream=True):
    """
    An example function to show args/Kwargs
    :param string_to_echo: string to echo back
    :param scream: if True run .upper() on string
    :return: string echoed response
    """
    if scream is True:
        string_to_echo = string_to_echo.upper()
    print(string_to_echo)
    return string_to_echo
```
This will log all the args, kwargs, and return result into the logger.



{{</columns>}}