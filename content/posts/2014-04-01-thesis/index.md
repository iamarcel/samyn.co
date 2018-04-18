---
title: "Thesis Adventures - April 1"
date: 2018-04-01T10:52:55+02:00
draft: false
---

## The Post-Processing Step

The OpenPose + 3D Pose Baseline code from [this blog post](http://akasuku.blog.jp/archives/73745862.html), has some odd
post-processing:

```python
max = 0
min = 10000
for i in range(poses3d.shape[0]):
    for j in range(32):
        tmp = poses3d[i][j * 3 + 2]
        poses3d[i][j * 3 + 2] = poses3d[i][j * 3 + 1]
        poses3d[i][j * 3 + 1] = tmp
        if poses3d[i][j * 3 + 2] > max:
            max = poses3d[i][j * 3 + 2]
        if poses3d[i][j * 3 + 2] < min:
            min = poses3d[i][j * 3 + 2]

for i in range(poses3d.shape[0]):
    for j in range(32):
        poses3d[i][j * 3 + 2] = max - poses3d[i][j * 3 + 2] + min
        poses3d[i][j * 3] += (spine_x - 630)
        poses3d[i][j * 3 + 2] += (500 - spine_y)
```

The first block swaps the y and z axes and computes the height minimum and
maximum. Okay. What's odd here is that the original `3d-pose-baseline` code does
*not* do this. It's probably so matplotlib shows the result nicely (by default,
the z axis is the height instead of the depth).

The second block is *really* mysterious to me. Sure, the first line inverts the
height. But where do the 630 and 500 constants come from? Why does every point
need to be offset'ed with respect to the spine?

I can't see a difference between the two (when I update the plotting code as
necessary so they're in the same orientation):

<video controls src="v1.mp4" style="max-width: 100%"></video>
<video controls src="v2.mp4" style="max-width: 100%"></video>


## Land of the Hunches
In 3D, the person is hunched over.
