For [[Hackathon 4|Hackathon four: the more the merrier]], [[Miles Steele|https://github.com/mlsteele]] and [[Renzo Lucioni|https://github.com/rlucioni]] prototyped a method of visualizing student paths through edX courseware. We came up with the following interactive graph displaying randomly generated data.

You can find the code in its [[GitHub repository|https://github.com/mlsteele/courseware-traversal]], and you can also view a [[live demo|http://mlsteele.github.io/courseware-traversal/]].

In both screenshots shown below, curves above the horizontal axis indicate forward movement through the courseware (e.g., advancing to the next lecture). Similarly, curves below the horizontal axis indicate backwards movement through the courseware (e.g., going back to review). Neither of these graphs actually have an explicit horizontal axis. What appears to be a horizontal axis is composed of student paths which visit each component of the courseware in order.

In the colored graph pictured below, the thickness of each line represents how many students followed the path indicated by that line. Mousing over a line will make it black and more opaque.
![visualization-colorful](http://snag.gy/mv1lK.jpg)

In this grayscale graph, each line represents the path of a single student. Mousing over a line will make it red and more opaque.
![visualization-br](http://snag.gy/tIzZN.jpg)