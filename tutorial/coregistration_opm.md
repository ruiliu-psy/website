---
title: Coregistration of Optically Pumped Magnetometer (OPM) data
tags: [tutorial, opm, coordsys, polhemus]
---

# Coregistration of Optically Pumped Magnetometer (OPM) data

## Introduction

Conventionally, MEG is recorded with SQUIDs, which are superconductive sensors that require cryocooling and hence need to be placed as an fixed helmet-shaped array in a dewar. Optically Pumped Magnetometers (OPMs) are a new type of magnetic field sensors for MEG. OPMs do not require cryocooling and can be placed individually. Due to their small size and flexibility, different strategies are used to place OPM sensors on the head.

For a good interpretation of the MEG signals recorded with the OPM sensors over the head, it is important to coregister the OPM sensors (location and orientation) with the head. Also for source reconstruction it is required to coregister the sensors with the anatomical MRI, the volume conduction model, and the source model. Whereas SQUID-based MEG systems come with standard and coregistration procedures, for OPMs there is not a single standard.

Some labs use flexible caps to place the OPMs, similar to EEG and fNIRS caps. These flexible caps don't constrain the orientation of the sensors very well, which means that the sensor orientation relative to the head can change during the experiment depending on the head orientation and that the sensors can wobble due to movement. Since the magnetic field measured with MEG is a vector, the measurement is sensitive to these orientation differences.

To ensure a well-defined sensor placement, labs also often use helmets to position the OPM sensors relative to the head. These helmets can be designed and 3-D printed to fit optimally to the individual head, or can be designed as standard helmets to fit multiple participants (with different sized helmets for children and adults).

This tutorial demonstrates different methods for coregistering OPM sensors. Each method is demonstrated including data that you can download to carry out all steps yourself. Furthermore, it discusses the advantages and disadvantages of each method.

In this tutorial we will _not_ consider the coregistration of OPM sensors in flexible EEG-like caps. We will also _not_ discuss the coregistration of individually designed 3D printed helmets, as for those the coregistration is usually part of the design process and the helmet will fit only one way on the participant's head. Finally, this tutorial will also not cover the processing of the MEG signals recorded from the participants brain, this is covered in the [preprocessing_opm](/tutorial/preprocessing_opm) tutorial.

## Background

The common aim of the coregistration methods that we explore in this tutorial is to align geometrical objects - i.e. 'things' that have a position and orientation in 3D space - with respect to one another. Ultimately, we want know the location  of the OPM sensors relative to the participant's head and brain.

In the examples below, the OPM sensor positions and orientations are initially expressed in a coordinate system relative to the FieldLine smart helmet. To facilitate the downstream analysis of the MEG signals, e.g. for group level analysis, it is customary to aim for the OPM sensors expressed in a coordinate system that is defined based on anatomical landmarks on the participant's head.

{% include markup/info %}
Some basic background about coordinate systems, and the exact definition of some widely used coordinate systems is given in this [FAQ](/faq/coordsys).

In general you can think of coordinate systems in the same way as time zones. To make an online appointment with a colleague on the other side of the world, you have to align your respective agendas. The appointment subsequently appears in your agenda according to your timezone, and in their agenda in another timezone.

The process of aligning or coregistering means figuring out how much shift is needed to get the same object (or appointment) expressed in your respective coordinate systems (or time zone) to make sure that it corresponds on both sides. Once you know the shift, you can express the object (or appointment) in either coordinate system (or time zone).
{% include markup/end %}

### The dataset used in this tutorial

The data for this tutorial was recorded with a 32-sensor FieldLine HEDscan v3 system with a so-called smart helmet. It can be downloaded from our [download server](https://download.fieldtriptoolbox.org/tutorial/coregistration_opm/).

## Procedure

This tutorial describes different ways to achieve coregistration:

- Using geometric information from a Polhemus 3D tracker, matching two sets of points that are known to match one-to-one.
- Using head localization coils, where the location of the coils on the head is known, and the location of the coils relative to the sensors can be calculated.
- Using a 3D model from a camera-based 3D scanner of the head and helmet, and aligning this model with more detailed anatomical and sensor information.
- Using sensor-depth information from the FieldLine smart helmet as a proxy for the head surface.  
- Using a computer-designed custom helmet for a specific individual

Procedural outlines of each of the examples are provided in more detail below.

## Coregistration using a Polhemus tracker

{% include markup/warning %}
The Polhemus device consists of an electromagnetic transmitter (the large knob) and one or multiple receivers. When the OPMs are placed in the same magnetically shielded room (MSR) as a SQUID MEG system, the SQUID sensors can be disturbed by the rather strong electromagnetic fields. Depending on the sensitivity of the SQUID system and local procedures, the Polhemus-based method might therefore not be optimal or available.

Other 3D pointing devices such as the Optotrak (optical) and the Zebris (acoustical) might be more appropriate to localize the OPMs that are operated in the MSR room together with the SQUID MEG system.
{% include markup/end %}

The following example is based on a Polhemus recording, which - besides a measurement of some points on the participant's head surface - contains the digitized locations of 8 small indentations that serve as landmarks on the FieldLine smart helmet. These 8 fixed locations are also defined in the `fieldlinebeta2` template helmet, but there expressed in a different coordinate system.  

This part consists of the following steps:

- Read in the headshape and change the coordinate system, using **[ft_read_headshape](/reference/ft_read_headshape)** and **[ft_convert_coordsys](/reference/ft_convert_coordsys)**. For visualization we use **[ft_plot_headshape](/reference/ft_plot_headshape)** and **[ft_plot_axes](/reference/ft_plot_axes)**.
- Identification of the reference points in the Polhemus measurement
- Calculation of the transformation parameters, using **[ft_electroderealign](/reference/ft_electroderealign)**.
- Apply the transformation to the sensors, using **[ft_transform_geometry](/reference/ft_transform_geometry)**, and **[ft_plot_sens](/reference/ft_plot_sens)** for visualization.

### Read the Polhemus file and impose a head-based coordinate system

This specific Polhemus measurement has been obtained as a pilot at the DCCN with the reference sensor mounted on plastic security glasses. Since the security glasses were too bulky to fit under the MEG helmet, we secured them around the neck of the participant. A more appropriate procedure could have been implemented by using smaller security glasses that would fit under the OPM helmet, or by separating the reference sensor from the glasses and taping it straight onto the forehead of the participant.

The Polhemus recording was done using the CTF software. The software requires the experimenter to click the left ear, the right ear, and the nasion. It then expresses all subsequent points according to the CTF convention for the definition of the X/Y/Z axes of the coordinate system, which is anterior-left-superior (ALS).

For consistency with the other examples in this tutorial, we will first convert the head-based coordinate system to be right-anterior-superior (RAS).

    %% read in the data and enforce the units to be in 'mm'
    headshape = ft_read_headshape('example1_head_markers.pos');
    headshape = ft_convert_units(headshape, 'mm');

    %% visualization, coordinate axes are initially ALS
    figure
    ft_plot_headshape(headshape)
    ft_plot_axes(headshape)
    view([-27 20])

{% include image src="/assets/img/tutorial/coregistration_opm/headshape_upsideup_ctf.png" width="400" %}
_Figure: Polhemus recorded headshape with the coordinate axes according to the CTF-convention: the X-axis is pointing towards the nose._

    headshape.coordsys = 'ctf';
    headshape = ft_convert_coordsys(headshape, 'neuromag');  % this rotates it such that the X-axis points to the right
    
    %% visualization, coordinate axes are now RAS
    figure
    ft_plot_headshape(headshape)
    ft_plot_axes(headshape)
    view([114 20])

{% include image src="/assets/img/tutorial/coregistration_opm/headshape_upsideup.png" width="400" %}
_Figure: Adjusted headshape expressed in the RAS coordinate system._
    
### Identification of reference points

The Polhemus file not only describes the shape of the head and face, but also a number of reference points on the helmet. You can recognize them in the previous figure that you made, especially if you rotate it such that you see the head from the top. There are 8 points visible that have a clear distance from the head surface.

The reference points correspond to the last 8 points that were digitized. The order in which they were digitized was written down on a piece of paper: first left, from front-to-back, then right, from front-to-back. The points on the FieldLine helmet are indicated as A1-A8, and the order in which they were recorded is therefore 'A5', 'A6', 'A7', 'A8', 'A1', 'A2', 'A3', 'A4'. As we know where the points are in the Polhemus recording and in the helmet definition, we can calculate the transformation parameters to move these points from the helmet coordinate system to the coordinate system that is defined within the Polhemus measurement. The same transformation can then be applied to the OPM sensors.

    %% select the reference points on the helmets, with their corresponding label
    fid_measured = [];
    fid_measured.pos(1,:) = headshape.pos(end-7,:);
    fid_measured.pos(2,:) = headshape.pos(end-6,:);
    fid_measured.pos(3,:) = headshape.pos(end-5,:);
    fid_measured.pos(4,:) = headshape.pos(end-4,:);
    fid_measured.pos(5,:) = headshape.pos(end-3,:);
    fid_measured.pos(6,:) = headshape.pos(end-2,:);
    fid_measured.pos(7,:) = headshape.pos(end-1,:);
    fid_measured.pos(8,:) = headshape.pos(end-0,:);
    fid_measured.label = {'A5', 'A6', 'A7', 'A8', 'A1', 'A2', 'A3', 'A4'};
    
To perform a later comparison, it is convenient to sort them from 1 to 8.

    [fid_measured.label, indx] = sort(fid_measured.label);
    fid_measured.pos = fid_measured.pos(indx,:);

We can explicitly add the fiducials to the data structure that describes the head shape. The **[ft_plot_headshape](/reference/ft_plot_headshape)** function will in that case explocitly plot them, including their labels.

    headshape.fid = fid_measured

We also have the definition of the OPM sensor locations with the corresponding set of reference points for the FieldLine beta 2 helmet. The **[ft_plot_sens](/reference/ft_plot_sens)** function will also plot the reference points or fiducials.

    load fieldlinebeta2;
    fieldlinebeta2 = ft_convert_units(fieldlinebeta2, 'mm');
    fid_helmet     = fieldlinebeta2.fid;

    %% show the misalignment
    figure
    ft_plot_headshape(headshape)
    ft_plot_axes(headshape)
    ft_plot_sens(fieldlinebeta2)
    view([102 5]);

{% include image src="/assets/img/tutorial/coregistration_opm/coreg_polhemus_before.png" width="400" %}
_Figure: The reference points of the Polhemus measurement are not aligned with those of the OPM helmet._

We will proceed with **[ft_electroderealign](/reference/ft_electroderealign)**, which was originally implemented to align EEG electrode positions to a head surface. As it turns out, it can also be used more general to align two sets of points.

### Calculation of the transformation parameters

The alignment parameters can be estimated using the `template` method in **[ft_electroderealign](/reference/ft_electroderealign)**. Since we want to express the OPM sensors' coordinates in the head coordinate system, the Polhemus measured positions will be used as the target.

    %% estimate the alignment parameters
    cfg         = [];
    cfg.method  = 'template';
    cfg.target  = fid_measured;
    cfg.elec    = fid_helmet;
    fid_aligned = ft_electroderealign(cfg);

### Apply the transformation to the OPM sensors

The output data structure `fid_aligned` not only contains the aligned fiducials, but also the parameters that were used to align (or transform) them. We can apply the same transformation parameters to the OPM sensors.

    fieldlinebeta2_head = ft_transform_geometry(fid_aligned.rigidbody, fieldlinebeta2, 'rigidbody');

    figure
    ft_plot_headshape(headshape)
    ft_plot_axes(headshape)
    ft_plot_sens(fieldlinebeta2_head)
    view([102 5]);

{% include image src="/assets/img/tutorial/coregistration_opm/coreg_polhemus_after.png" width="400" %}
_Figure: OPM sensor locations are in register with the Polhemus headshape._

If you rotate the image, the first thing to notice is that the nose is properly pointing towards the opening of the helmet where the face should be. Furthermore, carefull inspection shows that there are now two sets of overlapping fiducials. Since we made sure previously that the fiducials are sorted from 1 to 8, we can compute the difference between the positions in the aligned helmet and the Polhemus measurement. If the overlap is not so great, it means that the Polhemus measurement was not so accurate.

    fieldlinebeta2_head.fid.pos - headshape.fid.pos
    ans =
        0.3611    0.2083   -0.0140
        4.8747    1.9590    1.6937
        1.9225    0.0960    1.0279
       -1.0412    2.4382   -0.8980
        0.6837   -2.4648   -2.2817
       -2.8585   -2.4431   -0.3389
       -2.0272   -1.7663    0.5568
       -1.9592    0.4899    0.2620
   
## Coregistration using head localizer coils

Conventional SQUID MEG systems are based on certain number of sensors (e.g., 275 or 306) that are placed in a fixed-size helmet to accommodate most participants. Unless when using [custom headcasts](Barnes paper), the SQUID MEG helmet gives the participant a few cm of space around the head. The heads of different participants will therefore not be in the same position relative to the helmet, for an individual participant the position of the head in the helmet will differ between sessions, and can even vary within a session. Conventional SQUID MEG systems therefore commonly use head localization or head position indicator (HPI) coils. The HPI coils are placed on the head - usually on well-defined [anatomical landmarks](/faq/how_are_the_lpa_and_rpa_points_defined) - and at the start of the recording session a small current is passed through the coils to create small magnetic dipoles. Sometimes the localization is repeated at the end of the recording session, and some systems also have the possibility to do the localization continuously. These magnetic dipoles can be localized, thereby determining the position of the sensors relative to the anatomical landmarks. All commercial SQUID MEG systems have a standard procedure for this that is well-integrated in the acquisition protocol and software, consequently the MEG recordings stored by the acquisition software include the sensor positions in [head coordinates](/faq/coordsys).

OPM sensors allow for individual placement and use variable-sized helmets. Furthermore, labs that operate an OPM MEG system will not all have the same number of sensors; some labs have as few as 8 sensors, whereas other labs might have up to 128 sensors.

{% include markup/warning %}
To localize the HPI coils you need sufficient coverage to obtain a good spatial distribution of the field of the magnetic dipole generated by the coil. Both the position (3 parameters) and the orientation (2 angles) need to be estimated. If the magnetic dipole _moment_ of the coil is not calibrated, also the strength needs to be estimated.

A minimum of 6 OPM channels is needed to estimate 6 magnetic dipole parameters, but a reliable estimation requires more channels. Coregistration using head localizer coils is therefore less suited for OPM systems with fewer channels.
{% include markup/end %}

The dataset used here contains 32 channels, with the OPM-sensors relatively uniformely distributed over the 144 slots of the FieldLine "beta 2" helmet. We did not perform the measurement on a real participant's head, but rather used the CTF magnetic phantom, which is basically a plexiglass mount to which the HPI coils can be attached at fixed (and known) locations. The phantom head was positioned quite high into the helmet, to maximize the coverage of the coils.

We will analyse an ~1 minute segment of data, during which the 3 HPI coils were energized with 3 sinusoidal signals at different frequencies: at 8 Hz for the 'nasion' coil, 11 Hz for the 'right ear' coil, and 14 Hz for the 'left ear' coil. These sinewave signals are orthogonal, i.e., uncorrelated in time. When the data is bandpass filtered, we can get the signal generated by each of the individual coils. Subsequently we can fit dipoles to the spatial topography of the first principal component of the bandpass-filtered data.

This part exists of the following steps:
- To evaluate the MEG signal and the spectrum, we start off with **[ft_preprocessing](/reference/ft_preprocessing)**, **[ft_databrowser](/reference/ft_databrowser)**, **[ft_selectdata](/reference/ft_selectdata)** and **[ft_freqanalysis](/reference/ft_freqanalysis)**.
- Processing of the data to get the contribution of each individual HPI coil, using **[ft_preprocessing](/reference/ft_preprocessing)**, .
- Fitting of dipoles to the topographies of the first principal components of the bandpass filtered data, using **[ft_componentanalysis](/reference/ft_componentanalysis)**, and **[ft_dipolefitting](/reference/ft_dipolefitting)**. For visualization of the spatial topographies, we use **[ft_topoplotIC](/reference/ft_topoplotic)**, and for the dipole fit we start with a grid search, and we use **[ft_prepare_sourcemodel](/reference/ft_prepare_sourcemodel)** to create the search grid.  
- Calculation of the transformation matrix that moves the sensors to the head-based coordinate system, using **[ft_headcoordinates](/reference/ft_headcoordinates)**.
- Apply the transformation matrix to the sensors, using **[ft_transform_geometry](/reference/ft_transform_geometry)**.

### Processing of the data to capture the signal of the HPI coils

    % load in the data
    cfg              = [];
    cfg.dataset      = 'example2_magneticphantom_HPIplusdipoleset6_raw.fif';
    cfg.coilaccuracy = 0;
    data_all         = ft_preprocessing(cfg);

We can visualize the data with **[ft_databrowser](/reference/ft_databrowser)** to see where the sine-wave signals start and end.

    cfg = [];
    cfg.viewmode = 'vertical';
    cfg.blocksize = 300; % seconds
    ft_databrowser(cfg, data_all);

You can recognize four blocks of about 60 seconds each, followed by a final block of about 60 seconds in which you don't see much. In the first block all three coils were energized at the same time, at different (orthogonal) frequencies. That is the part of the data that we will use. A bit later in the recording each of the coils was energized individually; those pieces of data would have been usefull if all coils would have been driven with the same frequency. In the last block the magnetic dipole that is at the center of the CTF phantom was energized at 11 Hz; the signal there is much weaker since the coil is further away from the OPM sensors.

{% include markup/info %}
There are two channels which have the wrong position/orientation information in the specific example data used here. We won't elaborate on it but simply remove those channels from further processing.
{% include markup/end %}

We cut out the relevant time segment using **[ft_selectdata](/reference/ft_selectdata)**. After this step the data is exactly 300000 samples long (60 seconds times 5000 Hz).

    % this is the time of a single sample
    tsample = 1./data_all.fsample

    cfg         = [];
    cfg.latency = [0 60-tsample];
    cfg.channel = {'all' '-L212_bz' '-R212_bz'};     
    data        = ft_selectdata(cfg, data_all);
    
We cut the data into 10-second segments with 80% overlap and compute the averaged powerspectrum over all segments to verify the expected spectral peaks (and their harmonics) at 8, 11 and 14 Hz.

    cfg            = [];
    cfg.length     = 10;
    cfg.overlap    = 0.8;
    data_segmented = ft_redefinetrial(cfg, data)

    cfg           = [];
    cfg.method    = 'mtmfft';
    cfg.foilim    = [0 40];
    cfg.taper     = 'dpss';
    cfg.tapsmofrq = 0.2;
    cfg.pad       = 10;
    freq          = ft_freqanalysis(cfg, data_segmented);

    figure; plot(freq.freq, log10(mean(freq.powspctrm)));

{% include image src="/assets/img/tutorial/coregistration_opm/powerspectrum_hpi.png" width="400" %}
_Figure: Powerspectrum from a measurement containing strong signals at 8, 11 and 14 Hz, and harmonics._

To focus on the signals of the specific HPI-coils, we bandpass filter the data in the frequency bands corresponding to each of the coils, and cut off the edges for any potential filter edge artifacts.

    cfg            = [];
    cfg.bpfilter   = 'yes';
    cfg.bpfilttype = 'firws';
    cfg.usefftfilt = 'yes';
    cfg.bpfreq     = [7 9];
    data08         = ft_preprocessing(cfg, data); % nas

    cfg.bpfreq     = [10 12];
    data11         = ft_preprocessing(cfg, data); % rpa

    cfg.bpfreq     = [13 15];
    data14         = ft_preprocessing(cfg, data); % lpa
    
    cfg            = [];
    cfg.latency    = [4 56-1./data.fsample];
    data08         = ft_selectdata(cfg, data08);
    data11         = ft_selectdata(cfg, data11);
    data14         = ft_selectdata(cfg, data14);
    
    %% look at 2 seconds of the data
    figure;
    plot(data08.time{1}, data08.trial{1});
    xlim([4 6]);
    xlabel('time (s)');
    ylabel('magnetic field strength (T)');

{% include image src="/assets/img/tutorial/coregistration_opm/hpi_8hz.png" width="400" %}
_Figure: Two seconds of data, bandpass filtered around 8 Hz._

### Fit dipoles to the sensor topographies

We proceed by performing a principal component analysis (PCA) on the filtered data. The idea is that - given that the signals from the HPI coils are the strongest signals in the measurement, and given that we have bandpass filtered the data - the strongest principal components will represent  the 'spatial fingerprints' of each of the HPI coils. Those fingerprints will be used to perform a dipole fit, i.e., find the position of a dipole that optimally explain those principal components.

    cfg            = [];
    cfg.method     = 'pca';
    cfg.updatesens = 'no';
    comp08 = ft_componentanalysis(cfg, data08);
    comp11 = ft_componentanalysis(cfg, data11);
    comp14 = ft_componentanalysis(cfg, data14);

    %% look at the topographies
    cfg              = [];
    cfg.component    = 1;
    cfg.layout       = 'fieldlinebeta2bz_helmet.mat';
    cfg.gridscale    = 150;
    cfg.interplimits = 'sensors';
    cfg.figure       = subplot('position',[0 0 1/3 1]);
    ft_topoplotIC(cfg, comp08);
    cfg.figure       = subplot('position',[1/3 0 1/3 1]);
    ft_topoplotIC(cfg, comp11);
    cfg.figure       = subplot('position',[2/3 0 1/3 1]);
    ft_topoplotIC(cfg, comp14);

{% include image src="/assets/img/tutorial/coregistration_opm/hpi_topo.png" width="700" %}
_Figure: Spatial topographies of the signals generated by the 3 HPI coils._

For the fitting the magnetic dipole positions, we will use a grid search as an initial scan over the whole volume, followed by a iterative non-linear optimization. The grid search is motivated by the fact that a non-linear search of the whole parameter space (i.e., volume of space covered by the helmet) might result in convergence to a local minimum.

The following creates a sourcemodel that consists of a regular grid of dipole positions that will be used for the initial grid search.

    %% create a regular grid of dipole positions bounded by the helmet
    load fieldlinebeta2

    % make a fake headshape, we use this to make a fake headmodel
    fake_headshape      = [];
    fake_headshape.pos  = fieldlinebeta2.coilpos;
    fake_headshape.unit = 'm';

    % create a fake singleshell headmodel, this will act as the boundary for the grid
    cfg = [];
    cfg.method = 'singleshell';
    cfg.headshape = fake_headshape;
    cfg.numvertices = 144; % keep the same number of vertices as OPMs
    fake_headmodel = ft_prepare_headmodel(cfg);

    %% create the grid, grid points outside the fake head will be flagged as such
    cfg            = [];
    cfg.headmodel  = fake_headmodel;
    cfg.resolution = 0.0075;
    sourcemodel    = ft_prepare_sourcemodel(cfg);

    % this is the real volume conductor model that we want to use
    cfg = [];
    cfg.method = 'infinite';
    headmodel = ft_prepare_headmodel(cfg);

Now we can perform the dipole fits.

    cfg             = [];
    cfg.headmodel   = headmodel;
    cfg.grad        = fieldlinebeta2;
    cfg.component   = 1;
    cfg.gridsearch  = 'yes';
    cfg.sourcemodel = sourcemodel;
    dip08 = ft_dipolefitting(cfg, comp08);
    dip11 = ft_dipolefitting(cfg, comp11);
    dip14 = ft_dipolefitting(cfg, comp14);

The data in this example was obtained with the HPI coils placed on well-defined locations on the CTF magnetic phantom. The phantom, when seen from above, has the Nasion coil located at 12:00 o'clock and LPA and RPA at 9:00 and 3:00 o'clock. The radius of the phantom is 7.5 cm. Also accounting for the thickness of the coils, the expected distances between the localizer coils on the phantom are as follows:

- Left-Right: 15.32 cm
- Nasion-Left and Nasion-Right: 10.83 cm

We can verify the reconstructed distances as follows. Note that for a real measurement, one would need to measure the distance between the fiducials first (e.g. with a Polhemus scanner, see above) in order to be able to make such a comparison.

    % for verification
    disp(norm(dip11.dip.pos - dip14.dip.pos)*100) % in cm
    disp(norm(dip08.dip.pos - dip11.dip.pos)*100)
    disp(norm(dip08.dip.pos - dip14.dip.pos)*100)
    
### Definition of the head-based coordinate system

Now that we have identified the HPI coil locations, we can compute the coregistration matrix that transforms the HPI coil positions from 'helmet' or sensor coordinates to 'head' coordinates. Here, we use the neuromag convention.

    % transform fiducial coordinates to head coordinates (RAS)
    fid1 = dip08.dip.pos; % nas
    fid2 = dip14.dip.pos; % lpa
    fid3 = dip11.dip.pos; % rpa
    transform_sens2head = ft_headcoordinates(fid1, fid2, fid3, 'neuromag');

### Apply the transformation matrix to the sensors

Converting the actual OPM positions from 'helmet' or sensor coordinates to 'head' coordinates is done using the **[ft_transform_geometry](/reference/utilities/ft_transform_geometry)** function.

    fieldlinebeta2_head = ft_transform_geometry(transform_sens2head, fieldlinebeta2);

## Coregistration using a 3D scanner

{% include markup/warning %}
Note that if you are using 3D scanner based on an iPhone or iPad, such as the [Structure Sensor](https://structure.io), and if you have the OPMs in the same magnetically shielded room (MSR) as a SQUID MEG system, you will want to turn the iPhone or iPad to airplane mode prior to taking it into the MSR. Otherwise the electromagnetic fields of the cellular and/or wifi radio may cause problems with the SQUIDs.
{% include markup/end %}

The idea here is to make a sufficiently high quality 3D-model that captures the participant's facial features in register with the the helmet, such that the facial features can be used for coregistration with a surface image obtained from an anatomical MRI. From the image of the helmet, the position of the sensors can be deduced. Thus, the 3D-model serves as an intermediary to link the anatomy with the sensors.

This part consists of the following steps:
- Read the anatomical MRI, and assign a head-based coordinate system, using **[ft_read_mri](/reference/ft_read_mri)**, and **[ft_volumerealign](/reference/ft_volumerealign)**.
- Read in the 3D model, assign a meaningful coordinate system, and erase the irrelevant parts, using **[ft_read_headshape](/reference/ft_read_headshape)**, **[ft_meshrealign](/reference/ft_meshrealign)**, and **[ft_defacemesh](/reference/ft_defacemesh)**.
- Interactive alignment of the face - extracted from the 3D model -  with the MRI-extracted scalp surface, using **[ft_volumesegment](/reference/ft_volumesegment)**, **[ft_prepare_mesh](/reference/ft_prepare_mesh)**, and **[ft_meshrealign](/reference/ft_meshrealign)**.
- Interactive alignment of the helmet with the reference sensors/helmet, using **[ft_meshrealign](/reference/ft_meshrealign)**.
- Combination of the obtained alignment parameters into a single transformation matrix
- Application of the resulting transformation to the sensor array, using **[ft_transform_geometry](/reference/ft_transform_geometry)**.

### Definition of the head-based coordinate system

Here, we read in the anatomical MRI of the participant, and define the coordinate system based on the conventions that are typically used for SQUID-based MEG. That is, using anatomical landmarks on the surface of the head, a coordinate system is defined, that can also be defined using a Polhemus tracker (as in coregistration strategy 1, see above). Note that alternatively the coordinate system can be defined based on anatomical structures in the brain (cf. the anterior and posterior commissures), which facilitates spatial alignment/normalisation across participants.

    % read in the anatomical MRI
    mri   = ft_read_mri('example3_anatomical.nii.gz');
    ft_determine_coordsys(mri);

{% include image src="/assets/img/tutorial/coregistration_opm/mri_notaligned.png" width="400" %}
_Figure: anatomical MRI image with an not clearly defined coordinate system._

After reading in the MRI, you can check the coordinate system with `ft_determine_coordsys`. As the above figure shows, the axes are labeled as 'unknown', but it seems that they are oriented according to the RAS convention, while the origin of the coordinate system is ill-defined. For this reason, we will explicitly impose an anatomical landmark based coordinate system next, which requires interactive identification of the relevant landmarks (nasion, left/right pre auricular points).

    % define a head based coordinate system
    cfg          = [];
    cfg.coordsys = 'neuromag';
    mri          = ft_volumerealign(cfg, mri);
    ft_determine_coordsys(mri);

{% include image src="/assets/img/tutorial/coregistration_opm/mri_aligned.png" width="400" %}
_Figure: anatomical MRI image with a 'neuromag' coordinate system._

### Cleaning of the structure sensor scan

Here, we read in the 3D-model from the structure scan, and define a coordinate system that has its axes pointing into more or less canonical directions (relative to the participant). The next steps remove irrelevant parts of the image (e.g. the back of a chair etc), and here we also choose to separate the 'face' part from the 'helmet' part, to facilitate the alignment. Strictly speaking this separation is not necessary.

    scan      = ft_read_headshape('example3_face_helmet.obj');
    scan.unit = 'm';
    
    figure;hold on;
    ft_plot_headshape(scan);
    ft_plot_axes(scan);
    lighting gouraud; material dull; h=light;

{% include image src="/assets/img/tutorial/coregistration_opm/scan_notaligned.png" width="400" %}
_Figure: 3D-model with an not clearly defined coordinate system._

In the example model, the coordinate axes' orientations relative to the participant more or less are well-behaved, i.e. the axes are pointing approximately along the left/right, anterior/posterior, and superior inferior directions, but the order of the axes is not conventional. As a first step we might want to assign a better defined coordinate system to the model. Note that the exact coordinate system does not matter too much. Here we define the coordinate system such that the X/Y/Z axes are pointing into the same direction as the head  coordinate system defined in the MRI image, i.e. R(ight)A(nterior)S(uperior). We use `cfg.coordsys='neuromag'` because this method allows us to approximately indicate the N(asion)/L(eft preauricular point), and R(ight) preauricular point. Note that in the below procedure, the ears are not visible in the model, instead we will use the protruding points on the helmet's rim to define 'l' and 'r'.

    % approximately align the mesh to a RAS coordinate system,
    % by clicking on 'dummy' nas/lpa/rpa
    cfg          = [];
    cfg.method   = 'fiducial';
    cfg.coordsys = 'neuromag';
    scan         = ft_meshrealign(cfg, scan);
    
    figure;hold on;
    ft_plot_headshape(scan);
    ft_plot_axes(scan);
    view([125 10]);
    lighting gouraud; material dull; h=light;

{% include image src="/assets/img/tutorial/coregistration_opm/scan_sosoaligned.png" width="400" %}
_Figure: 3D-model with a coordinate system relating to the head and helmet._

In the example model, a large part of the body of the participant is also present, we remove it, in order to facilitate the alignment. The below code uses `ft_defacemesh` with `cfg.method='plane'`. This particular method throws away data points that are on one side of the plane, which is indicated by the direction of the stick that is sticking out from the middle of the plane. Here, good results were obtained, by setting the viewpoint in the interactive window to 'right', and then using the following numbers to define the cutting plane: rotate [-40 0 0], translate [0 0 -140]. Note that the viewpoint does not have a consequence for the points to be excluded.

    % cut off the irrelevant parts, this might require a few iterations
    cfg        = [];
    cfg.method = 'plane';
    scan         = ft_defacemesh(cfg, scan);
    
    figure;hold on;
    ft_plot_headshape(scan);
    ft_plot_axes(scan);
    view([125 10]);
    lighting gouraud; material dull; h=light;

{% include image src="/assets/img/tutorial/coregistration_opm/scan_facehelmet.png" width="400" %}
_Figure: 3D-model with only the face and helmet._

In the following, we separate the 'helmet' part of the model from the 'face' part of the model, because this facilitates the alignment performed below. Note that we need to ensure that any change in the coordinates of one of these objects should be reflected in the other object as well. Clearly, this is needed because this model of the two objects is the crucial link that links the anatomy with the sensors.

    % separate the face from the helmet, it's easier to keep the face at first
    % instance, and then go back to the original mesh to get the helmet
    scan_face    = scan;
    scan_face    = ft_defacemesh(cfg, scan_face); % viewpoint: front, rotate: [0 -90 0], translate: [85 0 20].
    scan_face    = ft_defacemesh(cfg, scan_face); % viewpoint: front, rotate: [0  90 0], translate: [-67 0 20].
    scan_face    = ft_defacemesh(cfg, scan_face); % viewpoint: left,  rotate: [140 0 0], translate: [0 0 55];
    
    figure;hold on;
    ft_plot_headshape(scan_face);
    ft_plot_axes(scan_face);
    view([125 10]);
    lighting gouraud; material dull; h=light;    
    
{% include image src="/assets/img/tutorial/coregistration_opm/scan_face.png" width="400" %}
_Figure: 3D-model with only the face._

The helmet mesh will be extracted by removing the face vertices from the model. This is a bit less straightforward with normal FieldTrip functions, and we need to do a little bit of coding, and use a function which is located in a private folder.

    % get the helmet by excluding the face nodes from the model
    scan_helmet  = scan;
    
    % this requires a temporary change into a private folder
    [ftver, ftdir] = ft_version;
    pdir = fullfile(ftdir, 'private');
    cd(pdir);
    
    % the intersect(a,b,'rows') does not give the full intersection because of duplicate points
    pos1 = scan_face.pos;
    pos2 = scan_helmet.pos;
    mindist = nan(size(pos1,1),1);
    indx    = nan(size(pos1,1),1);
    sel     = cell(size(pos1,1),1);
    for k = 1:size(pos1,1)
      delta = sqrt(sum((pos2 - pos1(k,:)).^2,2));
      [mindist(k), indx(k)] = min(delta);
      sel{k} = find(delta==mindist(k));
    end
    sel = unique(cat(1,sel{:}));
    [scan_helmet.pos, scan_helmet.tri] = remove_vertices(scan_helmet.pos, scan_helmet.tri, sel);

    figure;hold on;
    ft_plot_headshape(scan_helmet);
    ft_plot_axes(scan_helmet);
    view([125 10]);
    lighting gouraud; material dull; h=light;

{% include image src="/assets/img/tutorial/coregistration_opm/scan_helmet.png" width="400" %}
_Figure: 3D-model with only the helmet._
    
### Interactive alignment of the face with the MRI-based scalp surface

Now we will extract the scalp surface from the anatomical MRI, and interactively align this with the face extracted from the 3D model. In theory, an automatic algorithm, such as the iterative closest point (ICP) algorithm could be used, but the quality of the results highly depends on the number of points, and the quality of the initial alignment. In this example, we stick to a manual alignment. To achieve a reasonably good alignment, the following values can be specified for the rotation (without clicking the 'apply' button in between): [13.5 0.2 -4], and for the translation: [-0.003 0.0825 -0.0083]. (Note that the metric units are now expressed in 'm'.

    % segment the scalp
    cfg          = [];
    cfg.output   = 'scalp';
    seg          = ft_volumesegment(cfg, mri);
    
    % create a mesh for the scalp
    cfg             = [];
    cfg.tissue      = 'scalp';
    cfg.numvertices = 10000;
    mri_face        = ft_prepare_mesh(cfg, seg);
    mri_face        = ft_convert_units(mri_face, 'm');
    
    % align the 3D model's face to the face extracted from the anatomical image
    cfg             = [];
    cfg.headshape   = mri_face;
    cfg.meshstyle   = {'edgecolor','k','facecolor','skin'};
    scan_face_aligned = ft_meshrealign(cfg, scan_face);

    figure;hold on;
    ft_plot_headshape(mri_face,'facealpha',0.4);
    ft_plot_mesh(scan_face_aligned, 'facecolor','skin');    
    view([125 10]);
    lighting gouraud; material dull; h = light;

{% include image src="/assets/img/tutorial/coregistration_opm/face_aligned.png" width="400" %}
_Figure: 3D-model of the face aligned with MRI-derived face surface._

### Interactive alignment of the helmet with the reference sensors/helmet

We can also attempt to interactively align the helmet model with the template helmet. Here, we could use the following values for the rotation: [24 0 -3], and  for the translation: [-0.009 0.07 -0.05].

    load fieldlinebeta2
    fieldlinebeta2.coordsys = 'ras';

    cfg = [];
    cfg.grad = fieldlinebeta2;
    cfg.meshstyle = {'edgecolor','k','facecolor','skin'};
    scan_helmet_aligned = ft_meshrealign(cfg, scan_helmet);

    figure;hold on;
    ft_plot_sens(fieldlinebeta2);
    ft_plot_mesh(scan_helmet_aligned, 'facecolor', [0.5 0.5 1], 'facealpha', 0.4, 'edgecolor', 'none');
    view([125 10]);
    lighting gouraud; material dull; h = light;

{% include image src="/assets/img/tutorial/coregistration_opm/helmet_aligned.png" width="400" %}
_Figure: 3D-model of the helmet aligned with FieldLine helmet._

### Calculation of the transformation matrix

Now we can use the transformations that align the 3D scan's face with the MRI-derived facial surface, and that align the 3D scan's helmet with the sensor positions to calculate the transformation that aligns the sensors with the anatomy.

    transform_scan2helmet = scan_helmet_aligned.cfg.transform;
    transform_scan2face   = scan_face_aligned.cfg.transform;
    transform_helmet2face = transform_scan2face/transform_scan2helmet;
    
### Apply the transformation matrix to the sensors

The transformation matrix `transform_helmet2face` can now be used to update the sensor definition, which aligns the sensors with head-based coordinate system .

    % align the sensors to the head
    fieldlinebeta2_head = ft_transform_geometry(transform_helmet2face, fieldlinebeta2);
    
    figure;hold on;
    ft_plot_sens(fieldlinebeta2_head);
    ft_plot_headshape(mri_face, 'facecolor', [0.5 0.5 1], 'facealpha', 0.4, 'edgecolor', 'none');
    view([125 10]);
    lighting gouraud; material dull; h = light;

{% include image src="/assets/img/tutorial/coregistration_opm/sensors_face_aligned.png" width="400" %}
_Figure: sensors aligned with the anatomical MRI._

Note that in this particular example, the participant was not positioned very high in the FieldLine helmet.


## Coregistration using the sensors following the head shape

_This is specific for the FieldLine smart helmet._

More info to follow: stay tuned!


## Coregistration of a computer-designed individual custom helmet

More info to follow: stay tuned!


## Summary and suggested further reading

This tutorial gave an introduction on the coregistration of OPM data, specifically dealing with OPM data that has been collected with the sensors positioned in in a known helmet configuration.  

You may want to continue with the more general [tutorials](/tutorial/) on processing MEG (and EEG) data, or have a look at the [system specific details](/getting_started) for the OPM data that you are working with. Also, you may want to proceed with the [opm preprocessing tutorial](/tutorial/preprocessing_opm).

Furthermore, you can explore other pages that deal with OPMs:

{% include seealso tag1="opm" %}