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
3. A Documentation Window

This is meant to be modular and easy to hack on so that we can build quick windows and functions without having to build up the basics each time. 
<--->
![Dev Kit Diagram](/development_module/kit_diagram.png)
{{</columns>}}

## Code


