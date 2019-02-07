---
title: "Running your experiments and analyzing your results"
teaching: 0
exercises: 0
questions:
- "How to submit your short run?"
- "How to continue your run"
objectives:
- "Be able to do a quick run to test an experiment"
- "If successful resubmit the same experiment for a longer period"
- "Analyze the outputs from your experiment"
- "Prepare graphs using python"
keypoints:
- "Start an experiment from a spinup"
- "Continue an existing experiment"
- "Evaluate the CPU time required for a long run"
- "Make custom plots"
---

Now you are ready to submit your simulation.

<font color="red">On Abel:</font>

<pre>cd ~/cesm_case/f2000.T31T31.$EXPNAME

./f2000.T31T31.$EXPNAME.submit

squeue -u $USER
</pre>

If your simulation is **unsuccessful** you have to understand what happened!

There are in particular log files in the run directory (/work/users/$USER/f2000.T31T31.$EXPNAME/run/) which can provide some clues, although the error messages are not always explicit...

Open the latest log file with your favorit text editor (vi, emacs, etc.) and try to search for keywords like "ERROR" or "Error" or "error" (remember that the search is case sensitive).

Then correct any identified bug.

If your short simulation has **finished without crashing**, check the outputs: were your changes taken into account? Do you get significant results?

### Model timing data

A summary timing output file is produced after every CESM run. On Abel and in our case this file is placed in /work/users/$USER/archive/f2000.T31T31.$EXPNAME/cpl/logs and is nammed cpl.log.$date.gz (where $date is a datestamp set by CESM at runtime).

This file contains information which is useful for *load balancing a case* (i.e., to optimize the processor layout for a given model configuration, compset, grid, etc. such that the cost and throughput will be optimal).

For this lesson we will concentrate on the last few lines in the file and in particular the number of simulated years per computational day, which will help us evaluate the wallclock time required for long runs.

<font color="red">On Abel:</font>

<pre>vi cpl.log.190205-144355.gz

.......................
(seq_mct_drv): ===============       SUCCESSFUL TERMINATION OF CPL7-CCSM ===============
(seq_mct_drv): ===============        at YMD,TOD =    90201       0      ===============
(seq_mct_drv): ===============  # simulated days (this run) =    31.000  ===============
(seq_mct_drv): ===============  compute time (hrs)          =     0.347  ===============
(seq_mct_drv): ===============  # simulated years / cmp-day =     5.873  ===============
(seq_mct_drv): ===============  pes min memory highwater  (MB)   50.429  ===============
(seq_mct_drv): ===============  pes max memory highwater  (MB)  517.162  ===============
(seq_mct_drv): ===============  pes min memory last usage (MB)   -0.001  ===============
(seq_mct_drv): ===============  pes max memory last usage (MB)   -0.001  ===============
</pre>

*Here the throughput was 5.873 simulated years / cmp-day and it took 0.347 * 60 ~ 21 minutes to run the first month. Assuming that the other months will take approximately the same time, that represents about 3 months per hour and a bit more than 4 hours for 12 months.*


### Long experiment (14 months)

As for the previous exercice, you will work **in pairs** for this practical and you will **analyze the model outputs in pairs**.  
You will be using your previous experiment ~/cesm_case/f2000.T31T31.$EXPNAME (EXPNAME should be set depending on your experiment!) and run 14 months.  

#### Set a new duration for your experiment

Make sure you set the duration of your experiment properly. Here we wish to run 14 months from the control restart experiment but as it is a long run, we would rather continue to split it into chuncks of 1 month. 

*Note that splitting an experiment into small chunks is good practice: this way if something happens and the experiment crashes (disk quota exceeded, hardware issue, etc.) everything will not be lost and it will be possible to resume the run from the latest set of restart files.*

<font color="red">On Abel:</font>

<pre># Set EXPNAME properly

cd ~/cesm_case/f2000.T31T31.$EXPNAME
</pre>

Since we have already the first month done, we are going to continue the experiment instead of starting from scratch.

<font color="red">On Abel:</font>

<pre>./xmlchange CONTINUE_RUN=TRUE
</pre>

To perform a 14 months experiment, we would need to repeat this one month experiment 13 times. 

For this purpose there is a CESM option called RESUBMIT.

<font color="red">On Abel:</font>

<pre>./xmlchange -file env_run.xml -id RESUBMIT -val 13
</pre>


By setting this option, CAM5 will be running one month of simulation (once submitted) and automatically resubmit the next 12 months.  

<font color="red">On Abel:</font>
<pre>cd ~/cesm_case/f2000.T31T31.$EXPNAME

./f2000.T31T31.$EXPNAME.submit
</pre>


Regularly check your experiment (and any generated output files) and once it is fully done, [store your model outputs on norStore](norstore.html).

# Store model outputs on norStore

First make sure that your run was successful and check all the necessary output files were generated.  

To post-process and visualize your model outputs, it is VERY IMPORTANT you move them from Abel to norStore. Remember that all model outputs are generated in a semi-temporary directory and all your files will be removed after a few weeks!  

If you haven't set-up your [SSH keys](http://www.mn.uio.no/geo/english/services/it/help/using-linux/ssh-tips-and-tricks.html), the next commands (ssh and [rsync](http://www.tecmint.com/rsync-local-remote-file-synchronization-commands/)) will require you to enter your Unix password.  

Make sure you define EXPNAME properly (it depends on your experiment).

<font color="red">On Abel:</font>

<pre># If you are running CO2 experiment (otherwise adjust: sea_ice, SST, rocky)
export EXPNAME=CO2
</pre>

Then copy the archived files from abel to the norStore project area.

*It is sometimes sensible to also copy the run files and even the case directory, but that should not be necessary for this lesson.*

<font color="red">On Abel:</font>

<pre>ssh login.nird.sigma2.no 'mkdir -p /projects/NS1000K/GEO4962/outputs/$USER/archive'

rsync -avz /work/users/$USER/archive/f2000.T31T31.$EXPNAME $USER@login.nird.sigma2.no:/projects/NS1000K/GEO4962/outputs/$USER/archive/.
</pre>

Once the previous commands are successful, you are ready to [post-process and visualize](../../results.html) your data on login.nird.sigma2.no  

However, as your simulation is stored on the norStore project area, you can now [archive your experiment](archive.html) on the norStore archive (long-term archive i.e. several years).

# Post processing and visualization

You can always compare the results of your experiments to the control run, at any time (i.e., this applies for both the short and long runs).

An easy way to do this is to calculate the difference between for example the surface temperature field issued from the control run and that from your new experiment.

# Copy your output files from Abel to your virtual machine

Start a new **Terminal** on your JupyterHub and transfer your data. Do not forget to replace *YOUR_USER_NAME* by your actual user name and *YOUR_EXPERIMENT* by your actual experiment name (you have to do this because the Virtual machine and Abel are different systems, therefore all the environment variables that were defined on Abel are not known here).

<font color="blue">On the JupyterHub terminal:</font>

<pre>rsync -avzu --progress YOUR_USER_NAME@abel.uio.no:/work/users/YOUR_USER_NAME/archive/f2000.T31T31.YOUR_EXPERIMENT/ /opt/uio/GEO4962/$USER/f2000.T31T31.YOUR_EXPERIMENT/
</pre>

# Visualization with psyplot

Start a new **python3** notebook on your JupyterHub and type the following commands (in this example the *USER* is jeani and we have the first month of data from the sea ice experiment).

<font color="green">On jupyter:</font>

<pre>import psyplot.project as psy

month = '0009-01'

path = 'GEO4962/outputs/runs/f2000.T31T31.control/atm/hist/'
filename = path + 'f2000.T31T31.control.cam.h0.' + month + '.nc'
dsc = xr.open_dataset(filename, decode_cf=False)
Sc = dsc['TS'][0,:,:]

path = 'GEO4962/jupyter-jeani/f2000.T31T31.sea_ice/atm/hist/'
filename = path + 'f2000.T31T31.sea_ice.cam.h0.' + month + '.nc'
dssi = xr.open_dataset(filename, decode_cf=False)
TSsi = dssi['TS'][0,:,:]

diff = TSc - TSsi

diff.psy.plot.mapplot(title="Surface temperature [K]\nF2000_CAM5_T31T31-0009-01\nControl-Sea_Ice")
</pre>

<img src="../fig/TS_F2000_CAM5_T31T31_control-sea_ice-0009-01.png">

Psyplot is a high level tool which offers a convenient means to easily and quickly create plots directly from the netCDF file. However for customized graphs and more advanced analyse one usually uses lower level python packages.

## Using python

Start a new **python3** notebook on your JupyterHub.

<font color="green">On jupyter:</font>

<pre>import xarray as xr
import numpy as np
import cartopy.crs as ccrs
from cartopy.util import add_cyclic_point
import matplotlib.pyplot as plt

%matplotlib inline

path = 'GEO4962/jupyter-jeani/f2000.T31T31.sea_ice/atm/hist/'
experiment = 'f2000.T31T31.sea_ice'
month = '0009-01'

filename = path + experiment + '.cam.h0.' + month + '.nc'

dset = xr.open_dataset(filename, decode_cf=False)
TSsi = dset['TS'][0,:,:]
lat = dset['lat'][:]
lon = dset['lon'][:]
dset.close()

TSmin = 200
TSmax = 350
TSrange = np.linspace(TSmin, TSmax, 16, endpoint=True)

TS_cyclic_si, lon_cyclic = add_cyclic_point(TSsi, coord=lon)

fig = plt.figure(figsize=[8, 8])
ax = plt.axes(projection=ccrs.Orthographic(central_longitude=20, central_latitude=40))
cs = ax.contourf(lon_cyclic, lat, TS_cyclic_si,
             transform=ccrs.PlateCarree(),
             levels=TSrange,
             extend='max',
             cmap='jet')
ax.set_title(experiment + '-' + month + '\n' + TSsi.long_name)
ax.coastlines()
ax.gridlines()

fig.colorbar(cs, shrink=0.8, label=TSsi.units)
</pre>
You can now use the command [savefig](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.savefig.html) to save the current figure into a file.

<font color="green">On jupyter:</font>

<pre>fig.savefig('Sea_ice-' + month)
</pre>

<img src="../fig/Sea_ice-0009-01.png">

(See 

{% include links.md %}
