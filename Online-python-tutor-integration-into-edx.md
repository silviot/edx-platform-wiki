Created by Carlos Rocha and Philip Guo for
[edX Hackathon 4](https://github.com/edx/edx-platform/wiki/Hackathon-four%3A-the-more-the-merrier)

The goal of this hack is to integrate Philip's open-source [Online Python Tutor](http://www.pythontutor.com) project into edX Studio.

## Motivation

Tens of thousands of students are already using [Online Python Tutor](http://www.pythontutor.com) in Computer Science MOOCs such as 6.00x, and instructors of several new edX courses have expressed interest in using this tool as well.

Thus, it would be great to provide an easy way for instructors to insert these code visualizations as modules in their courses.


## Results

For this hack, we created a module and user interface in edX Studio that lets instructors create a Python Tutor component.

This component embeds the visualization in an `iframe`, so no code needs to be hosted on edX servers. Here's a screenshot:

![Online Python Tutor in edX Studio](http://pgbovine.net/opt/opt-edx-studio.png)

## Code

The code lives in the [rocha/pythontutor branch](https://github.com/edx/edx-platform/tree/rocha/pythontutor) of edx-platform.
