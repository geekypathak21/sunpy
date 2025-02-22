.. _plotting:

********
Plotting
********

sunpy makes use of `Matplotlib <https://matplotlib.org/>`_ for all of its plotting - as such, it tries to follow the Matplotlib plotting philosophy.
We'll start by going over the basics of plotting in Matplotlib.
If you are already familiar with Matplotlib, skip ahead to the second section.

Matplotlib Tutorial
*******************

This tutorial is a summary of one that can be found in the `Matplotlib usage documentation <https://matplotlib.org/stable/users/explain/api_interfaces.html>`__.

Matplotlib provides two main pathways for plotting.
One is meant for interactive use (e.g. on the command-line) and the other for non-interactive use (e.g. in scripts).
It is important to recognize though that the interactive-use pathway (referred to as pyplot) just provides shortcuts for doing many of the more advanced non-interactive functions in the background.
It is therefore possible to switch between the two as necessary and it is possible to use pyplot in a non-interactive way.
In this manner pyplot is just a shortcut to making it quicker to set up plot axes and figures.
In order to get access to the full interactive capabilities of pyplot it is necessary to turn this feature on.

Pyplot
------
Here is a simple example of pyplot usage.

.. plot::
    :include-source:

    import matplotlib.pyplot as plt
    plt.plot(range(10), range(10))
    plt.title("A simple Plot")
    plt.show()

The `~matplotlib.pyplot.show` command opens a plot on the screen and blocks execution until the plot window is closed.
The `~matplotlib.pyplot.show` command only works once - if you call `~matplotlib.pyplot.show` again after the above code is executed nothing happens.

To turn on interactive plotting for pyplot use the command ::

    >>> plt.ion()   # doctest: +SKIP

In interactive mode, the plot will appear at the first `~matplotlib.pyplot.plot` command and most commands will update the plot as you call them.
Here is some example code::

    >>> plt.plot(range(10), range(10))   # doctest: +SKIP
    >>> plt.title("Simple Plot")   # doctest: +SKIP

In this example, you'll see that the title appears right on the plot when you call it.
Note that in this case the `~matplotlib.pyplot.show` command isn't needed as the plot shows up right when you create it.

The following command ::

    >>> plt.ioff()   # doctest: +SKIP

turns off interactivity.

Advanced Pyplot
---------------
If you need more fine-grained control over plots the recommended path is to use pyplot and access the figures and axes objects.
This is shown in the following example.

.. plot::
    :include-source:

    import matplotlib.pyplot as plt
    import numpy as np
    x = np.arange(0, 10, 0.2)
    y = np.sin(x)
    fig = plt.figure()
    ax = plt.subplot()
    ax.plot(x, y)
    ax.set_xlabel('x')
    plt.show()

In Matplotlib, `~matplotlib.figure.Figure` is the top-level container for a single figure and `~matplotlib.axes.Axes` is the top-level container for a set of axes.
The above example creates a figure then creates an axes and populates the plot in ``ax``.
With this method you now have your hands on the `~matplotlib.axes.Axes` object so you can do things
like change the labels on the x and y axes or add a legend.
In the previous section, pyplot took care of creating these objects for you so you didn't have to worry about creating them yourself.

Plotting in sunpy
*****************

To be consistent with Matplotlib, sunpy has developed a standard plotting interface which supports both simple and advanced Matplotlib usage.
The following examples focus on the map object, but both ``peek()`` and ``plot()`` work on time series objects too.

peek()
------
For quick and easy access to a plot `~sunpy.map.GenericMap` and `~sunpy.timeseries.GenericTimeSeries` define their own ``peek()`` methods which create a plot for you and show it without you having to deal with any Matplotlib setup.
This is so that it is easy to take a quick look at your data.
For example:

.. plot::
    :include-source:

    import sunpy.map
    import sunpy.data.sample
    smap = sunpy.map.Map(sunpy.data.sample.AIA_171_IMAGE)
    smap.peek(draw_limb=True)

This creates a plot window with all axes defined, a plot title, and the image of the map data defined by the contents of the map.
In non-interactive mode the plot window blocks the command line terminal and must be closed before doing anything else.

plot()
------

For more advanced plotting the base sunpy objects also provide a `~sunpy.map.mapbase.GenericMap.plot` command.
This command is similar to the pyplot `~matplotlib.pyplot.imshow` command in that it will create a figure and axes object for you if you haven't already.

When you create a plot with `~sunpy.map.GenericMap.peek` or
`~sunpy.map.GenericMap.plot`, sunpy will use `astropy.visualization.wcsaxes` to
represent coordinates on the image accurately, for more information see
:ref:`wcsaxes-plotting`.

Using `~sunpy.map.GenericMap.plot` it is possible to customise the look of the
plot by combining sunpy and matplotlib commands, for example you can over plot
contours on the Map:

.. plot::
    :include-source:

    import matplotlib.pyplot as plt
    import astropy.units as u

    import sunpy.map
    import sunpy.data.sample

    aia_map = sunpy.map.Map(sunpy.data.sample.AIA_171_IMAGE)
    aia_map.plot()
    aia_map.draw_limb()

    # let's add contours as well
    aia_map.draw_contours([10,20,30,40,50,60,70,80,90] * u.percent)

    plt.colorbar()
    plt.show()


In this example, the `~matplotlib.figure.Figure` and
`~astropy.visualization.wcsaxes.WCSAxes` instances are created explicitly, and
then used to modify the plot:

.. plot::
    :include-source:

    import matplotlib.pyplot as plt
    import astropy.units as u
    from astropy.coordinates import SkyCoord

    import sunpy.map
    import sunpy.data.sample

    smap = sunpy.map.Map(sunpy.data.sample.AIA_171_IMAGE)

    fig = plt.figure()
    # Provide the Map as a projection, which creates a WCSAxes object
    ax = plt.subplot(projection=smap)

    im = smap.plot()

    # Prevent the image from being re-scaled while overplotting.
    ax.set_autoscale_on(False)

    xc = [0,100,1000] * u.arcsec
    yc = [0,100,1000] * u.arcsec

    coords = SkyCoord(xc, yc, frame=smap.coordinate_frame)

    p = ax.plot_coord(coords, 'o')

    # Set title.
    ax.set_title('Custom plot with WCSAxes')

    plt.colorbar()
    plt.show()

It is possible to create the same plot, explicitly not using `~astropy.visualization.wcsaxes`, however, this will not have the features of `~astropy.visualization.wcsaxes` which include correct representation of rotation and plotting in different coordinate systems.
Please see this example :ref:`sphx_glr_generated_gallery_map_plot_frameless_image.py`.

.. _wcsaxes-plotting:

Maps with coordinate systems
----------------------------

By default :ref:`map` uses the `astropy.visualization.wcsaxes` module to improve
the representation of world coordinates, and calling
`~sunpy.map.GenericMap.plot` or `~sunpy.map.GenericMap.peek()` will use wcsaxes
for plotting. Unless a standard `matplotlib.axes.Axes` object is explicitly
created.

To explicitly create a `~astropy.visualization.wcsaxes.WCSAxes` instance do the
following ::

    >>> fig = plt.figure()   # doctest: +SKIP
    >>> ax = plt.subplot(projection=smap)   # doctest: +SKIP

when plotting on an `~astropy.visualization.wcsaxes.WCSAxes` axes, it will by
default plot in pixel coordinates, you can override this behavior and plot in
'world' coordinates by getting the transformation from the axes with
``ax.get_transform('world')``.

.. note::

    World coordinates are always in **degrees** so you will have to convert to degrees.

.. code-block:: python

    >>> smap.plot()   # doctest: +SKIP
    >>> ax.plot((100*u.arcsec).to_value(u.deg), (500*u.arcsec).to_value(u.deg),
    ...         transform=ax.get_transform('world'))   # doctest: +SKIP

Finally, here is a more complex example using sunpy maps, wcsaxes and Astropy
units to plot a AIA image and a zoomed in view of an active region.

.. plot::
    :include-source:

    import matplotlib.pyplot as plt
    from matplotlib import patches
    import astropy.units as u
    from astropy.coordinates import SkyCoord

    import sunpy.map
    import sunpy.data.sample

    # Define a region of interest
    length = 250 * u.arcsec
    x0 = -100 * u.arcsec
    y0 = -400 * u.arcsec

    # Create a sunpy Map, and a second submap over the region of interest.
    smap = sunpy.map.Map(sunpy.data.sample.AIA_171_IMAGE)
    bottom_left = SkyCoord(x0 - length, y0 - length,
                        frame=smap.coordinate_frame)
    top_right = SkyCoord(x0 + length, y0 + length,
                        frame=smap.coordinate_frame)
    submap = smap.submap(bottom_left, top_right=top_right)

    # Create a new matplotlib figure, larger than default.
    fig = plt.figure(figsize=(5, 12))

    # Add a first Axis, using the WCS from the map.
    ax1 = fig.add_subplot(2, 1, 1, projection=smap)

    # Plot the Map on the axes with default settings.
    smap.plot()

    # Draw a box on the image
    smap.draw_quadrangle(bottom_left, height=length * 2, width=length * 2)

    # Create a second axis on the plot.
    ax2 = fig.add_subplot(2, 1, 2, projection=submap)

    submap.plot()

    # Add a overlay grid.
    submap.draw_grid(grid_spacing=10*u.deg)

    # Change the title.
    ax2.set_title('Zoomed View', pad=35)

    # Add some text
    ax2.text(
        (-100*u.arcsec).to_value(u.deg),
        (-300*u.arcsec).to_value(u.deg),
        'A point on the Sun',
        color="white",
        transform=ax2.get_transform('world')
    )

    plt.show()
