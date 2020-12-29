# Making a Live Dev Environment in DPG

## This is a 2 part series:

1. Setting up live code execution in a live DPG program ( You are here)
2. [Set up a general development UI kit that can be re-used easily for each component and project]({{<relref development_module.md>}} "Dev Tool Module Link")

## Overview

Here we're going to set up a development environment that will allow dynamically excute code on the fly while your program is running. The biggest perk is that you can execute multiple commands on the fly to quickly test and iterate on your ideas without having to reload your program quite as frequently. After that we will expand this out and wrap it in an overall dev-UI module that will launch all the dev windows cleanly with some helper methods to make our life easier.

## Live Code Execution

The method and approach is very simple: we create a window with a text input that has a callback function that runs exec on the text input value.



{{<columns>}} <!-- begin columns block -->
### Approach
* We create an input window
* The execution callback injects our code into the live program

<--->

### Approach Diagram

![Approach Diagram](/Live_Enu_Diag.png)

{{</columns>}}


## Code

```python 
import dearpygui.core as c
import dearpygui.simple as s


def execute(*_args):
    """
    Executes arbitrary command input in running program
    :param _args: This catches sender and data arguments that we aren't using here
    :return: None, executes command
    """
    command = c.get_value("command##input")
    exec(command)


with s.window(name="command##window", autosize=True, x_pos=0, y_pos=0):
    c.add_input_text(name="command##input",
                     width=500,
                     height=300,
                     multiline=True,
                     on_enter=True,
                     callback=execute,
                     )

c.start_dearpygui()

```

You should see something somewhat like:

![Window screenshot](/live_env_example_1.png)

Okay big deal, we created a window with a text input.

But the cool thing is that you can type in any python and it'll execute live!


### Example code to execute:
```python
with s.window("Hello World", autosize=True):
    c.add_text("Hello world!")

c.show_logger()
c.log_info("Foo")
c.log_info("Check it out ma - no IDE!")

with s.window("Canvas", x_pos=0, y_pos=300, autosize=True):
    c.add_drawing("Draw", width=300, height=300)
    c.draw_circle("Draw",
                  center=[150, 150],
                  radius=50,
                  color=[125, 125, 125],
                  fill=[125, 125, 200])
````

Running this should cause your window to look like:

![After live code execution](/live_code_execution_2.png)

### Security Considerations

{{<hint danger>}}
### **exec as a security risk**

`exec` is powerful but it's power also lends it to be a huge security hole! It tries to execute any string passed into it without any validation or protection. Executing python code from untrusted input like this is dangerous AF.

For development and personal projects it's not a big deal but for any real apps with potential malicious actors, this is incredibly dangerous.

{{</hint>}}

Imagine your mean co-worker walking up to your laptop and writing
```python
from pathlib import Path
# DON'T RUN THIS
for file in Path('/').glob('**/*'):
    if file.is_file():
        print(f"{file} is deleted")
        # file.unlink()
```
Okay I commented out the actual command here (in case you ran it out of curiosity) but this is basically python's version of 'delete System32' and would quickly fuck over your computer.

We can mitigate this risk by isolating our development tools into a developer module so that in production they cannot be invoked. We work on this in [part 2]({{<relref development_module.md>}} "Dev Tool Module Link").

Funny story - when commiting these examples to github pylint auto-flagged exec as a problem.

## A More Complex Example

We have a project involving making an old-school digital clock style display and we're we're trying to get it to render numbers that look like:

![A digital number 8](/more_complex_target.png)

We have created a 'Number' class and created a method called `draw_line` that lets us draw the shape on the canvas in the proper orientation but now that we can draw the shapes, we use the methods from above to play with this code and find coordinates to draw an 8.

The full code is annotated below, you can pretty much run it as is and it'll put you into a live console that you can start playing with the display.

{{<hint info>}}

**Code-Style**

I tend to use classes and object-oriented programming methods. You can use whatever style suites you. I mostly use classes as a way to bundle all the methods and state together but you can replace the attributes in `__init__` with `c.set_data/c.get_data` calls and tear out the self from all function definitions.

{{</hint>}}


{{<expand "Annotated Code">}}
```python
""" Shows more complex execution usage"""
from enum import Enum

import dearpygui.core as c
import dearpygui.simple as s


class Direction(str, Enum):
    """
    An enum to indicate which of the 5 line types should be rendered
    This isn't super critical, mostly makes selecting specific values easy.
    """
    up = "up"
    down = "down"
    left = "left"
    right = "right"
    center = "center"


class Number:
    """
    Class for creating a single 'digital' number display
    """

    def __init__(self, color=None):
        self.name = "Canvas" # our drawing name
        # This sets color of polygon drawn below
        if not color:
            self.color = [120, 255, 120]
        else:
            self.color = color

    def clear(self):
        """
        Clears the canvas drawing
        """
        c.clear_drawing(self.name)

    # Feel free to skip the messy code below, this draws a polygon.
    def draw_line(
            self,
            x_offset=0,
            y_offset=0,
            direction: Direction = Direction.left,
            tag: str = None,
    ):
        """
        Creates a line in the number box in the digital clock format
        :param x_offset: x translation magnitude, as x gets larger, the line shifts right
        :param y_offset: y translation magnitude, as y increases the line shifts down
        :param direction: Which type of line to render
        :param tag: string to tag the line with
        :return: drawn polygon on self.canvas
        """
        if tag is None:
            tag = str(direction.value)
        translate = lambda coords: [coords[0] + x_offset, coords[1] + y_offset]

        if direction == Direction.left:
            position = [(0.0, 0.0), (20.0, 20.0), (20.0, 60.0), (0.0, 80.0)]
        elif direction == Direction.up:
            position = [(0.0, 0.0), (20.0, 20.0), (60.0, 20.0), (80.0, 0.0)]
        elif direction == Direction.down:
            position = [(80.0, 20.0), (60.0, 0.0), (20.0, 0.0), (0.0, 20.0)]
        elif direction == Direction.right:
            position = [(20.0, 0.0), (0.0, 20.0), (0.0, 60.0), (20.0, 80.0)]
        else:
            position = [
                (0.0, 10.0),
                (10.0, 0.0),
                (50.0, 0.0),
                (60.0, 10.0),
                (50.0, 20.0),
                (10.0, 20.0),
                (0.0, 10.0),
            ]

        c.draw_polygon(
            self.name,
            points=list(map(translate, position)),
            color=[255, 255, 255, 0],
            fill=self.color,
            tag=tag,
        )

    def execute(self, *_args):
        """
        Our execution callback again
        :param _args: catching sender/data
        :return: executed command
        """
        command = c.get_value("command##input")
        exec(command)

    def run(self):
        """
        This method defines our windows and creates our command input dev
        window. 
        """

        window_args = dict(
            autosize=False,
            height=200,
            width=200,
            x_pos=0,
            y_pos=0,
        )
        with s.window("Drawing", **window_args):
            c.add_drawing(self.name, width=90, height=150, )

        with s.window("command##window", autosize=True, y_pos=200, x_pos=0):
            c.add_input_text(name="command##input", width=600, height=300, multiline=True, on_enter=True,
                             callback=self.execute)

        c.start_dearpygui()


if __name__ == '__main__':
    # We instantiate the class and launch our GUI
    num = Number()
    num.run()

```
{{</expand>}}