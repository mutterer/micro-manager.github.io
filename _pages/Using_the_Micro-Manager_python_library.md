---
autogenerated: true
title: Using the Micro-Manager python library
redirect_from: /wiki/Using_the_Micro-Manager_python_library
layout: page
section: Extend
---

The easiest way to control Micro-Manager through Python is the
[Pycro-manager](https://github.com/micro-manager/pycro-manager) library.

The instructions below are for an alternative mechanism in which you
compile the micro-manager core yourself with python bindings.

**MMCorePy** is a wrapper that allows you to control microscope hardware
from python interactive session or script. It's support Windows, Mac and
Linux.

Micromanager's main parts:

-   CMMCore - basic module, written in C++. Script languages like python
    just wrap it by [swig](http://www.swig.org).
-   Device adapters - various libraries that allow support for various
    hardware. If you want to built one and extend MM devise support,
    follow [this
    guide](Building_Micro-Manager_Device_Adapters).
-   MMCorePy - python wrapper. MM build scripts are support both python
    2 and 3, but windows version still ships with python 2 bindings
    only.
-   MMCoreJ - java wrapper
-   MMStudio - Micromanager GUI (technically it is ImageJ plugin).

## Environment setup

You must install python2 and numpy. Windows users may prefer using an
python distribution instead manual separate installation.

### Manual

-   [python 2.7.x](http://python.org/) (python2 is default for windows
    now)
-   [numpy 1.7.x](http://scipy.org/) Micromanager represent imaging data
    as multidimensional numpy arrays.

### Using python distributions

It's convenient to install a distribution which includes Python, numpy,
scientific libraries, GUI frameworks and IDEs. All distributions have a
free version, some of them have extended paid version, and you can
request free academic license.

-   [Enthought's python distribution
    (EPD)](https://www.enthought.com/products/epd/free/)
-   [Anaconda](http://continuum.io/downloads), has package manager.
-   [PythonXY](https://code.google.com/p/pythonxy) totally free.

### Useful libraries

-   Scipy - scientific algorithms, multidimensional image processing
    toolbox.
-   Matplotlib - fastest way to show your image data.
-   Opencv - computer vision and image processing library. Sometimes
    faster than scipy.
-   Pillow - very basic image processing. Scipy uses it for image
    loading and writing.
-   Scikit-image - "pythonic" scientific-oriented image processing
    algorithms collection.
-   [IPython](http://ipython.org/) - improved interactive python
    environment

## Micromanager installation

### Windows & Mac

[Download](Download_Micro-Manager_Latest_Release) and install
Micro-Manager on your computer. Add Micromanager installation folder to
[PYTHONPATH](https://docs.python.org/2/using/cmdline.html#envvar-PYTHONPATH)
(i.e. "*C:\\Program Files\\Micro-Manager-1.4*", it should contain
<small>*MMCorePy.py*</small> and <small>*\_MMCorePy.pyd*</small> files).
Create variable if it not exist. At now you can import MMCorePy without
an error.

### Linux

MM package from your distribution repository, in most cases, ships with
CMMCore, MMCorePy (python 2 or 3 wrapper) and MMCoreJ, but without GUI.
Python and Numpy would be installed by your package manager as
dependency. In normal way, you don't need changing any system variable.

## Using Python API

Familiarize yourself with Micro-Manager and learn how to connect it to
your hardware by MMStudio GUI.

-   Find your device on [this page](Device_Support) and
    figure out what adapter you need.
-   [Micro-Manager Configuration
    Guide](Micro-Manager_Configuration_Guide) help you to
    understand how properties work.
-   Read the general [Micro-Manager Programming
    Guide](Micro-Manager_Programming_Guide)
-   Use
    [API](https://valelab.ucsf.edu/~MM/doc/MMCore/html/class_c_m_m_core.html).

### First steps

{% include notice icon="info" content="Next code snippets aims to be most generic. We
use numpy and matplotlib **as is** from pure python interactive shell, but
it's convenient to use IPython with nice autocompletion
capabilities." %}

Start python interactive session. Import \`MMCorePy\` and make sure
everything is working properly.

```
   >>> import MMCorePy
   >>> mmc = MMCorePy.CMMCore()  # Instance micromanager core
   >>> mmc.getVersionInfo()
   'MMCore version 2.3.2'
   >>> mmc.getAPIVersionInfo()
   'Device API version 59, Module API version 10'
```

We just get some basic information about current Micromanager
installation. If there an \`ImportError\`, check your PYTHONPATH
variable. If output is too verbose, run
<small>mmc.enableStderrLog(False); mmc.enableDebugLog(False)</small>.

### Device loading

{% include notice icon="info" content="You can get all needed parameter's names from Micromanager configuration file, generated by MMStudio." %}

Let's take step closer to hardware. Micromanager have couple of dummy
devices, suitable for learning purposes. Load DemoCamera:

```
   # Demo camera example, continuation of previous listing
   >>> mmc.loadDevice('Camera', 'DemoCamera', 'DCam')
   >>> mmc.initializeAllDevices()
   >>> mmc.setCameraDevice('Camera')
```

### Property discovery

Every device has
[properties](Micro-Manager_User's_Guide#exploring-devices-deviceproperty-browser)
- settings that let you control the device more precisely. Default
values should be fine, but if you need something sophisticated, [this
example](https://github.com/radioxoma/micromanager-samples/blob/master/mm_print_properties.py)
help you figure out how to explore it.

### Snapping single image

Images returned as numpy array by calls to an instance of the pythonized
Micro-Manager
[CMMCore](https://valelab.ucsf.edu/~MM/doc/MMCore/html/class_c_m_m_core.html)
class. The array <small>dtype</small> depends on property named
*PixelType* (see below).

#### Grayscale

```
   >>> mmc.snapImage()
   >>> img = mmc.getImage()  # img - it's just numpy array
   >>> img
   array([[12, 12, 13, ..., 11, 12, 12],
          [12, 12, 13, ..., 11, 12, 12],
          [12, 13, 13, ..., 12, 12, 12],
          ...,
          [22, 22, 22, ..., 22, 22, 22],
          [22, 22, 22, ..., 22, 22, 22],
          [22, 22, 22, ..., 22, 22, 22]], dtype=uint8)
```

DemoCamera snaps grayscale 8-bit image, by default. It presented as
two-dimensional numpy array. Let's show image data with matplotlib.

```
   >>> import matplotlib.pyplot as plt
   >>> plt.imshow(img, cmap='gray')
   >>> plt.show()  # And window will appear
```

#### Color

Of course, color image is more suitable for optical microscopy purposes.
So take one, if your camera support it:

```
   >>> mmc.setProperty('Camera', 'PixelType', '32bitRGB')  # Change pixel type
   >>> rgb32 = mmc.getImage()
   >>> rgb32
   array([[1250067, 1250067, 1315860, ..., 1250067, 1250067, 1250067],
          [1250067, 1315603, 1315860, ..., 1250067, 1250067, 1250067],
          [1250067, 1315859, 1315860, ..., 1250067, 1250067, 1250067],
          ...,
          [1246483, 1246483, 1246483, ..., 1181204, 1246740, 1246484],
          [1246483, 1246483, 1246483, ..., 1246740, 1246740, 1246483],
          [1246483, 1246483, 1312019, ..., 1246740, 1246740, 1246483]], dtype=uint32)
```

Interesting output isn't it? We expect something like 3-dimensional RGB
array, but get bunch of 32-bit uints in 2-D shape.

#### Numpy array

Now we should look at RGB32 pixel data structure. Every pixel has 32-bit
depth and contain 4 values for blue, green, red and blank channel. Blank
channel is more technical peculiarity, than necessity.

```
low memory address    ---->      high memory address
| pixel | pixel | pixel | pixel | pixel | pixel |...
|-------|-------|-------|-------|-------|-------|...
|B|G|R|A|B|G|R|A|B|G|R|A|B|G|R|A|B|G|R|A|B|G|R|A|...
```

[http://avisynth.nl/index.php/RGB32](http://avisynth.nl/index.php/RGB32)

Let's numpy handle that.

```
>>> import numpy as np
>>> rgb32.shape
(512, 512)
>>> rgb = rgb32.view(dtype=np.uint8).reshape(
        rgb32.shape[0], rgb32.shape[1], 4)[...,2::-1]
>>> rgb.shape
(512, 512, 3)
>>> rgb.dtype
dtype('uint8')
```

It is a fastest way to get pixel data as RGB array without copying.
There is no conversion - just creating new view to same data. Now you
can process image with scipy or scikits-image. Note, that opencv uses
BGR order (replace slice to `[..., :3]` for that).

### Continuous acquisition

{% include notice icon="warning" content="'''Don't run this code directly.''' It's a partial sample." %}

```
mmc.startContinuousSequenceAcquisition(1)
while True:
    if mmc.getRemainingImageCount() > 0:
        frame = mmc.getLastImage()
        # or frame = mmc.popNextImage()
```

### Code examples

A longer example script,
[MMCoreWrapDemo.py](https://github.com/mdcurtis/micromanager-upstream/blob/master/bindist/any-platform/MMCoreWrapDemo.py),
is available in the Micro-Manager root directory.

Also check out [micromanager-samples
repo](https://github.com/radioxoma/micromanager-samples) (live video
acquisition, property discovery etc).

## Further reading

-   [Image manipulation and processing using Numpy and
    Scipy](http://scipy-lectures.github.io/advanced/image_processing/index.html)
    by Python Scientific Lecture Notes.
-   [Scikits-image
    gallery](http://scikit-image.org/docs/dev/auto_examples)
-   [Lectures on scientific computing with
    Python](https://github.com/jrjohansson/scientific-python-lectures)
-   [OpenCV-Python
    Tutorials](http://docs.opencv.org/trunk/doc/py_tutorials/py_tutorials.html)

Written by Eugene Dvoretsky -- [Radioxoma](/users/Radioxoma)
09:19, 14 June 2014 (PDT)
