Mecode
======


### GCode for all

Mecode is designed to simplify GCode generation. It is not a slicer, thus it
can not convert CAD models to 3D printer ready code. It simply provides a
convenient, human-readable layer just above GCode. If you often find
yourself manually writing your own GCode, then mecode is for you. This a 
fork of jminardy's mecode that was modified by DerAndere so it can be used
to communicate with USB controller boards that feature microcontrollers that by 
design reset upon serial connection (e.g the Anet V1.0 board of the 
Anet A2, A3, A6 or A8 3D-printer which has a Microchip Atmel ATmega 
microcontroller of the AVR family) using Pyhon 3.7. 

Basic Use with under Microsoft Windows
--------------------------------------
Connect a CNC controller board that runs a firmware for communication and G-code
interpretation (e.g. Marlin 2.0) to the computer. For this example the computer
runs Windows 10 and the connection is via USB and the controller board is 
recognized at virtual com port COM3. On the computer, create a 
Python script that contains the following code: 
```python
import mecode  #  See https://github.com/DerAndere1/mecode . License: MIT 
impoet time  # provides sleep()
g = mecode.G(direct_write=True, direct_write_mode="serial", printer_port="COM3", baudrate=115200)   # direct communication via serial connection at port COMx (with x=3) under Microsoft Windows.
g.write("M302 S0")   # send g-Code. Here: allow cold extrusion. Danger: Make sure extruder is clean without filament inserted 
g.write("G28")   # send g-Code. Here: Home all axis 
g.move(10, 10, 10)   # move 10mm in x and 10mm in y and 10mm in z
g.retract(10)   # move extruder motor
time.sleep(10)  # wait 10 seconds to make sure all commands where executed 
```
When running the above Python script, a USB serial connection will be 
established and G-codes will be sent directly to the controller board via 
that USB serial connection. Best performance is achieved when using full speed
USB 2.0 or better (480 Mbit/s, e.g. with the newest Smoothieboard running Marlin 
2 firmware).

Basic Use
---------
To use, simply instantiate the `G` object and use its methods to trace your
desired tool path.

```python
from mecode import G
g = G()
g.move(10, 10)  # move 10mm in x and 10mm in y
g.arc(x=10, y=5, radius=20, direction='CCW')  # counterclockwise arc with a radius of 20
g.meander(5, 10, spacing=1)  # trace a rectangle meander with 1mm spacing between passes
g.abs_move(x=1, y=1)  # move the tool head to position (1, 1)
g.home()  # move the tool head to the origin (0, 0)
```

By default `mecode` simply prints the generated GCode to stdout. If instead you
want to generate a file, you can pass a filename and turn off the printing when
instantiating the `G` object.

```python
g = G(outfile='path/to/file.gcode', print_lines=False)
```

*NOTE:* `g.teardown()` must be called after all commands are executed if you
are writing to a file. This can be accomplished automatically by using G as
a context manager like so:

```python
with G(outfile='file.gcode') as g:
    g.move(10)
```

When the `with` block is exited, `g.teardown()` will be automatically called.

The resulting toolpath can be visualized in 3D using the `mayavi` or `matplotlib`
package with the `view()` method:

```python
g = G()
g.meander(10, 10, 1)
g.view()
```

The graphics backend can be specified when calling the `view()` method, e.g. `g.view('matplotlib')`.
`mayavi` is the default graphics backend.

All GCode Methods
-----------------

All methods have detailed docstrings and examples.

* `set_home()`
* `reset_home()`
* `feed()`
* `dwell()`
* `home()`
* `move()`
* `abs_move()`
* `arc()`
* `abs_arc()`
* `rect()`
* `meander()`
* `clip()`
* `triangular_wave()`

Matrix Transforms
-----------------

A wrapper class, `GMatrix` will run all move and arc commands through a 
2D transformation matrix before forwarding them to `G`.

To use, simply instantiate a `GMatrix` object instead of a `G` object:

```python
g = GMatrix()
g.push_matrix()      # save the current transformation matrix on the stack.
g.rotate(math.pi/2)  # rotate our transformation matrix by 90 degrees.
g.move(0, 1)         # same as moves (1,0) before the rotate.
g.pop_matrix()       # revert to the prior transformation matrix.
```

The transformation matrix is 2D instead of 3D to simplify arc support.

Renaming Axes
-------------

When working with a machine that has more than one Z-Axis, it is
useful to use the `rename_axis()` function. Using this function your
code can always refer to the vertical axis as 'Z', but you can dynamically
rename it.

Installation
------------

The easiest method to install mecode is with pip:

```bash
sudo pip install mecode
```

To install from source:

```bash
$ git clone https://github.com/jminardi/mecode.git
$ cd mecode
$ pip install -r requirements.txt
$ python setup.py install
```

Optional Dependencies
---------------------
The following dependencies are optional, and are only needed for
visualization. An easy way to install them is to use
[Canopy][0] or [conda][1].

* numpy
* mayavi
* matplotlib

[0]: https://www.enthought.com/products/canopy/
[1]: https://store.continuum.io/cshop/anaconda/

TODO
----
* add pressure box comport to `__init__()` method
* build out multi-nozzle support
    * include multi-nozzle support in view method.
* factor out aerotech specific methods into their own class

Credits
-------
This software was developed by the [Lewis Lab][2] at Harvard University.
The files printer.py and main.py were modified by [DerAndere][3],[4] to 
resolve issues with serial communication and to replace os.path library with 
libraries pathlib and inspect for future-proof compatibility with Python 3.7.
Files header.txt and footer.txt were adapted to replace Aerotech-specific 
content with specific content for the lab robot [pipetBot-A8][3],[4] to be 
usable with the Graphical G-code Generator / robot control software 
[GGCGen by DerAndere][3],[5].

[2]: http://lewisgroup.seas.harvard.edu/
[3]: https://it-by-derandere.blogspot.com
[4]: http://www.github.com/DerAndere1/
[5]: http://www.gitlab.com/DerAndere/