Specutils Specifications
------------------------

author: S. M. Crawford, N. Earl, A. Ginsburg, E. Tollerud

date-created: 2017 April 21 <replace with the date you submit the APE>

date-last-revised: 2017 March 7 <keep this up to date anytime something changes>

type:  Standard Track

status: Discussion


Abstract
--------

Spectroscopy lies at the core of much of astronomy.  It is the goal of this APE to  coordinate and plan the development of spectroscopic software.   A number of tools have already been developed within astropy as well in individual packages, and in this APE, we propose the specifications for a Spectrum1D object, and outline a list of tools that are needed by the community.  

Detailed description
--------------------

Spectroscopy in astronomy

Spectrum 1D in specutils

problems: already has been forked into Spectrum1DRef, not widely adopted, no longer maintained

issues: order of arguments, reversible wcs/dispersion, using astropy.io 

Need for common analysis tools

Functionality we want in general spectroscopic tools (specutils, specreduce, or specviz)

- detecting lines
- measuring lines
  - fitting models to lines
  - measuring equivalent widhts
  - centroid of lines
- redshift measurementions
- heliocentric corrections
- 

Iraf onedspec package:
        aidpars - Automatic line identification parameters and algorithm
   autoidentify - Automatically identify lines and fit dispersion
          bplot - Batch plots of spectra
      calibrate - Apply extinction and flux calibrations to spectra
      continuum - Fit the continuum in spectra
       deredden - Apply interstellar extinction correction
        dispcor - Dispersion correct and resample spectra
      disptrans - Transform dispersion units and apply air correction
         dopcor - Apply doppler corrections
       fitprofs - Fit gaussian profiles
       identify - Identify features in spectrum for dispersion solution
         lcalib - List calibration file data
         mkspec - Generate an artificial spectrum (obsolete)
          names - Generate a list of image names from a string
         ndprep - Make neutral density filter calibration image
      odcombine - Combine spectra having different wavelength ranges (new)
     refspectra - Assign wavelength reference spectra to other spectra
     reidentify - Automatically identify features in spectra
      rspectext - Convert ascii text spectra to image spectra
     sapertures - Set or change aperture header information
         sarith - Spectrum arithmetic
         sbands - Bandpass spectrophotometry of spectra
       scombine - Combine spectra having different wavelength ranges
        scoords - Set spectral coordinates as a pixel array (1D spectra only)
          scopy - Select and copy apertures in different spectral formats
       sensfunc - Create sensitivity function
     setairmass - Compute effective airmass and middle UT for an exposure
          setjd - Compute and set Julian dates in images
           sfit - Fit spectra and output fit, ratio, or difference
          sflip - Flip data and/or dispersion coordinates in spectra
        sinterp - Interpolate a table of x,y pairs to create a spectrum
       skytweak - Sky subtract 1D spectra after tweaking sky spectra
          slist - List spectrum header parameters
       specplot - Stack and plot multiple spectra
      specshift - Shift spectral dispersion coordinate systems
          splot - Preliminary spectral plot/analysis
       standard - Identify standard stars to be used in sensitivity calc
       telluric - Remove telluric features from 1D spectra
      wspectext - Convert 1D image spectra to ascii text spectra


Branches and pull requests
--------------------------

Any pull requests or development branches containing work on this APE should be
linked to from here.  (An APE does not need to be implemented in a single pull
request if it makes sense to implement it in discrete phases). If no code is yet
implemented, just put "N/A"


Implementation
--------------

This section lists the major steps required to implement the APE.  Where
possible, it should be noted where one step is dependent on another, and which
steps may be optionally omitted.  Where it makes sense, each  step should
include a link related pull requests as the implementation progresses.


Backward compatibility
----------------------

This section describes the ways in which the APE breaks backward compatibility.


Alternatives
------------

If there were any alternative solutions to solving the same problem, they should
be discussed here, along with a justification for the chosen approach.


Decision rationale
------------------

<To be filled in by the coordinating committee when the APE is accepted or rejected>
