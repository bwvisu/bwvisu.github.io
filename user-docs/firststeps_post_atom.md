# pastAtom user guide #


**postAtom**, a framework for large-scale data visualization of particles, volumes, and vector fields.

# Using the application #

After starting the application, you'll want to load a particle, volume, or flow data set.

* After loading a dataset, you can select different visualization modi in the toolbar (Default mode for particle data is 'Glyph').
* On the bottom left you find general settings, where you can e.g. scale the data set or change the particle size.
* You can edit the mapping from attributes to color and save/load them on the bottom of the application.

### Navigating the scene ###

* Use the left mouse button to rotate around the center point
* The center can be translated with the right mouse button
* Zoom in or out with the mouse wheel

### Using The Color Map Editor ###

* The Color Map Editor is used to assign a color and opacity to data values
* Pre-defined color map can be loaded. Several are bundled with the application.
* The minimum and maximum value (over all time steps) of the selected data attribute is shown on the left and on the right of the editor.
* You can set a custom min/max value by clicking or by using the sliders

## Flow (vector field) analysis

Visualization of uniform, gridded (Eulerian) vector fields. A specific NetCDF and AMIRA formats are the supported data format.

## Visualizing particles ##

* You can open particle data in the .h5part format
* When opening a file for the first time, global minima and maxima are detected and stored to disk for later re-use. This might take a while.

### Particle Flow analysis ###

Some features in this mode require CUDA support (and enough GPU RAM for the data).

* Shows multiple views to visualize and analyze the data.
* You can add more views in the menu (e.g. View -> Add scatter plot).

Time-dependent features:
* You can compute the FTLE and LCS distance for the chosen time-interval (t_end) and smoothing length (dx)
* Other time-dependent features can be computed, including divergence and vorticity. These quantities are computed for every step in the interval and are summed and normalized (Lagrangian accumulation).

Parallel coordinate plots are incredible useful in this mode ( View -> Add parallel coordinate plot)
* Select a region on an axis using the left mouse button.
* You can also switch axes using the left mouse button, by clicking on an axis and then releasing the button on the respective other axis.
* By selecting an axis with a left click and then dragging it far away, the dimension can be removed
* By right clicking on an axes, all filters on that axis will be removed
* Right clicking between axes will remove all filters

Points:
* By default, the particles are rendered as small spheres. The size can be changed in the general settings.
* You can change to "Gaussians", which lead to a more smooth, continous result since particles are not rendered as points/spheres but as semi-transparent Gaussians.

Arrows:
* Particles can be rendered as vector arrow glyphs that represent velocity (requires 'u','v','w' datasets)
* To avoid occlusion, arrows are only shown up to a user-defined max. number that can be changed

Surfaces:
* After performing a selection, you can click *(Extract surface)* to extract a surface from the selection (with the resp. resolution)
* Make sure to enable rendering surfaces

Pathlines:
* You should set tEnd and the max. number of pathlines. Click generate.
* Note that for larger datasets pathline generation might take a while (since all timesteps in [current, tEnd] have to be loaded)
* You can now select the number of lines, i.e. the number of seed points, and the start and end time. Furthermore, you can select an attribute and edit the Color Map.
* In the general settings, you can set the thickness of the pathlines
* Make sure to enable rendering pathlines

Clusters:
* Renders spheres but uses the 'diameter' attribute for size.
