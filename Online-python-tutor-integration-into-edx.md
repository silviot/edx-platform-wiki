Created by Carlos Rocha and Philip Guo
[for edX Hackathon 4](https://github.com/edx/edx-platform/wiki/Hackathon-four%3A-the-more-the-merrier)

The goal of this project was to integrate Philip's open-source [Online Python Tutor](http://www.pythontutor.com) project into edX Studio.

The code lives in the [rocha/pythontutor branch](https://github.com/edx/edx-platform/tree/rocha/pythontutor) of edx-platform.

## Motivation

Lots of students and instructors are already using [Online Python Tutor](http://www.pythontutor.com) in Computer Science MOOCs such as 6.00x, and instructors of several new edX courses have already expressed an interest in using this tool.

Thus, it would be great to provide an easy way for instructors to insert those visualizations as modules in their courses.

## Results

This hack involved creating a module and interface in edX Studio that lets instructors create a Python Tutor component. This component embeds the visualization in an `iframe`, so no code needs to be hosted on edX servers. Here's a screenshot:

![Online Python Tutor in edX Studio](http://pgbovine.net/opt/opt-edx-studio.png)