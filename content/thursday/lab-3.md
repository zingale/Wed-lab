---
title: "Lab 3: Stable relationships"
author: Annachiara Picco (lead), Matthias Fabry, Lucas de Sá, Lieke Van Son
weight: 4
math: true
toc: true
---

<div style="text-align: justify;">

<!-- # Lab 3: Stable relationships -->


<span style="color: #e7876c;">Timing: approximately 2 hours divided into 2 x 1 hour blocks </span>


## Introduction

In this last minilab3 we will pick up on the system you evolved in minilab1 and follow its further evolution into a double black hole binary. Remember that at the end of your minilab1 you had two systems: 
- One system underwent mass transfer during the Main Sequence (Case A mass transfer), with final properties listed in the below [Table 1](#table-binary).
- The other underwent mass transfer after the Main Sequence (Case B mass transfer), with final properties as in [Table 2](#table-caseB).

<div style="display: flex; justify-content: center; margin-bottom: 2rem; margin-top: 2rem;">
<table id="table-binary" style="margin:auto; text-align:center;">
  <tr>
    <th></th>
    <th>Primary (Stripped star)</th>
    <th>Secondary (Accretor)</th>
  </tr>
  <tr>
    <td>Mass</td>
    <td>16.8 M☉</td>
    <td>39.6 M☉</td>
  </tr>
  <tr>
    <td>Ω / Ωcrit</td>
    <td>0.02</td>
    <td>0.15</td>
  </tr>
  <tr style="background: rgba(246, 171, 59, 0.22);">
    <td>Orbital Period</td>
    <td colspan="2" style="text-align:center;">4.5 days</td>
  </tr>
  <tr>
    <td>Mass ratio</td>
    <td colspan="2" style="text-align:center;">0.42</td>
  <tr>
  <td>Final model</td>
  <td>
    <a href="../lab3/final1_caseA.mod" download>
      <code>final1_caseA.mod</code>
    </a>
  </td>
  <td>
    <a href="../lab3/final2_caseA.mod" download>
      <code>final2_caseA.mod</code>
    </a>
  </td>
</tr>
<caption><strong>Table 1:</strong> CASE A binary at the end of minilab1.</caption>
</table>
</div>

<div style="display: flex; justify-content: center;">
<table id="table-caseB" style="margin:auto; text-align:center;">
  <tr>
    <th></th>
    <th>Primary (Stripped star)</th>
    <th>Secondary (Accretor)</th>
  </tr>
  <tr>
    <td>Mass</td>
    <td>17.14 M☉</td>
    <td>40.8 M☉</td>
  </tr>
  <tr>
    <td>Ω / Ωcrit</td>
    <td>?</td>
    <td>?</td>
  </tr>
  <tr style="background: rgba(246, 171, 59, 0.22);">
    <td>Orbital Period</td>
    <td colspan="2" style="text-align:center;">32.2 days</td>
  </tr>
  <tr>
    <td>Mass ratio</td>
    <td colspan="2" style="text-align:center;">0.42</td>
  <tr>
  <td>Final model</td>
  <td>
    <a href="../lab3/final1_caseB.mod" download>
      <code>final1_caseB.mod</code>
    </a>
  </td>
  <td>
    <a href="../lab3/final2_caseB.mod" download>
      <code>final2_caseB.mod</code>
    </a>
  </td>
</tr>
<caption><strong>Table 2:</strong> CASE B binary at the end of minilab1.</caption>
</table>
</div>

Both these systems have a rapidly rotating (spun-up) secondary that has accreted a lot of mass from the primary; it is, therefore, "rejuvenated" (fresh fuel has prolonged its Main Sequence lifetime), and is happily burning hydrogen (H) in its core. The primary, on the other hand, has already depleted helium (He) in its core and has been "stripped" off its H-rich envelope. 

> [!Important]
> Notice that the Case A system has a much lower orbital period than the Case B. These different post-mass transfer properties determine a different further evolutionary history!
> In this minilab3, we will understand **how both types of binaries need to evolve in order to form gravitational wave sources** such as merging binary black holes.
> 
> **SPOILER**: both systems will evolve with a further stage of mass transfer, which may be 
> - <u>STABLE</u>: relatively long-lived (on the thermal or even nuclear timescale of the donor) and self regulated, with a quiet detachment afterwards. We will study this in [Section 1](#1-stable-mass-transfer).
> - <u>UNSTABLE</u>: or "Common envelope", a fast (~dynamical timescale) stage in which the envelope of the donor engulfs the binary. We will study this in [Section 2](#2-common-envelope-evolution).


## 1. Stable mass transfer

Let's start by copying the standard work directory from ```$MESA_DIR``` to your preferred local folder:

```bash
cp -r $MESA_DIR/binary/work stable_MT
cd stable_MT
```

Inspect your ```inlist_project``` and pay attention to the following two controls:

- **`evolve_both_stars`** When false, MESA will model just one star, and keep the other as a point mass, while solving for the full orbital evolution of the binary. This is convenient for binaries with compact objects, like black holes (BH). You can find more info in here: `$MESA_DIR/binary/defaults/binary_job.defaults`

- **`limit_retention_by_mdot_edd`** When true, the accretion rate of material onto the point-mass companion will be capped to a physical limit, called "Eddington limit". This is because the infall of hot stellar material onto a compact object creates radiation pressure that may halt the accretion process. In this case, the excess transferred mass is ejected from the binary with the specific angular momentum of the point mass. You can find more info in here: `$MESA_DIR/binary/defaults/binary_controls.defaults`

As you can see, our simulation is already set up to model one of the components as a point mass with Eddington limited accretion. Ideal starting point for us to model a binary with a BH.

### Building the setup

Now we will perform some adjustments to the template. A significant number of these are meant to make the simulation faster in order to be efficiently computed in the duration of this lab.

#### Modify `history_columns.list` 
Grab the `history_columns.list` file from `$MESA_DIR/star/defaults` and copy it into your work folder:

```bash
cp -r $MESA_DIR/star/defaults/history_columns.list .
```

We will add an option to this file to visualize the quantities in the Kippenhahn diagram in the `pgstar` window. Search through this file for the string `mixing_regions`. Add below:

```fortran
mixing_regions 10

```

#### Modify `binary_controls` in `inlist_project`

By default MESA also includes the effect of magnetic braking for angular momentum loss. This implementation is meant for late type stars and should be removed when working with massive binaries. Additionally, by default MESA reduces the growth of the BH mass to account for the rest-mass energy radiated away during accretion, determined by a radiative efficiency parameter. For simplicity, in this lab we will switch this control off. For these purposes, you can include:

```fortran
! be 100% sure magnetic braking is always off
do_jdot_mb = .false.
! don't reduce the BH accretion efficiency
use_radiation_corrected_transfer_rate = .false.
```

To run the simulation faster we will relax multiple timestepping controls of the binary module by including:

```fortran
! relax timestep controls
fm = 0.1d0
fa = 0.02d0
fa_hard = 0.04d0
fr = 0.5d0
fj = 0.01d0
fj_hard = 0.02d0
```
The exact purpose of each of these controls can be checked in the defaults file `$MESA_DIR/binary/defaults/binary_controls.defaults`. Contrary to the `star` module, there is not a single `time_delta_coeff` control to easily scale all timesteps, so each of these controls relax the solver conditions based on the binary properties (orbital separation, Roche Lobe radius, ecc.).

Finally, let's add some options for the output of our simulation:
```fortran
! output preferences
photo_interval = 50
history_interval = 1
```

#### Modify `controls` in `inlist1`
We will add a lot of options here that involve the individual stripped star model. Most of them are purely to make the run faster; we add also a prescription for the convective overshooting and a terminating condition at core-Helium depletion. Going through everything is beyond the scope of this lab (we are focusing on binaries here 🙃 and you have seen many of these already in the previous days), but feel free to dig into them later if you have time! Remember that the exact purpose of each of these controls can be checked in the defaults file `$MESA_DIR/star/defaults/controls.defaults`.

{{< details title="Modified `controls` for `inlist1`" closed="true" >}}
```fortran
&controls

   extra_terminal_output_file = 'log1' 
   log_directory = 'LOGS1'

   ! we use step overshooting
   overshoot_scheme(1) = 'step'
   overshoot_zone_type(1) = 'burn_H'
   overshoot_zone_loc(1) = 'core'
   overshoot_bdy_loc(1) = 'top'
   overshoot_f(1) = 0.345
   overshoot_f0(1) = 0.01

   ! a bit of exponential overshooting for convective core during He burn
   overshoot_scheme(2) = 'exponential'
   overshoot_zone_type(2) = 'burn_He'
   overshoot_zone_loc(2) = 'core'
   overshoot_bdy_loc(2) = 'top'
   overshoot_f(2) = 0.01
   overshoot_f0(2) = 0.005

   use_ledoux_criterion = .true.
   alpha_semiconvection = 1d0

   ! stop when the center mass fraction of h1 drops below this limit
   ! relax default dHe/He, otherwise growing convective regions can cause things to go at a snail pace
   dX_limit_species(1) = 'he4'
   dX_div_X_limit(1) = 5d0
   dX_div_X_limit_min_X(1) = 1d-1
   ! we're not looking for much precision at the very late stages
   dX_nuc_drop_limit = 5d-2
   ! stop when the center mass fraction of he4 drops below this limit
   xa_central_lower_limit_species(1) = 'he4'
   xa_central_lower_limit(1) = 1d-2

   ! reduce resolution and solver tolerance to make runs faster
   mesh_delta_coeff = 3d0
   time_delta_coeff = 3d0
   varcontrol_target = 1d-2
   use_gold2_tolerances = .false.
   use_gold_tolerances = .true.

   ! Use scaled corrections to aid the solver
   scale_max_correction = 0.03d0
   ignore_min_corr_coeff_for_scale_max_correction = .true.
   ignore_species_in_max_correction = .true.
   scale_max_correction_for_negative_surf_lum = .true.

   use_superad_reduction = .true.
   eps_mdot_leak_frac_factor = 0d0

   ! output options
   profile_interval = 500
   history_interval = 1
   terminal_interval = 1
   write_header_frequency = 10
   max_num_profile_models = 10000

/ ! end of controls namelist
```
{{< /details>}}

#### Modify `pgstar` in `inlist1`

Playing with `pgstar` can be very entertaining, but for this lab we will use a pre-made window. You can copy all this content and replace the default entirely:

{{< details title="Nice `pgstar` window" closed="true" >}}

```fortran
&pgstar

   pgstar_interval = 1

   pgstar_age_disp = 2.5
   pgstar_model_disp = 2.5

   !### scale for axis labels
   pgstar_xaxis_label_scale = 1.3
   pgstar_left_yaxis_label_scale = 1.3
   pgstar_right_yaxis_label_scale = 1.3

   Grid2_win_flag = .true.

   Grid2_win_width = 15
   Grid2_win_aspect_ratio = 0.65 ! aspect_ratio = height/width

   Grid2_plot_name(4) = 'Mixing'

   Grid2_num_cols = 7 ! divide plotting region into this many equal width cols
   Grid2_num_rows = 8 ! divide plotting region into this many equal height rows
   Grid2_num_plots = 6 ! <= 10

   Grid2_plot_name(1) = 'TRho_Profile'
   Grid2_plot_row(1) = 1 ! number from 1 at top
   Grid2_plot_rowspan(1) = 3 ! plot spans this number of rows
   Grid2_plot_col(1) =  1 ! number from 1 at left
   Grid2_plot_colspan(1) = 2 ! plot spans this number of columns 
   Grid2_plot_pad_left(1) = -0.05 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(1) = 0.01 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(1) = 0.00 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(1) = 0.05 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(1) = 0.65 ! multiply txt_scale for subplot by this


   Grid2_plot_name(5) = 'Kipp'
   Grid2_plot_row(5) = 4 ! number from 1 at top
   Grid2_plot_rowspan(5) = 3 ! plot spans this number of rows
   Grid2_plot_col(5) =  1 ! number from 1 at left
   Grid2_plot_colspan(5) = 2 ! plot spans this number of columns 
   Grid2_plot_pad_left(5) = -0.05 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(5) = 0.01 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(5) = 0.03 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(5) = 0.0 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(5) = 0.65 ! multiply txt_scale for subplot by this
   Kipp_title = ''
   Kipp_show_mass_boundaries = .true.

   Grid2_plot_name(6) = 'HR'
   HR_title = ''
   Grid2_plot_row(6) = 7 ! number from 1 at top
   Grid2_plot_rowspan(6) = 2 ! plot spans this number of rows
   Grid2_plot_col(6) =  6 ! number from 1 at left
   Grid2_plot_colspan(6) = 2 ! plot spans this number of columns 

   Grid2_plot_pad_left(6) = 0.05 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(6) = -0.01 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(6) = 0.0 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(6) = 0.0 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(6) = 0.65 ! multiply txt_scale for subplot by this

   History_Panels1_title = ''      
   History_Panels1_num_panels = 3

   History_Panels1_xaxis_name='model_number'
   History_Panels1_max_width = -1 ! only used if > 0.  causes xmin to move with xmax.

   History_Panels1_yaxis_name(1) = 'period_days' 
   History_Panels1_other_yaxis_name(1) = ''
   History_Panels1_yaxis_log(1) = .true.
   History_Panels1_yaxis_reversed(1) = .false.
   History_Panels1_ymin(1) = -101d0 ! only used if /= -101d0
   History_Panels1_ymax(1) = -101d0 ! only used if /= -101d0        
   !History_Panels1_dymin(1) = 0.1 

   History_Panels1_yaxis_name(2) = 'lg_mtransfer_rate' !
   History_Panels1_yaxis_reversed(2) = .false.
   History_Panels1_ymin(2) = -8d0 ! only used if /= -101d0
   History_Panels1_ymax(2) = -1d0 ! only used if /= -101d0        
   History_Panels1_dymin(2) = 1 

   ! ADD THE L2 MASS OUTFLOW RATE TO THE HISTORY PANEL
   History_Panels1_other_yaxis_name(2) = '' 
   History_Panels1_other_yaxis_reversed(2) = .false.
   History_Panels1_other_ymin(2) = -8d0 ! only used if /= -101d0
   History_Panels1_other_ymax(2) = -1d0 ! only used if /= -101d0        
   History_Panels1_other_dymin(2) = 1 

   History_Panels1_yaxis_name(3) = 'rl_relative_overflow_1'
   History_Panels1_other_yaxis_name(3) = ''
   History_Panels1_yaxis_reversed(3) = .false.

   Grid2_plot_name(2) = 'Text_Summary1'
   Grid2_plot_row(2) = 7 ! number from 1 at top
   Grid2_plot_rowspan(2) = 2 ! plot spans this number of rows
   Grid2_plot_col(2) = 1 ! number from 1 at left
   Grid2_plot_colspan(2) = 4 ! plot spans this number of columns 
   Grid2_plot_pad_left(2) = -0.08 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(2) = -0.10 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(2) = 0.08 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(2) = -0.04 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(2) = 0.19 ! multiply txt_scale for subplot by this
   Text_Summary1_name(7,1) = 'period_days'
   Text_Summary1_name(8,1) = 'star_2_mass'
   Text_Summary1_name(7,4) = ''

   ! ADD THE TDELAY TO THE TEXT SUMMARY
   Text_Summary1_name(8,4) = ''

   Grid2_plot_name(3) = 'Profile_Panels3'
   Profile_Panels3_title = 'Abundance-Power-Mixing'
   Profile_Panels3_num_panels = 3
   Profile_Panels3_yaxis_name(1) = 'Abundance'
   Profile_Panels3_yaxis_name(2) = 'Power'
   Profile_Panels3_yaxis_name(3) = 'Mixing'

   Profile_Panels3_xaxis_name = 'mass'
   Profile_Panels3_xaxis_reversed = .false.

   Grid2_plot_row(3) = 1 ! number from 1 at top
   Grid2_plot_rowspan(3) = 6 ! plot spans this number of rows
   Grid2_plot_col(3) = 3 ! plot spans this number of columns 
   Grid2_plot_colspan(3) = 3 ! plot spans this number of columns 

   Grid2_plot_pad_left(3) = 0.09 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(3) = 0.07 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(3) = 0.0 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(3) = 0.0 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(3) = 0.65 ! multiply txt_scale for subplot by this

   Grid2_plot_name(4) = 'History_Panels1'
   Grid2_plot_row(4) = 1 ! number from 1 at top
   Grid2_plot_rowspan(4) = 6 ! plot spans this number of rows
   Grid2_plot_col(4) =  6 ! number from 1 at left
   Grid2_plot_colspan(4) = 2 ! plot spans this number of columns 
   Grid2_plot_pad_left(4) = 0.05 ! fraction of full window width for padding on left
   Grid2_plot_pad_right(4) = 0.03 ! fraction of full window width for padding on right
   Grid2_plot_pad_top(4) = 0.0 ! fraction of full window height for padding at top
   Grid2_plot_pad_bot(4) = 0.07 ! fraction of full window height for padding at bottom
   Grid2_txt_scale_factor(4) = 0.65 ! multiply txt_scale for subplot by this

   Grid2_file_flag = .true.
   Grid2_file_dir = 'png1'
   Grid2_file_prefix = 'grid_'
   Grid2_file_interval = 1 ! 1 ! output when mod(model_number,Grid2_file_interval)==0
   Grid2_file_width = -1 ! negative means use same value as for window
   Grid2_file_aspect_ratio = -1 ! negative means use same value as for window
      
/ ! end of pgstar namelist
```

{{< /details>}}

#### Get `final1_caseA.mod` and `final2_caseA.mod`
Grab the final models `final1_caseA.mod` and `final2_caseA.mod` from your minilab1 (or download them from [Table 1](#table-binary)) and copy them into your work folder.

<!-- #### Get `history_columns.list` and `binary_history_columns.list`
Grab the <a href="/files/binary_history_columns.list" download>`history_columns.list`</a> and <a href="/files/binary_history_columns.list" download>`history_columns.list`</a> and copy them into your work folder. -->




### The stripped star becomes a BH companion
After He depletion in its core, the stripped star has a very short remaining lifetime: for a star of total mass ~ 20 $M_{\odot}$ (and He-core mass of ~ 10 $M_{\odot}$), you can expect it to live only another ~ 300 years! For simplicity, we will just assume that the properties at core He depletion can be representative of those at the end of the star's life. Additionally, we will assume that all the mass contained in the primary directly collapses to form a BH.

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>inlist_project</code>
  </div>

Find the mass of the stripped star at core He depletion from your <code>minilab1</code> and make it directly collapse into a BH. Set the period of your binary to be the one you found at the end of <code>minilab1</code>.
</div>



<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>How do I find the BH mass?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Your <code>inlist_project</code> already has <code>evolve_both_stars = .false.</code>, so one of the two stars will be treated as a point mass --> BH!
  To find the mass of this BH, you can open the <code>final1_caseA.mod</code> and look at the header.


  </div>
</details>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22)
  ">
    💡 <strong><code>m1</code> or <code>m2</code>?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22)
  ">

  Remember which one was stripped in Minilab1 (the primary, <code>final1_caseA.mod</code>), and which one was accreted (the secondary, <code>final2_caseA.mod</code>). The stripped star will become the BH (<code>m2</code>), and the accreted star will become your new primary (<code>m1</code>).

  </div>
</details>


{{< details title="Solution" closed="true" >}}
```fortran
   m1 = 39.6d0 ! primary mass in Msun
   m2 = 16.8d0 ! BH mass in Msun
   initial_period_in_days = 4.5d0 ! final period from minilab1
```
{{< /details >}}

Now you have set up the properties of your binary system. We are only missing one piece: we need to load the model of the accretor from minilab1 as our new primary star.

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>inlist1</code>
  </div>

Load the right model from your <code>minilab1</code> in your `inlist1`.
</div>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Which command is it?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Checking inside `$MESA_DIR/star/defaults/star_job.defaults`. You can search for the string `load`...

  </div>
</details>

{{< details title="Solution" closed="true" >}}
Inside your `inlist1`:
```fortran
&star_job
   ...
   load_saved_model = .true.
   load_model_filename = 'final2_caseA.mod'
   ...
/ ! end of star_job namelist
```
{{< /details >}}

> [!NOTE]
> Did you need to set both masses in `inlist_project`? Nope! Since you loaded a pre-computed model for your new primary (the accretor of minilab1), its mass and properties will be read directly from the model `final2_caseA.mod`.


### Computing the time delay
<a id="eq-tdelay"></a>
Compact binaries composed of neutron stars and BHs gradually lose orbital energy through the emission of gravitational waves. As a consequence, the orbit shrinks over time until the two compact objects eventually merge.  

The **gravitational-wave time delay** (or simply *delay time*) is the time required for this inspiral and merger to occur **if gravitational-wave emission were the only process acting on the binary orbit**. Delay times are particularly important in astrophysics because they determine *when* mergers happen relative to the formation of the stars, and therefore affect the populations of gravitational-wave sources observed by detectors such as LIGO, Virgo and KAGRA[^GWTC4].

For a circular orbit, the merger timescale derived by Peters (1964)[^peters1964] is

$$t_{\rm delay} =\frac{5}{256}\frac{c^5 a^4}{G^3 m_1 m_2 (m_1 + m_2)} ,\,\tag{1}$$

where:

- $a$ is the orbital separation at the BH + BH stage;
- $m_1$ and $m_2$ are the masses of the two BHs,
- $G$ is the gravitational constant,
- $c$ is the speed of light.

Notice the strong dependence $t_{\rm delay} \propto a^4$: even modestly wider binaries can take dramatically longer to merge. The dependence on the masses is a bit weaker, but the rule of thumb is that more massive systems will merge faster.

In practice, delay times are often expressed in **Gigayears (Gyr)**, where $1~{\rm Gyr} = 10^9$ years, because this makes it easy to compare them to the age of the Universe $\approx 13.8$ Gyr). This comparison tells us whether a compact binary has enough time to merge within cosmic history: binaries with $t_{\rm delay} \lesssim 13.8$ Gyr may merge and be observable today as gravitational-wave sources, while systems with longer delay times are effectively undetectable (technically, they have "not yet happened" anywhere in the Universe 🤓).

> [!Note]
> Remember that **so far, you have initialized a system with a star + BH**, since you have collapsed all the mass of the stripped star of minilab1 into a point mass <code>m2</code>, and initialized your <code>m1</code> to be the accretor of minilab1. **After evolving your system further**, your <code>m1</code> will reach Helium depletion in its core, and at the point it will very close to become the second BH of your system: that's how **you will achieve a BH + BH binary**! From that point onwards, the $t_{\mathrm{delay}}$ calculation will make sense, as the interaction between the two BHs is expected to be only via gravity.


> [!Important]
> In principle, BHs can accrete mass (and in fact, that is when they become [X-Ray active](https://en.wikipedia.org/wiki/X-ray_binary) 🌝), and this option in your `inlist_project`
> ```fortran
> limit_retention_by_mdot_edd = .true.
> ```
> will make such that the accretion rate onto the BH is Eddington-limited (we talked about it in [here](#1-stable-mass-transfer)). This is a standard assumption when treating BH accretors, which is observationally motivated by beautiful systems like the galactic microquasar [SS433](https://en.wikipedia.org/wiki/SS_433), where powerful relativistic jets are thought to drive material away from the central binary. However, keep in mind that this is not exact science and **there are other possible ways to treat (and limit) accretion onto BHs**: we will explore one further down in the lab.

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>run_binary_extras.f90</code>
  </div>

Let's compute the $t_{\mathrm{delay}}$ in Gigayears (<a href="#eq-tdelay">Eq. (1)</a>) as an extra binary history column `tdelay(Gyr)` in `run_binary_extras.f90`, and print its value on the `pgstar` window, in the Text Summary part.
</div>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Mind the units...</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  In stellar astrophysics and in MESA we like to use the centimeter-gram-second units, therefore we saved our own useful constants to convert between energy units, or from seconds to years, and such.

  Those constants are readily accessible in <code>run_star_extras.f90</code>, if you know what their name is 🙃

  You may want to use:
  <ul>
    <li><code>clight</code>, the speed of light in $\mathrm{cm}\:\mathrm{s}^{-1}$
    <li><code>secyer</code>, the conversion between years and seconds</li>
    <li><code>standard_cgrav</code>, the gravitational constant in c.g.s.</li>
  </ul>

  Loaded via: <code>use const_def</code><br>
  See also: <code>$MESA_DIR/const/public/const_def.f90</code>

  </div>
</details>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Fighting with the binary pointer <code>b%</code> ?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Don't be scared, the `binary_info` structure and its type `b` work very similarly to the `star_info` and its type `s`! Try to find the quantities of interest inside `$MESA_DIR/binary/public/binary_data.inc`, and refer to them with the pointers `b% m(1)`, `b% m(2)`, ecc. Pay attention to the units!


  </div>
</details>

{{< details title="Solution for `run_binary_extras.f90`" closed="true" >}}
```fortran
subroutine data_for_extra_binary_history_columns(binary_id, n, names, vals, ierr)
  type(binary_info), pointer :: b
  integer, intent(in) :: binary_id
  integer, intent(in) :: n
  character(len=maxlen_binary_history_column_name) :: names(n)
  real(dp) :: vals(n)
  integer, intent(out) :: ierr
  ierr = 0
  call binary_ptr(binary_id, b, ierr)
  if (ierr /= 0) then
      write (*, *) 'failed in binary_ptr'
      return
  end if
  
  names(1) = 'tdelay(Gyr)'
  vals(1) = (5d0/256d0) * (clight**5 * b%separation**4) / &
        (standard_cgrav**3 * b%m(1) * b%m(2) * (b%m(1) + b%m(2)))

  ! convert from seconds -> years -> Gyr
  vals(1) = vals(1) / secyer / 1d9

end subroutine data_for_extra_binary_history_columns
```

```fortran
integer function how_many_extra_binary_history_columns(binary_id)
  use binary_def, only: binary_info
  integer, intent(in) :: binary_id
  how_many_extra_binary_history_columns = 1
end function how_many_extra_binary_history_columns
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Too many <code>pgstar</code> things to look at?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Search for the string "`TDELAY`" in `inlist1` 😏


  </div>
</details>

{{< details title="Solution for `pgstar`" closed="true" >}}
```fortran
! ADD THE TDELAY TO THE TEXT SUMMARY
Text_Summary1_name(8,4) = 'tdelay(Gyr)'
```
{{< /details >}}

 You have come a long way, congrats! We are finally ready to start our first run. 


> [!WARNING]
> Never forget to do `./clean` and `./mk` after modifying the `run_binary_extras.f90` file.

<div style="
  max-width: 600px;
  margin: 20px auto;
  border: 1px solid none;
  border-radius: 8px;
  background-color: #f5c2c2;
  overflow: hidden;
  color: black;
  font-family: sans-serif;
">

  <!-- Header -->
  <div style="
    background-color: #d9534f;
    color: black;
    font-weight: bold;
    text-align: center;
    padding: 8px;
  ">
    RUN 1 (5 minutes on 4 cores, 749 models)
  </div>

  <!-- Body -->
  <div style="
    padding: 15px;
    text-align: center;
  ">
    <p style="margin: 0;">
      Run your star + BH model with <code>./rn | tee output.txt</code>.<br>
      In case you need them, here are the complete inlists for this run:
      <a href="../lab3/stable_MT_SOL.zip" download>
        <code>stable_MT_SOL.zip</code>
      </a>
    </p>
  </div>

</div>

Your `pgstar` window should look like something like this (this is the very last model of your run, model 749):

<!-- ![pgstar_stable_caseA_new](/thursday/lab3/pgstar_stable_caseA_new.png) -->
<a id="fig-caseA"></a>
[![Case A figure](/thursday/lab3/pgstar_stable_caseA_new.png)](/thursday/lab3/pgstar_stable_caseA_new.png)

**Figure 1.** Stable mass transfer, Case A evolution for a star + BH binary (click to zoom in!).

- Make sure that the **Kippenhahn diagram** shows nice convective zones (filled in light blue) and the Helium core (the solid green line). If it is the case, you did well in putting `mixing_regions 10` in `history_columns.list` (as indicated in this <a href="#modify-history_columnslist">section</a>) ☺️ Otherwise, no problem. Do it now, as we will look into the Kippenhahn diagram for a later run. You can also download the correct file <a href="../lab3/history_columns.list" download> <code>here</code></a>.
- Check that the **info about the `tdelay(Gyr)`** column is appearing in the Text Summary, as you can see in here. If not, you may have done something wrong with the implementation... You can try again, but if you're short on time, just look at <a href="#fig-caseA">Figure 1</a> (click to zoom in!) to answer to the Analysis of the run questions here below.

### Analysis of the run: Case A mass transfer!
Here are some discussion points for you to understand what happened physically to your star + BH system; you will only need to look at <a href="#fig-caseA">Figure 1</a> (click to zoom in!). Try to think about it and answer together with your table.

1. Which type of mass transfer do you observe in this star + BH run?  
   {{< details title="Solution" closed="true" >}}
  The mass transfer starts during the Main Sequence → Case A!
  You can see it from the shape of the Hertzsprung-Russel diagram (ask your TA if you don't know!).
{{< /details >}}

2. How much mass did the donor star lose?
   {{< details title="Solution" closed="true" >}}
The donor star started from a mass of $39.8\:M_{\odot}$ (see [Table 1](#table-binary)), and is now $23.95\:M_{\odot}$. You can read this value in the Text Summary (`star_mass`), or look at the Kippenhahn diagram. So it lost $15.85\:M_{\odot}$, which corresponds to 40% of its initial mass!
{{< /details >}}
3. Is the donor star stripped to its core?
      {{< details title="Solution" closed="true" >}}
Almost! Look at the Text Summary (`he_core_mass`) and / or the Kippenhahn diagram. The Helium core is $22.50\:M_{\odot}$, and the donor is stripped to a total mass of $23.95\:M_{\odot}$. The Helium core amounts to 93.9% of the star, while the Hydrogen-rich envelope amounts to $23.95−22.50=1.45\:M_{\odot}$, only 6.1%.
{{< /details >}}
4. How much mass did the BH accrete? Do you understand why?
      {{< details title="Solution" closed="true" >}}
The BH accretor started from a mass of $16.8\:M_{\odot}$ (see [Table 1](#table-binary)), and is now $16.98\:M_{\odot}$ (see `star_2_mass` value). Therefore it only accreted $0.18\:M_{\odot}$, so it increased its mass by only 1.1%. The Eddington-limited accretion made such that the mass transfer becomes almost completely non-conservative (i.e., all the material is expelled from the binary).
{{< /details >}}
5. How did the orbit evolve during mass transfer? And what is the final period?
  {{< details title="Solution" closed="true" >}}
The system started with a period of $4.5$ days, and has followed a sort of parabola-shaped path (see the `log_period_days` plot) that has led to shrinkage. At the end of the mass transfer episode, the orbit is $2.4$ days wide. This amounts to almost 50% overall shrinkage!
{{< /details >}}
6. Assume that the donor star will collapse into a BH of mass equal to its mass at Helium depletion (end of the run). Will the system merge within the age of the Universe?
      {{< details title="Solution" closed="true" >}}
You have calculated the `tdelay(Gyr)` yourself 🌝 You see it is equal to 4.28 Gyr, less than the age of the Universe ($\sim 13.8\:\mathrm{Gyr}$). Yes, we have chirping binary black holes!
{{< /details >}}
7. How is the final mass ratio of your BH + BH binary?
      {{< details title="Solution" closed="true" >}}
The donor will collapse into a BH of mass $23.95\:M_{\odot}$, and the companion BH has a mass of $16.98\:M_{\odot}$. The mass ratio is then $\sim 0.7$! This number is right about what more detailed population synthesis studies expect for stable mass transfer to produce 😌 
{{< /details >}}


### Orbital tightening from L2 mass loss
So far, we have considered an **Eddington-limited** mass-transfer scenario, in which matter that cannot be accreted by the black hole is expelled from the vicinity of the accretor itself. This is the so-called **isotropic re-emission mode**. In this picture, the expelled material removes the **specific angular momentum of the accretor** from the binary system.

However, this is not the only possible way for matter to leave the binary. 3D hydrodynamical simulations [^lu2022] show that when the mass-transfer rate becomes sufficiently high (roughly $\dot{M} \gtrsim 10^{-4}\ M_\odot\,\mathrm{yr}^{-1}$), some of the transferred material can instead be lifted all the way to the **second Lagrangian point**, $L_2$. This is the Lagrangian point located on the far side of the less massive object in the binary (see <a href="#fig-L2">Figure 2</a>).

Because the $L_2$ point is located farther away from the center of mass than the accretor itself, material escaping through $L_2$ carries away **much more angular momentum** than in the isotropic re-emission case.

<a id="fig-L2"></a>
[![L2 outflow](/thursday/lab3/L2_outflow.jpeg)](/thursday/lab3/L2_outflow.jpeg)
**Figure 2.** Schematics[^lu2022] of $L_2$ outflow in a binary, where the $\Phi$s indicate different levels of gravitational equipotential; $L_1$ is the first Lagrangian point (through which material can flow). 

<a id="eq-Jdot_iso"></a>
In practice, this introduces an additional contribution to the orbital angular momentum evolution of the binary system, which for simplicity we will write as

$$\dot{J}_{\mathrm{ml}}=\dot{J}_{\mathrm{isotropic}}+\dot{J}_{\mathrm{L2}},$$

where these $\dot{J}$ is the time derivative of the angular momentum component $J$ (and "ml" = mass loss). The angular momentum loss associated with matter expelled through the $L_2$ point can be written as

$$\dot{J}_{\mathrm{L2}}=\upsilon\times\dot{M}_{\mathrm{MT}}\left[\left(\frac{m_{\mathrm{accretor}}}     {m_{\mathrm{accretor}} + m_{\mathrm{donor}}}-x_{\mathrm{L2}}\right)a\right]^2\frac{2\pi}{P} ,\,\tag{2}$$

while the standard isotropic re-emission contribution is

$$\dot{J}_{\mathrm{isotropic}}=\beta\times\dot{M}_{\mathrm{MT}}\left(\frac{m_{\mathrm{donor}}}     {m_{\mathrm{accretor}} + m_{\mathrm{donor}}}a\right)^2\frac{2\pi}{P} .\,\tag{3}$$

Here:

- $\dot{M}_{\mathrm{MT}}$ is the mass transfer rate
- $a$ is the orbital separation,
- $P$ is the orbital period,
- $x_{\mathrm{L2}}$ is the position of the $L_2$ point in units of the orbital separation,
- $\beta$ is the fraction of transferred mass expelled through isotropic re-emission,
- $\upsilon$ is the fraction expelled through the $L_2$ outflow channel.

These efficiency factors determine how conservative the mass transfer is:

- $\upsilon + \beta = 1$  → all transferred material is expelled from the system;
- $\upsilon + \beta = 0$  → fully conservative mass transfer (everything is retained in the system).
- $\epsilon\equiv\upsilon+\beta$  → this is the same $\epsilon$ that you saw in minilab1 (where $\epsilon=1$ is for conservative mass transfer, and $\epsilon=0$ is fully non-conservative), but this time it is modified to our purpose of having only two types of mass leakage: the isotropic re-emission mode, and $L_2$ overflow.

$L_2$ mass outflow has been associated observationally with **circumbinary outflows** (see the CBO in <a href="#fig-L2">Figure 2</a>) in nearby ($\lesssim 10$ Megaparsecs!) **[ultraluminous X-ray sources](https://en.wikipedia.org/wiki/Ultraluminous_X-ray_source)**. These outflows are thought to absorb and reprocess radiation from the central accreting source, naturally producing the infrared excess observed in the ultraluminous X-ray sources[^lu2022]. An even closer (in our Galaxy!) candidate for this type of mass loss is again [SS433](https://en.wikipedia.org/wiki/SS_433), for which spectroscopic observations have been interpreted as evidence for material escaping through the $L_2$ region and forming a circumbinary structure[^bowler2010]. While **there is no direct smoking gun system where we directly see gas leaving from $L_2$**, we infer it through their required angular-momentum loss, the presence of circumbinary structures, and consistency with extreme mass-transfer regimes.

> [!IMPORTANT]
>In the context of gravitational wave sources, $L_2$ mass outflow is expected to efficiently tighten star + BH binaries that are residing in quite wide orbits, so that after the detachment, the binary will be already close enough to start chirping at the formation of the second BH! **In this part of the minilab3, we will demonstrate that, in presence of $L_2$ mass outflow, a wide binary, like the Case B system you produced in minilab1, can form a gravitational wave source after stable mass transfer.**
>
> You can start from the same setup as you developed so far (also downloadable <a href="../lab3/stable_MT_SOL.zip" download> <code>here</code></a>):
> ```bash
> cp -r stable_MT stable_MT_L2
>```
> Remind yourself of the properties of your Case B system in [Table 2](#table-caseB), and download the `final1_caseB.mod` and `final2_caseB.mod`. You see that the masses are mostly the same, but you will have to change the period, and load the right model 😎
> {{< details title="Solution for `inlist_project`" closed="true" >}}
>```fortran
>&binary_controls
>  ...
>  m1 = 40.8d0 ! donor mass in Msun
>  m2 = 17.14d0 ! companion mass in Msun
>  initial_period_in_days = 32.2d0
>  ...
>/ ! end of binary_controls namelist
>```
>{{< /details >}}
> {{< details title="Solution for `inlist1`" closed="true" >}}
>```fortran
>&star_job
>  ...
>  load_saved_model = .true.
>  load_model_filename = 'final2_caseB.mod'
>  ...
>/ ! end of star_job namelist
>```
>{{< /details >}}


> [!NOTE]
> 1. Your setup still has the Eddington-limited mass accretion rate switched on, like we wanted to in our first part of the lab. But now we want to introduce our own prescription for how the mass outflows from the system, introducing $L_2$ outflow! Therefore, we will set this to `.false.` now in `inlist_project`
  > ```fortran
  > limit_retention_by_mdot_edd = .false.
  >```
> 2. To compute the orbital evolution due to mass losses from the system (`jdot_ml` equation) and the changes in mass due to accretion / leakages (`mdots` equation), MESA uses two default routines: 
>   - **`default_jdot_ml`**, to be found in `$MESA_DIR/binary/private/binary_jdot.f90`. If you want to create your own personalized version of it, the empty hook from where you usually start is: `$MESA_DIR/binary/other/mod_other_binary_jdot.f90`
>  - **`adjust_mdots`**, to be found in `$MESA_DIR/binary/private/binary_mdot.f90`. If you want to create your own personalized version of it, the empty hook from where you usually start is: `$MESA_DIR/binary/other_mod_other_adjust_mdots.f90`

      

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>run_binary_extras.f90</code>
  </div>

Let's make such that our binary will lose 35% of transferred material through the second Lagrangian point $L_2$ ($\upsilon=0.35$), and the rest 65% will be lost from the vicinity of the accretor ($\beta=0.65$). You will need to introduce two personalized routines: `my_jdot_ml` and `my_adjust_mdots`, and an `x_ctrl(1)` in `inlist1`. This is a difficult task, so **don't be scared and read all the hints**!
</div>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(236, 72, 153, 0.14);
    border-left:4px solid rgba(236, 72, 153, 0.14);
  ">
    🎁 <strong>Let's find L2 in <code>run_binary_extras.f90</code></strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(236, 72, 153, 0.14);
    border-left:4px solid rgba(236, 72, 153, 0.14);
  ">

  This is a gift for you (or simply, something you can find in literature [^marchant2021] and you don't need to know how to code on the spot 🤙🏻). Copy this entirely at the end of your `run_binary_extras.f90`.

  {{< details title="Routine to copy into `run_binary_extras.f90`" closed="true" >}}
  ```fortran
! ROCHE POTENTIAL FIRST DERIVATIVE
! To find the Lagrangian points numerically, by bisection
real(dp) function dPhidx(b,x) result(derivative)
  real(dp), intent(in) :: x
  real(dp) :: q, x_cm
  type(binary_info), pointer :: b
  ! include 'formats.inc'

  q = b% m(b% d_i)/b% m(b% a_i)
  x_cm = 1/(1+q)
  derivative = 1/(x*abs(x)) +1/(q*(x-1)*abs(x-1)) -(x-x_cm)*(q+1)/q

end function dPhidx


! FUNCTION TO FIND THE COORDINATE OF L2 IN UNITS OF SEPARATION
real(dp) function find_L2(b) result(L2)
  real(dp) :: limit, tolerance, x, upper_bound, lower_bound, dPhi_new,q
  type(binary_info), pointer :: b
  ! include 'formats.inc'

  q = b% m(b% d_i)/b% m(b% a_i)

  if (q < 1) then
    upper_bound = 0d0
    lower_bound = -1d0
    limit = abs(upper_bound-lower_bound)/abs(lower_bound)
  end if

  if (q .GE. 1) then
    upper_bound = 2d0
    lower_bound = 1d0
    limit = abs(upper_bound-lower_bound)/abs(upper_bound)
  end if

  x = 0d0

  tolerance = 0.000001d0

  do while (limit > tolerance)
    x = (lower_bound+upper_bound)/2
    dPhi_new = dPhidx(b,x)
    if (dPhi_new > 0) then
        lower_bound = x
    else if (dPhi_new < 0) then
        upper_bound = x
    else
        exit
    end if

    if (q < 1) then
        limit = abs(upper_bound-lower_bound)/abs(lower_bound)
    end if

    if (q .GE. 1) then
        limit = abs(upper_bound-lower_bound)/abs(upper_bound)
    end if

  end do
  L2 = (upper_bound + lower_bound)/2
end function find_L2
  ```
{{< /details >}}

  </div>
</details>



<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>35% of mass lost via L2?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  We can make use of an `x_ctrl(1)` to set the fraction of mass that is lost from the system removing the specific angular momentum of the $L_2$ point!

  </div>
</details>

{{< details title="Solution for `inlist1`" closed="true" >}}
```fortran
&controls
  ...
  x_ctrl(1) = 0.35d0
  ...
/ ! end of controls namelist
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>How to lose the remaining 65%?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  To set the fraction $\beta$ of mass that is leaving the system with the specific angular momentum of the accretor (isotropic re-emission mode), there is actually a default control that you can look up in `$MESA_DIR/binary/defaults/binary_controls.defaults`. You can look for the string `beta` 🙃

  </div>
</details>


{{< details title="Solution for `inlist_project`" closed="true" >}}
```fortran
&controls
  ...
  ! transfer efficiency controls
   limit_retention_by_mdot_edd = .false.
   mass_transfer_beta = 0.65d0
  ...
/ ! end of controls namelist
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Is all mass lost in the way that I want?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  In `$MESA_DIR/binary/defaults/binary_controls.defaults`, give a look at the meaning and the default values of all these controls: <code>mass_transfer_alpha</code>, <code>mass_transfer_delta</code>, <code>mass_transfer_gamma</code>.

  Additionally, remember that you have set this in `inlist_project`
  ```fortran
  limit_retention_by_mdot_edd = .false.
  ```
  

  </div>
</details>

{{< details title="Solution: yes 😛" closed="true" >}}

As you saw in minilab1, these fractions $\alpha$, $\gamma$ and $\delta$ describe other possible leakages of mass, which we are not interested in. Luckily, they are all set to zero by default! So the only only that we have set is $\beta=0.65$, as wanted. The remaining 0.35 mass outflow will be calculated by us with our own personalized fraction $\upsilon$ (or `x_ctrl(1)`).

Additionally, the Eddington limit is switched off, so that we are in full control of where 35%+65%=100% of the mass goes!

{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Activating new <code>run_binary_extras.f90</code> hooks </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  We are going to create our own personalized `run_binary_extras.f90` routines to compute the orbital evolution due to mass losses from the system (`jdot_ml` equation) and the changes in mass due to accretion / leakages (`mdots` equation). We need to instruct `inlist_project` to use the new routines! Try to find the right controls in `$MESA_DIR/binary/defaults/binary_controls.defaults`. You can look for the string `use_other_`.

  </div>
</details>

{{< details title="Solution for `inlist_project`" closed="true" >}}
```fortran
&binary_controls
  ...
  ! ADD personalized run_binary_extras.f90 routine
  use_other_jdot_ml = .true.
  use_other_adjust_mdots = .true.
  ...
/ ! end of binary_controls namelist
```


{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Set the function pointers in <code>run_binary_extras.f90</code> </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  As usual when we activate personalized hooks in `run_binary_extras.f90`, we also need to instruct `extras_binary_controls` to point to those new routines, that we will call `my_adjust_mdots` and `my_jdot_ml`.

  </div>
</details>

{{< details title="Solution for <code>extras_binary_controls</code>" closed="true" >}}
```fortran
subroutine extras_binary_controls(binary_id, ierr)
  ...
  ...
  ! EXTRA SHRINKAGE FOR L2 MASS OUTFLOW!
  ! Default routine: $MESA_DIR/binary/private/binary_jdot.f90
  ! But the empty hook from where you usually start is: 
  ! $MESA_DIR/binary/other/mod_other_binary_jdot.f90
  b% other_jdot_ml => my_jdot_ml

  ! Default routine: $MESA_DIR/binary/private/binary_mdot.f90
  ! But the empty hook from where you usually start is: 
  ! $MESA_DIR/binary/other/mod_other_adjust_mdots.f90
  b% other_adjust_mdots => my_adjust_mdots
  ...
  ...
end subroutine extras_binary_controls
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Creating <code>my_adjust_mdot</code> </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Start by copying this guided skeleton entirely inside your `run_binary_extras.f90`. This is just a commented version of the classic empty routine `null_other_adjust_mdots` from `$MESA_DIR/binary/other_mod_other_adjust_mdots.f90`! So in the future, you will know where to start. 

  {{< details title="Guided skeleton of `my_adjust_mdots`" closed="true" >}}
```fortran
subroutine my_adjust_mdots(binary_id, ierr)
    use binary_def, only : binary_info, binary_ptr
    use const_def, only: dp
    integer, intent(in) :: binary_id
    integer, intent(out) :: ierr
    type (binary_info), pointer :: b
    real(dp) :: fixed_xfer_fraction
    ierr = 0
    call binary_ptr(binary_id, b, ierr)
    if (ierr /= 0) then
    write(*,*) 'failed in binary_ptr'
    return
    end if
    
    ! THIS IS WHERE YOU CAN IMPOSE THE MASS TRANSFER FRACTION
    ! In your minilab1, this looked like: 
    ! epsilon = 1 - alpha - beta - delta - gamma.
    ! In this minilab3, we want to only model beta (isotropic re-emission)
    ! and upsilon (L2 outflow), while all the rest is already set to zero
    ! in the defaults.
    b% fixed_xfer_fraction = 0d0    !!!modify this
    
    
    ! EDDINGTON ACCRETION RATE
    ! Usually, one should also eval mdot_edd here by calling the default
    ! functions using the ones provided through binary_lib. 
    ! But in our minilab3, we want to ignore Eddington limits anyway...
    b% mdot_edd = 0d0
    b% mdot_edd_eta = 0d0


    ! WIND MASS TRASNFER EFFICIENCY
    ! Usually, one should also eval the wind mass transfer efficiency 
    ! here by calling the default functions provided through binary_lib.
    ! But we want to ignore wind mass transfer for simplicity...
    b% mdot_wind_transfer(:) = 0d0
    b% wind_xfer_fraction(:) = 0d0


    ! MASS CHANGES IN THE TWO STARTS DUE TO MASS TRANSFER
    ! Set mdot for the donor
    b% s_donor% mstar_dot = 0d0     !!!modify this
    ! Set mdot for the accretor
    ! point_mass_i is 0 if both stars are evolved, is 1 if there is a BH!
    if (b% point_mass_i == 0) then
        b% component_mdot(b% a_i) = 0d0
    else
        b% component_mdot(b% a_i) = 0d0   !!!modify this
    end if
    ! Accretion luminosity is useful only if you have a compact 
    ! accretor and want to use it to compute the Eddington limit, 
    ! but you can set it to zero if you want to ignore that...
    b% accretion_luminosity = 0d0


    ! mdot_system_transfer is mass lost from the vicinity of each star;
    ! such mass will be removing the angular momentum of the respective star.
    ! We want to model this for the accretor (you can access it via 
    ! b% mdot_system_transfer(b% a_i)), this is the 
    ! isotropic re-emission mode! But we want to ignore it for the donor.
    b% mdot_system_transfer(:) = 0d0    !!!modify this

    ! mdot_system_cct is mass lost from a circumbinary coplanar toroid.
    ! we are not interestered in modeling this one :) 
    b% mdot_system_cct = 0d0 

    end subroutine my_adjust_mdots
```
{{< /details >}}

Read all the comments and try to fill in where you see `!!! modify this`. Keep in mind the following:

- You have set your $\beta$ with the control `mass_transfer_beta`, and your $\upsilon$ with `x_ctrl(1)`. You can access quantities related to your donor from the `binary_info` structure as `b% s_donor`!
- The donor should have an `mdot` that corresponds to the mass transfer rate (which is defined negative, since it loses mass!)
- The accretor should have an `mdot` that corresponds to a fraction $\upsilon+\beta$ of the mass transfer rate (and which sign?) 

  </div>
</details>

{{< details title="Solution for `my_adjust_mdots`" closed="true" >}}
```fortran
subroutine my_adjust_mdots(binary_id, ierr)
    use binary_def, only : binary_info, binary_ptr
    use const_def, only: dp
    integer, intent(in) :: binary_id
    integer, intent(out) :: ierr
    type (binary_info), pointer :: b
    real(dp) :: fixed_xfer_fraction
    ierr = 0
    call binary_ptr(binary_id, b, ierr)
    if (ierr /= 0) then
    write(*,*) 'failed in binary_ptr'
    return
    end if
    
    ! THIS IS WHERE YOU CAN IMPOSE THE MASS TRANSFER FRACTION
    ! In your minilab1, this looked like: 
    ! epsilon = 1 - alpha - beta - delta - gamma.
    ! In this minilab3, we want to only model beta (isotropic re-emission)
    ! and upsilon (L2 outflow), while all the rest is already set to zero
    ! in the defaults.
    b% fixed_xfer_fraction = 1 - b% mass_transfer_beta - b% s_donor% x_ctrl(1)    !!!modify this
    
    
    ! EDDINGTON ACCRETION RATE
    ! Usually, one should also eval mdot_edd here by calling the default
    ! functions using the ones provided through binary_lib. 
    ! But in our minilab3, we want to ignore Eddington limits anyway...
    b% mdot_edd = 0d0
    b% mdot_edd_eta = 0d0


    ! WIND MASS TRASNFER EFFICIENCY
    ! Usually, one should also eval the wind mass transfer efficiency 
    ! here by calling the default functions provided through binary_lib.
    ! But we want to ignore wind mass transfer for simplicity...
    b% mdot_wind_transfer(:) = 0d0
    b% wind_xfer_fraction(:) = 0d0


    ! MASS CHANGES IN THE TWO STARTS DUE TO MASS TRANSFER
    ! Set mdot for the donor
    b% s_donor% mstar_dot = b% mtransfer_rate     !!!modify this
    ! Set mdot for the accretor
    ! point_mass_i is 0 if both stars are evolved, is 1 if there is a BH!
    if (b% point_mass_i == 0) then
        b% component_mdot(b% a_i) = 0d0
    else
        b% component_mdot(b% a_i) = -b% mtransfer_rate* b% fixed_xfer_fraction   !!!modify this
    end if
    ! Accretion luminosity is useful only if you have a compact 
    ! accretor and want to use it to compute the Eddington limit, 
    ! but you can set it to zero if you want to ignore that...
    b% accretion_luminosity = 0d0


    ! mdot_system_transfer is mass lost from the vicinity of each star;
    ! such mass will be removing the angular momentum of the respective star.
    ! We want to model this for the accretor (you can access it via 
    ! b% mdot_system_transfer(b% a_i)), this is the 
    ! isotropic re-emission mode! But we want to ignore it for the donor.
    b% mdot_system_transfer(b% d_i) = 0d0
    b% mdot_system_transfer(b% a_i) = b% mtransfer_rate * b% mass_transfer_beta    !!!modify this

    ! mdot_system_cct is mass lost from a circumbinary coplanar toroid.
    ! we are not interestered in modeling this one :) 
    b% mdot_system_cct = 0d0 

end subroutine my_adjust_mdots
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Creating <code>my_jdot_ml</code> </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  This is the part in which you will use <a href="#eq-Jdot_iso">Eq. (2)</a> and <a href="#eq-Jdot_iso">Eq. (3)</a>.
  Start by copying this guided skeleton entirely inside your `run_binary_extras.f90`. This is just a commented version of the classic empty routine `null_other_jdot_ml` from `$MESA_DIR/binary/other/mod_other_binary_jdot.f90`! So in the future, you will know where to start. 

  {{< details title="Guided skeleton of `my_jdot_ml`" closed="true" >}}
```fortran
subroutine my_jdot_ml(binary_id, ierr)
    use binary_def, only : binary_info, binary_ptr
    integer, intent(in) :: binary_id
    integer, intent(out) :: ierr
    type (binary_info), pointer :: b
    ierr = 0
    call binary_ptr(binary_id, b, ierr)
    if (ierr /= 0) then
    write(*,*) 'failed in binary_ptr'
    return
    end if

    ! THIS IS WHERE YOU CAN IMPOSE THE ANGULAR MOMENTUM LOSS RATE 
    ! ASSOCIATED WITH MASS LOSS
    ! Remember that you have two types of mass loss: 
    ! the isotropic re-emission mode (beta) and the L2 outflow (upsilon).
    ! The first one is already implemented in MESA, 
    ! we just need to find how to write it.
    ! Look at the default routine that MESA uses in 
    ! $MESA_DIR/binary/private/binary_jdot.f90, and copy the relevant piece.
    ! You can also check the formula that we wrote on the website for Jdot_isotropic, it should correspond :) 
    b% jdot_ml = 0      !!!leave this like that and modify below
    b% jdot_ml = b% jdot_ml +  ...  !!!modify this

    ! Now add the L2 outflow contribution, which is not included in the default MESA jdot routine, so you will have to write it from scratch using the formula on the website!
    ! Watch out: in the formula, you will need to find the coordinate of L2 in units of separation, which is not a built-in MESA quantity, but you can calculate it using the function find_L2 that we provided as a gift!
    b% jdot_ml = b% jdot_ml +  ...     !!!modify this

end subroutine my_jdot_ml
```
{{< /details >}}

Read all the comments and try to fill in where you see `!!! modify this`.

  </div>
</details>

{{< details title="Solution for `my_jdot_ml`" closed="true" >}}
```fortran
subroutine my_jdot_ml(binary_id, ierr)
    use binary_def, only : binary_info, binary_ptr
    integer, intent(in) :: binary_id
    integer, intent(out) :: ierr
    type (binary_info), pointer :: b
    real(dp) :: x_L2
    ierr = 0
    call binary_ptr(binary_id, b, ierr)
    if (ierr /= 0) then
    write(*,*) 'failed in binary_ptr'
    return
    end if

    ! THIS IS WHERE YOU CAN IMPOSE THE ANGULAR MOMENTUM LOSS RATE 
    ! ASSOCIATED WITH MASS LOSS
    ! Remember that you have two types of mass loss: 
    ! the isotropic re-emission mode (beta) and the L2 outflow (upsilon).
    ! The first one is already implemented in MESA, 
    ! we just need to find how to write it.
    ! Look at the default routine that MESA uses in 
    ! $MESA_DIR/binary/private/binary_jdot.f90, and copy the relevant piece.
    ! You can also check the formula that we wrote on the website for Jdot_isotropic, it should correspond :) 
    b% jdot_ml = 0d0
    b% jdot_ml = b% jdot_ml + (b% mdot_system_transfer(b% a_i) + b% mdot_system_wind(b% a_i))*&
    pow2(b% m(b% d_i)/(b% m(b% a_i)+b% m(b% d_i))*b% separation)*2*pi/b% period *&
    sqrt(1 - pow2(b% eccentricity))

    ! Now add the L2 outflow contribution, which is not included in the default MESA jdot routine, so you will have to write it from scratch using the formula on the website!
    ! Calculate the coordinate of L2 in units of separation
    x_L2 = abs(find_L2(b))
    ! Add the contribution to jdot from mass lost from L2
    b% jdot_ml = b% jdot_ml + (b% mtransfer_rate * b% s_donor% x_ctrl(1))*&
    ((b% m(b% a_i)/(b% m(b% a_i)+b% m(b% d_i))-x_L2)*b% separation)**2*2*pi/b% period 

end subroutine my_jdot_ml
```
{{< /details >}}


<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Showing the L2 rate on <code>pgstar</code> </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Final touch: visualization of the (log of) the mass loss from $L_2$ in our `pgstar` window. We will need to modify the `data_for_extra_binary_history_columns` and `how_many_extra_binary_history_columns` in `run_binary_extras.f90` as usual.

  Don't forget that `b% mtransfer_rate` is negative, and is in cgs units. Invoke the constants `Msun` and `secyer`!

  </div>
</details>

{{< details title="Solution for `run_binary_extras.f90`" closed="true" >}}
```fortran
   subroutine data_for_extra_binary_history_columns(binary_id, n, names, vals, ierr)
      type(binary_info), pointer :: b
      integer, intent(in) :: binary_id
      integer, intent(in) :: n
      character(len=maxlen_binary_history_column_name) :: names(n)
      real(dp) :: vals(n)
      integer, intent(out) :: ierr
      ierr = 0
      call binary_ptr(binary_id, b, ierr)
      if (ierr /= 0) then
         write (*, *) 'failed in binary_ptr'
         return
      end if

      names(1) = 'tdelay(Gyr)'
      vals(1) = (5d0/256d0) * (clight**5 * b%separation**4) / &
      (standard_cgrav**3 * b%m(1) * b%m(2) * (b%m(1) + b%m(2)))

      ! convert from seconds -> years -> Gyr
      vals(1) = vals(1) / secyer / 1d9

      ! L2 mass outflow rate in Msun/yr
      names(2) = "lg_mdot_L2"
      ! Let's take the log of the absolute value 
      vals(2) = log10(abs((b% mtransfer_rate * b% s_donor% x_ctrl(1) )/Msun*secyer))

   end subroutine data_for_extra_binary_history_columns
```

```fortran
integer function how_many_extra_binary_history_columns(binary_id)
      use binary_def, only: binary_info
      integer, intent(in) :: binary_id
      how_many_extra_binary_history_columns = 2
   end function how_many_extra_binary_history_columns
```
{{< /details >}}

{{< details title="Solution for `inlist1`" closed="true" >}}
```fortran
! ADD THE L2 MASS OUTFLOW RATE TO THE HISTORY PANEL
History_Panels1_other_yaxis_name(2) = 'lg_mdot_L2'
```
{{< /details >}}



> [!WARNING]
> Don't forget to do `./clean` and `./mk` after modifying the `run_binary_extras.f90` file.

<div style="
  max-width: 600px;
  margin: 20px auto;
  border: 1px solid none;
  border-radius: 8px;
  background-color: #e8f6ff;
  overflow: hidden;
  color: black;
  font-family: sans-serif;
">

  <!-- Header -->
  <div style="
    background-color: #4fa2d9;
    color: black;
    font-weight: bold;
    text-align: center;
    padding: 8px;
  ">
    RUN 2 (7 minutes on 4 cores, 712 models)
  </div>

  <!-- Body -->
  <div style="
    padding: 15px;
    text-align: center;
  ">
    <p style="margin: 0;">
      Run your star + BH model with L2 outflow.<br>
      In case you need them, here are the complete inlists for this run:
      <a href="../lab3/stable_MT_L2_SOL.zip" download>
        <code>stable_MT_L2_SOL.zip</code>
      </a>
    </p>
  </div>

</div>

Your `pgstar` window should look like something like this (this is the very last model of your run, model 712):

<!-- ![pgstar_stable_caseB](/thursday/lab3/pgstar_stable_caseB_new.png) -->
<a id="fig-caseB"></a>
[![Case B figure](/thursday/lab3/pgstar_stable_caseB_new.png)](/thursday/lab3/pgstar_stable_caseB_new.png)

**Figure 3.** Stable mass transfer, Case B evolution for a star + BH binary (click to zoom in!).

- Make sure the mass loss rate from $L_2$ is appearing in your mass transfer rate plot. If it looks like <a href="#fig-caseB">Figure 3</a>, you must have done everything right 🍻🍻


### Analysis of the run: Case B mass transfer!
Here are some discussion points for you to understand what happened physically to your star + BH system; you will only need to look at <a href="#fig-caseB">Figure 3</a> (click to zoom in!). Try to think about it and answer together with your table.

1. Which type of mass transfer do you observe in this star + BH run?  
   {{< details title="Solution" closed="true" >}}
  The mass transfer starts after the Main Sequence → Case B!
  You can see it from the shape of the Hertzsprung-Russel diagram (ask your TA if you don't know!).
{{< /details >}}
2. How much mass did the donor star lose? How is it compared to the previous run that was assuming Eddington-limited accretion?
   {{< details title="Solution" closed="true" >}}
The donor star started from a mass of $40.8\:M_{\odot}$ (see [Table 1](#table-caseB)), and is now $24.77\:M_{\odot}$. You can read this value in the Text Summary (`star_mass`), or look at the Kippenhahn diagram. So it lost $16.03\:M_{\odot}$, which corresponds to 40% of its initial mass! More or less, like in the Case A system.
{{< /details >}}
3. How much mass did the BH accrete? Any difference with the previous run?
      {{< details title="Solution" closed="true" >}}
The BH accretor started from a mass of $17.14\:M_{\odot}$ (see [Table 1](#table-caseB)), and is now... $17.14\:M_{\odot}$ (see `star_2_mass` value)! It accreted absolutely nothing, as we wanted. Our setup is constructed such that $\upsilon+\beta=1$, i.e. 100% of the mass that the donor transfers is expelled from the binary.
{{< /details >}}
4. How much did the orbit shrink? Compare with the previous run.
      {{< details title="Solution" closed="true" >}}
The system started with a period of $32.2$ days. At the end of the mass transfer episode, the orbit is $3.0$ days wide. This amounts to 91% overall shrinkage! Quite wild, and for sure wilder than the Eddington-limited case.
{{< /details >}}
5. Will the system merge within the age of the Universe?
      {{< details title="Solution" closed="true" >}}
We have in this case a time delay of $7.38$ Gyr. So yes, another chirping binary!
{{< /details >}}
6. How is the final mass ratio of your BH + BH binary? Any difference with the previous run?
      {{< details title="Solution" closed="true" >}}
The donor will collapse into a BH of mass $24.77\:M_{\odot}$, and the companion BH has a mass of $17.14\:M_{\odot}$. The mass ratio is then $\sim 0.7$! Not much difference with the previous run, and still consistent with population synthesis studies of stable mass transfer 😌 
{{< /details >}}

<!-- #### ➕ BONUS: CASE B comparison!
Only if they had the time in minilab1 to do caseB. -->


## 2. Common envelope evolution
Common envelope (CE) evolution is a phase during which the envelope of the donor star engulfs the whole binary. The embedded system experiences strong drag forces while orbiting inside the envelope, causing the orbit to shrink and orbital energy to be transferred to the gas. Friction, shocks, and recombination energy are thought to help unbind part of the envelope, often leaving behind circumstellar material around the system. Observationally, CE events are frequently associated with luminous red novae (see [V1309 Sco](https://en.wikipedia.org/wiki/V1309_Scorpii), the “Rosetta stone” of this class of transients 💥).

<a id="eq-MKH"></a>
**CE occurs** when mass transfer becomes unstable. This can happen **in binaries with very extreme mass ratios** (i.e. the donor is much more massive than the accretor), or when the donor star is highly evolved and possesses a deep convective envelope. In these situations, mass transfer may initially proceed stably, but the donor envelope may not be able to shrink fast enough to remain inside its Roche lobe. The resulting runaway increase in mass-transfer rate creates a positive feedback loop: mass loss shrinks the Roche lobe and simultaneously drives further expansion of the donor envelope. There is currently no consensus on the condition for when this process becomes irreversible (i.e., **the actual CE onset**), but a rule of thumb is to **compare the mass-transfer rate $\dot{M}_{\mathrm{MT}}$ to the Kelvin–Helmholtz timescale $t_{\mathrm{KH}}$ of the donor** and define the onset of CE when

$$\dot{M}_{\mathrm{MT}}\gtrsim10 \times \frac{m_{\mathrm{donor}}}{t_{\mathrm{KH}}} .\,\tag{4}$$

The final outcome of CE evolution is highly uncertain because the process is intrinsically three-dimensional and hydrodynamical (thus computationally expensive to simulate). In some cases the binary merges completely if the inspiral is too strong; in others, **the envelope is successfully ejected and the binary survives with a much tighter orbit.** In literature, the final orbital separation post-CE, $a_{\mathrm{post-CE}}$, is commonly computed using the *energy formalism*[^ivanova2013]. In this framework, the binding energy of the donor envelope is assumed to be balanced by a fraction of the released orbital energy:

<a id="eq-PpostCE"></a>
$$E_{\mathrm{bind}}=\alpha_{\mathrm{CE}}\Delta E_{\mathrm{orb}}=\alpha_{\mathrm{CE}}\frac{G}{2}\left(\frac{(m_{\mathrm{donor,i}}-m_{\mathrm{env}})m_{\mathrm{accretor,f}}}{a_{\mathrm{post-CE}}}-\frac{m_{\mathrm{donor,i}}m_{\mathrm{accretor,i}}}{a_{\mathrm{i}}}\right) .$$

Solving for the post-CE separation gives

$$a_{\mathrm{post-CE}}=(m_{\mathrm{donor,i}}-m_{\mathrm{env}})m_{\mathrm{accretor,f}}\left(\frac{2E_{\mathrm{bind}}}{\alpha_{\mathrm{CE}}G}+\frac{m_{\mathrm{donor,i}}m_{\mathrm{accretor,i}}}{a_{\mathrm{i}}}\right)^{-1} ,\,\tag{5}$$

which is directly translatable into the post-CE period via the III Kepler's law:

$$P_{\mathrm{post-CE}}=2\pi\sqrt{\frac{a_{\mathrm{post-CE}}^3}{G\left[(m_{\mathrm{donor,i}}-m_{\mathrm{env}})+m_{\mathrm{accretor,f}}\right]}}\, . \tag{6}$$

Here:<a id="eq-ebind"></a>

- $m_{\mathrm{donor,i}}$ is the donor mass at CE onset, and $m_{\mathrm{env}}$ is the mass of the donor envelope, such that one can assume $m_{\mathrm{donor,i}} - m_{\mathrm{env}}=m_{\mathrm{He\:core}}$, i.e. the remaining Helium core of the donor star,
- $m_{\mathrm{accretor,i}}$ and $m_{\mathrm{accretor,f}}$ are the accretor masses at onset and post-CE, usually assumed to be equal: $m_{\mathrm{accretor,i}}=m_{\mathrm{accretor,f}}$,
- $a_{\mathrm{i}}$ is the orbital separation at CE onset,
- $a_{\mathrm{post-CE}}$ is the orbital separation post-CE,
- $G$ is the gravitational constant,
- $\alpha_{\mathrm{CE}}$ is the CE efficiency parameter, usually assumed to be $\alpha_{\mathrm{CE}}\simeq 1$,
- $E_{\mathrm{bind}}$ is the binding energy of the donor envelope. This can be
  estimated by integrating the gravitational and internal energy of the envelope layers:

  
  $$E_{\mathrm{bind}}=\int_{m_{\mathrm{He\:core}}}^{m_{\mathrm{donor}}}\left(-\frac{Gm}{r}+u-\epsilon_{\mathrm{diss}}\right)\, dm ,\,\tag{7}$$

  and $m$ is the mass enclosed within a shell, $r$ is the radius of the shell, $u$ is the specific internal energy, $\epsilon_{\mathrm{diss}}$ is the correction due to molecular hydrogen dissociation energy, $dm$ is the mass of the shell.


> [!IMPORTANT]
> In the context of gravitational wave sources, CE has been classically invoked as a way to form double BHs binaries, due to its efficient tightening of the orbit of star + BH systems prior to the evolution into BH + BH binaries. Pretty much as the stable mass transfer channels that we have seen above 😁 **The aim of this exercise is to explore how CE evolution can form gravitational wave sources and compare its outcome to the stable mass transfer channel**.
>
> You can start from the same setup as you developed for the Case A mass transfer (also downloadable <a href="../lab3/stable_MT_SOL.zip" download> <code>here</code></a>):
> ```bash
> cp -r stable_MT CE
>```
> Remind yourself of the properties of your Case A system in [Table 1](#table-binary). In particular, look at the mass ratio: $\sim 0.4$. This is a very mild mass ratio, and in fact the mass transfer between star + BH was stable! Let's **change the BH mass to $5\:M_{\odot}$: this gives us a very extreme mass ratio ($\sim 0.1$) that will favor CE evolution instead** 😎
> {{< details title="Solution for `inlist_project`" closed="true" >}}
>```fortran
>&binary_controls
>  ...
>  m1 = 39.6d0 ! donor mass in Msun
>  m2 = 5d0 ! companion mass in Msun
>  initial_period_in_days = 4.5d0
>  ...
>/ ! end of binary_controls namelist
>```
>{{< /details >}}

> [!CAUTION]
> In principle, if you ran your setup as is, it will work: the primary will evolve on its Main Sequence, initiate a Case A mass transfer episode, and reach very high mass transfer rates. MESA will start complaining with smaller and smaller timesteps, several retries, convergence issues. **This regime is not only numerically challenging and expensive for the solver, but also quite meaningless, as CE is inherently a 3D-hydro process** on the dynamical timescale and out of hydrostatic equilibrium!

> [!NOTE]
> 1. MESA actually has a suite of routines for modeling the CE stage in 1D in the most physically-motivated possible way (mostly following the formalism from Marchant et al. (2021)[^marchant2021])! We will not make use of these, but if you are curious, you can give a look at [last year's Summer School binary day](https://mesa-leuven.4d-star.org/tutorials/wednesday/lab-3). We will instead **stop our simulation at CE onset, and use the energy formalism to predict the post-CE properties of our binary.**
> 2. When you want to **pass information between `run_star_extras.f90` and `run_binary_extras.f90`** (for example, if you calculate something in `data_for_extra_history_columns` and you want to use that quantity in `data_for_extra_binary_history_columns`), **you can use the `s% xtra` array!**
> 3. If you don't know how quantities are called, you can check `$MESA_DIR/star_data/public/star_data_step_work.inc` for the `s%` structure, and `$MESA_DIR/binary/public/binary_data.inc` for the `b%` structure.

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>run_star_extras.f90</code>
  </div>

Calculate an extra history column `Ebind(erg)` for the binding energy $E_{\mathrm{bind}}$ of the hydrogen envelope of our star, in $\mathrm{erg}$ (<a href="#eq-ebind">Eq. (7)</a>). Then save its value into `s% xtra`.
</div>


<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Why is <code>run_star_extras.f90</code> empty...? </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  Your `run_star_extras.f90` looks kinda empty because it is running the standard routines in `standard_run_star_extras.inc`. We need to copy those routines in here and modify them. Do you remember where they are? You can always do a 
  
  ```bash
  `grep -nri standard_run_star_extras.inc $MESA_DIR/star`
  ```
  
  in your terminal and try to find the file yourself.

  </div>
</details>

{{< details title="Solution" closed="true" >}}
Copy the entire content of `$MESA_DIR/star/job/standard_run_star_extras.inc` in place of the line `include 'standard_run_star_extras.inc'`.
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Remember how to loop?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  You can loop over the star's shells with a fortran loop from `k=1` (surface) to `k=s% nz` (center). Watch out: in <a href="#eq-ebind">Eq. (7)</a> you have an integral from the (He-rich) core of the star to the surface, so you'll want your loop to go through only hydrogen-rich shells, to compute the `Ebind` of the envelope.

  You can check whether the shells are hydrogen-rich with something like `s% X(k) > 0.1d0`, where `s% X(k)` is the hydrogen mass fraction at the mass shell `k`.

  </div>
</details>

{{< details title="Skeleton of your loop" closed="true" >}}
```fortran
do k=1, s% nz
  if (s% X(k) > 0.1d0) then
      Ebind = ...
  else
      exit
  end if
end do
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(236, 72, 153, 0.14);
    border-left:4px solid rgba(236, 72, 153, 0.14);
  ">
    🎁 <strong>Gift: Hydrogen dissociation energy!</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(236, 72, 153, 0.14);
    border-left:4px solid rgba(236, 72, 153, 0.14);
  ">

  This is another gift for you (for the sake of time, but it is still an interesting exercise to get to know several MESA constants in `$MESA_DIR/const/public/const_def.f90`). This is how you can code the molecular hydrogen dissociation energy $\epsilon_{\mathrm{diss}}$ in the calculation of `Ebind(erg)`:

  ```fortran
  avo*4.52d0/2d0*ev2erg*s% X(k)
  ```

  {{< details title="Curious to understand why?" closed="true" >}}
The hydrogen dissociation energy contribution enters the integrand of <a href="#eq-ebind">Eq. (7)</a> as

$$\epsilon_{\mathrm{diss}}=\frac{N_A \, E_{\mathrm{H_2}}}{2}\, X ,$$

Here $E_{\mathrm{H_2}} = 4.52\,\mathrm{eV}$ is the dissociation energy of molecular hydrogen, $N_A$ (`avo`) is Avogadro’s number, `ev2erg` converts eV to erg, and $X = s\%X(k)$ is the hydrogen mass fraction in each zone. The factor $1/2$ accounts for the two hydrogen atoms per $\mathrm{H_2}$ molecule, giving the energy per gram of stellar material.
{{< /details >}}

  </details>


  </div>
</details>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Mind the units... </strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  You want <a href="#eq-ebind">Eq. (7)</a> to give you a quantity in $\mathrm{erg}$, the cgs unit for energy. So be sure to check all the units in `$MESA_DIR/star_data/public/star_data_step_work.inc`.

  </div>
</details>

{{< details title="Full solution for `Ebind(erg)`" closed="true" >}}
```fortran
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
  integer, intent(in) :: id, n
  character (len=maxlen_history_column_name) :: names(n)
  real(dp) :: vals(n)
  real(dp) :: Ebind
  integer :: k
  integer, intent(out) :: ierr
  type (star_info), pointer :: s
  ierr = 0
  call star_ptr(id, s, ierr)
  if (ierr /= 0) return

  ! note: do NOT add the extras names to history_columns.list
  ! the history_columns.list is only for the built-in history column options.
  ! it must not include the new column names you are adding here.

  names(1)="Ebind(erg)"
  Ebind = 0d0
  do k=1, s% nz
    if (s% X(k) > 0.1d0) then
        Ebind = Ebind + s% dm(k)*(-standard_cgrav*s% m(k)/s% r(k)+s% energy(k)-avo*4.52d0/2d0*ev2erg*s% X(k))
    else
        exit
    end if
  end do
  vals(1) = Ebind
  s% xtra(1) = Ebind

end subroutine data_for_extra_history_columns
```

```fortran
integer function how_many_extra_history_columns(id)
    integer, intent(in) :: id
    integer :: ierr
    type (star_info), pointer :: s
    ierr = 0
    call star_ptr(id, s, ierr)
    if (ierr /= 0) return
    how_many_extra_history_columns = 1
end function how_many_extra_history_columns
```

{{< /details >}}

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>run_binary_extras.f90</code>
  </div>

Let's implement a stopping condition at the onset of the common envelope episode, i.e. when the mass transfer rate exceeds $10\times\dot{M}_{\mathrm{KH}}$ (<a href="#eq-MKH">Eq. (4)</a>), and let's show the Kelvin-Helmholtz mass loss rate on the `pgstar` window together with `lg_mtransfer_rate`.
</div>

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>MESA already computes <code>kh_timescale</code> 🤓</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

   Surprise, you don't need to compute $t_{\mathrm{KH}}$ for <a href="#eq-MKH">Eq. (4)</a>, because there is an associated quantity to be found in `$MESA_DIR/star_data/public/star_data_step_work.inc`.  

  </div>
</details>

{{< details title="Solution" closed="true" >}}
It's called `s% kh_timescale`, and it's in years.
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Where do I put the stopping condition?</strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

   For a single star simulation, you would put it into the `extras_finish_step` routine... For a binary, it is very similar 🧠

  </div>
</details>

{{< details title="Full stopping condition" closed="true" >}}
```fortran
integer function extras_binary_finish_step(binary_id)
  type(binary_info), pointer :: b
  type (star_info), pointer :: s
  integer, intent(in) :: binary_id
  integer :: ierr
  call binary_ptr(binary_id, b, ierr)
  if (ierr /= 0) then  ! failure in  binary_ptr
      return
  end if
  extras_binary_finish_step = keep_going

  if (abs(b% mtransfer_rate)*secyer/Msun > 10d0*b% s_donor% m(1) /Msun / b% s_donor% kh_timescale) then
      extras_binary_finish_step = terminate
      write(*,*) "Terminate due to mdot>10*M_kh: CE onset!"
  end if

  end function extras_binary_finish_step
```
{{< /details >}}

<details>
  <summary style="
    cursor:pointer;
    padding:0.5rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">
    💡 <strong>Visualizing on <code>pgstar</code></strong>
  </summary>

  <div style="
    padding:0.75rem;
    background:rgba(246, 171, 59, 0.22);
    border-left:4px solid rgba(246, 171, 59, 0.22);
  ">

  We want to visualize the Kelvin-Helmholtz mass loss rate together with `lg_mtransfer_rate`, therefore we need to save the logarithm of $\dot{M}_{\mathrm{KH}}$ as an extra history column, called something like `log10(Mdot_KH)`.

  Additionally, we want to modify the `pgstar` namelist in `inlist1` in a clever spot. I would put it where you put the $L_2$ rate early on: search for the string "L2"

  </div>
</details>

{{< details title="Solution for `data_for_extra_binary_history_columns`" closed="true" >}}
Notice that we had already the calculation of `tdelay(Gyr)` from the first run. So you will have to also increase by 1 the count of `how_many_extra_binary_history_columns`.

```fortran
subroutine data_for_extra_binary_history_columns(binary_id, n, names, vals, ierr)
  type(binary_info), pointer :: b
  integer, intent(in) :: binary_id
  integer, intent(in) :: n
  character(len=maxlen_binary_history_column_name) :: names(n)
  real(dp) :: vals(n)
  real(dp) :: a_postCE, P_postCE
  integer, intent(out) :: ierr
  ierr = 0
  call binary_ptr(binary_id, b, ierr)
  if (ierr /= 0) then
      write (*, *) 'failed in binary_ptr'
      return
  end if

  names(1) = 'tdelay(Gyr)'
  vals(1) = (5d0/256d0) * (clight**5 * b%separation**4) / &
  (standard_cgrav**3 * b%m(1) * b%m(2) * (b%m(1) + b%m(2)))

  ! convert from seconds -> years -> Gyr
  vals(1) = vals(1) / secyer / 1d9

  ! KH rate threshold for CE onset
  names(2) = 'log10(Mdot_KH)'
  vals(2) = log10(b% s_donor% m(1) / Msun / b% s_donor% kh_timescale)

end subroutine data_for_extra_binary_history_columns
```
{{< /details >}}

{{< details title="Solution for `pgstar` in `inlist1`" closed="true" >}}
```fortran
History_Panels1_other_yaxis_name(2) = 'log10(Mdot_KH)'
```
{{< /details >}}

<div style="
  margin:1rem 0;
  padding:0.8rem 1rem;
  background:rgba(16,185,129,0.10);
  border-left:5px solid #10b981;
">

  <div style="font-weight:600; margin-bottom:0.5rem;">
    🧪 Task: Modify <code>run_binary_extras.f90</code>
  </div>

If you have time, try to implement:
1. ⭐️**BONUS**⭐️ $P_{\mathrm{post-CE}}$ in days (<a href="#eq-PpostCE">Eq. (6)</a>) as an extra history column `P_postCE(days)`, and show its value in the Text Summary window of `pgstar` → you will have to transport the `Ebind` information from `run_star_extras.f90` to `run_binary_extras.f90` with `s% xtra`!
2. ⭐️**BONUS**⭐️ $t_{\mathrm{delay,\:post-CE}}$ in Gyrs as an extra history column `tdelay_postCE(Gyr)`, and show its value in the Text Summary window of `pgstar`→ there's nothing difficult in this task, it is basically the same calculation as you did in [here](#computing-the-time-delay) for <a href="#eq-tdelay">Eq. (1)</a>. But this time, we want to use the masses and separation post-CE!
   
>[!CAUTION]
>🚨🚨 No problem if you don't have time to try, but still copy the full solution from here into your `run_binary_extras.f90`:
>{{< details title="Fully solved `data_for_extra_binary_history_columns`" closed="true" >}}
>```fortran
>subroutine data_for_extra_binary_history_columns(binary_id, n, names, vals, ierr)
>  type(binary_info), pointer :: b
>  integer, intent(in) :: binary_id
>  integer, intent(in) :: n
>  character(len=maxlen_binary_history_column_name) :: names(n)
>  real(dp) :: vals(n)
>  real(dp) :: a_postCE, P_postCE
>  integer, intent(out) :: ierr
>  ierr = 0
>  call binary_ptr(binary_id, b, ierr)
>  if (ierr /= 0) then
>      write (*, *) 'failed in binary_ptr'
>      return
>  end if
>
>  names(1) = 'tdelay(Gyr)'
>  vals(1) = (5d0/256d0) * (clight**5 * b%separation**4) / &
>  (standard_cgrav**3 * b%m(1) * b%m(2) * (b%m(1) + b%m(2)))
>
>  ! convert from seconds -> years -> Gyr
>  vals(1) = vals(1) / secyer / 1d9
>
>  ! KH rate threshold for CE onset
>  names(2) = 'log10(Mdot_KH)'
>  vals(2) = log10(b% s_donor% m(1) / Msun / b% s_donor% kh_timescale)
>
>  ! POST-COMMON ENVELOPE period
>  ! We will use the energy formalism to compute the post-CE separation a_postCE (in centimeters, for convenience!), and then convert it into the post-CE period P_postCE, in days.
>  names(3) = 'P_postCE(days)'
>  ! Post-CE orbital separation in cm
>  a_postCE = (b% s_donor% he_core_mass * Msun) * b% m(2) / &
>      ( (2d0 * abs(b% s_donor% xtra(1))) / standard_cgrav + &
>      (b% m(1) * b% m(2)) / b% separation )
>
>  ! Post-CE orbital period in days
>  P_postCE = 2d0*pi * sqrt( a_postCE**3 / &
>      ( standard_cgrav * (b% s_donor% he_core_mass * Msun + b% m(2)) ) ) / secday
>  vals(3) = P_postCE
>
>  ! POST-COMMON ENVELOPE TIME DELAY
>  ! The formula is the same that you implemented already before, but this time we will use the post-CE separation (just computed above), the mass of the BH which stays the same, and the mass of the stripped star which is now the core mass.
>  names(4) = 'tdelay_postCE(Gyr)'
>  vals(4) = (5d0/256d0) * (clight**5 * a_postCE**4) / &
>    (standard_cgrav**3 * b% s_donor% he_core_mass * Msun * b%m(2) * (b% s_donor% he_core_mass * Msun + b%m(2)))
>  ! convert from seconds -> years -> Gyr
>  vals(4) = vals(4) / secyer / 1d9
>
>end subroutine data_for_extra_binary_history_columns
>```
>{{< /details >}}
> and include the relevant columns in your `pgstar` inlist in `inlist1`:
> {{< details title="Add `tdelay_postCE(Gyr)` and `P_postCE(days)`" closed="true" >}}
>```fortran
> Text_Summary1_name(7,4) = 'P_postCE(days)'
>
> ! ADD THE TDELAY TO THE TEXT SUMMARY
> Text_Summary1_name(8,4) = 'tdelay_postCE(Gyr)'
>```
>{{< /details >}}
> and bring up this count: `how_many_extra_binary_history_columns = 4`.

</div>


> [!WARNING]
> Never forget to do `./clean` and `./mk` after modifying the `run_binary_extras.f90` file.

Well done, we're at our third and last run of the day!

<div style="
  max-width: 600px;
  margin: 20px auto;
  border: 1px solid none;
  border-radius: 8px;
  background-color: #cdffea;
  overflow: hidden;
  color: black;
  font-family: sans-serif;
">

  <!-- Header -->
  <div style="
    background-color: #4fd99f;
    color: black;
    font-weight: bold;
    text-align: center;
    padding: 8px;
  ">
    RUN 3 (2 minutes on 4 cores, 560 models)
  </div>

  <!-- Body -->
  <div style="
    padding: 15px;
    text-align: center;
  ">
    <p style="margin: 0;">
      Run your common envelope model with <code>./rn | tee output.txt</code>.<br>
      In case you need them, here are the complete inlists for this run:
      <a href="../lab3/CE_SOL.zip" download>
        <code>CE_SOL.zip</code>
      </a>
    </p>
  </div>

</div>

Your `pgstar` window should look like something like this (this is the very last model, right when CE starts according to our implemented criterion of <a href="#eq-MKH">Eq. (4)</a>, model number 560):

<a id="fig-CEcaseA"></a>
[![CE case A figure](/thursday/lab3/pgstar_CE_caseA_new.png)](/thursday/lab3/pgstar_CE_caseA_new.png)

**Figure 4.** Common envelope evolution at its onset for a star + BH binary (click to zoom in!).

- Make sure that **the Kelvin-Helmholtz rate `log10(Mdot_KH)`** is appearing in the plot of `lg_mtransfer_rate`. You can see that the threshold stays around $10^{-2}\:M_{\odot}\:\mathrm{yr}^{-1}$, which gets easily surpassed by our mass transfer episode after a few models. 
- Make sure also the new **Text Summary information on $t_{\mathrm{delay\: post-CE}}$ and $P_{\mathrm{post-CE}}$** from the bonus tasks are appearing: `tdelay_postCE(Gyr)` and `P_postCE(days)`. If you don't see them, you must have missed something, but no worries. It was a long implementation! You can try to fix it, or just go to the "Analysis of the run" section and simply look at <a href="#fig-CEcaseA">Figure 4</a> (click to zoom in!) to answer to the conceptual questions.

### Analysis of the run: runaway mass transfer!
Here are some discussion points; you will only need to look at <a href="#fig-CEcaseA">Figure 4</a> (click to zoom in!). Try to think about it and answer together with your table.

1. How is the mass transfer rate evolving, and how can you see that you are at CE onset?
   {{< details title="Solution" closed="true" >}}
The mass transfer rate is on a steep journey of ever-growing disaster 😨 You can see it from the `lg_mtransfer_rate` plot, where also we have highlighted the `log10(Mdot_KH)` to show an indication of the timescale over which the star as whole fwould be able to relax thermally. 

You can also see it from the `rl_relative_overflow` plot, in which you see a plateaux at a value above 1 (i.e. the radius of the star keeps being bigger than its Roche Lobe).
{{< /details >}}

Assume now that, after CE, your system survives as a binary, and the envelope of the stripped star is fully lifted out of the system. Further, the star will quickly evolve into a BH with mass equal to its own Helium core mass. According to the energy formalism that we used to compute the post-CE properties,

2. Does this star + BH system produce a gravitational wave source?
      {{< details title="Solution" closed="true" >}}
Indeed, since the `tdelay_postCE` amounts to only 30 000 years. This will merge fast 😵‍💫
      {{< /details >}}

3. Is the post-CE orbital period tighter in this case, with respect to the case of stable mass transfer?
   {{< details title="Solution" closed="true" >}}
Definitely tighter. In here we have something like $\sim 0.02\:\mathrm{days}=30\:\mathrm{minutes}$! As compared to the stable mass transfer cases, we have 2 order of magnitudes difference. 
      {{< /details >}}

4. Does this post-CE orbit make sense, i.e. could our two BHs actually fit in it?
      {{< details title="Solution" closed="true" >}}
Yes! A binary composed of black holes with masses  $21.94\,M_\odot$ and $M_2 = 5\,M_\odot$ can physically exist with an orbital period of $P = 0.02$ days. Using Kepler’s third law,

$$
a^3 = \frac{G(M_1+M_2)P^2}{4\pi^2} \approx 0.93\,R_\odot.
$$

This means the two black holes orbit at a separation comparable to the radius of the Sun 🌞 For comparison, the Schwarzschild radii (a sort of indication of the BHs' size, $\propto \frac{2GM}{c^2}$) are much smaller:

- $\sim 65$ km for the $21.94\,M_\odot$ BH  
- $\sim 15$ km for the $5\,M_\odot$ BH 
      {{< /details >}}

1. Is the final mass ratio of your BH + BH binary different with respect to the stable mass transfer case?
   {{< details title="Solution" closed="true" >}}
The answer is yes again: the mass ratio in this case would be $\sim 0.23$, much more extreme than in the stable mass transfer case.
      {{< /details >}}



## 3. Conclusions
Congratulations for making it till here! 🥳🥳 In this last lab we have completed our overview of binary evolution from Zero Age Main Sequence stars to binary black holes. We have learned three possible ways to form gravitational wave sources from the star + BH configuration: Case A stable mass transfer, Case B stable mass transfer, and common envelope evolution, and all three possibilites have been shown to contribute to the observed sample of gravitational waves detected by LIGO, Virgo and KAGRA interferometers[^GWTC4]. 

>[!IMPORTANT]
>Whether or not the relationship between star + BH remains stable determines different properties in the BH + BH resulting binary, and singling out the fingerprint of the different channels is still a hot topic: this is how the gravitational wave sources that we see today can teach us something about the (love and hate!) history of their parent stellar progenitors ♥️

## References
[^peters1964]: [Peters (1964), Gravitational Radiation and the Motion of Two Point Masses](https://ui.adsabs.harvard.edu/abs/1964PhRv..136.1224P)
[^SS433]: [Wikipedia — SS433](https://en.wikipedia.org/wiki/SS_433)
[^GWTC4]: [The LIGO Scientific Collaboration, the Virgo Collaboration, the KAGRA Collaboration, et al. (2025a), GWTC-4.0: Updating the Gravitational-Wave Transient Catalog with Observations from the First Part of the Fourth LIGO-Virgo-KAGRA Observing Run](https://arxiv.org/abs/2508.18082)
[^lu2022]: [Lu et al. (2022), Stable mass transfer via L2 outflows in massive binaries](https://ui.adsabs.harvard.edu/abs/2023MNRAS.519.1409L)
[^marchant2021]: [Marchant et al. (2021), The role of mass transfer and common envelope evolution in the formation of merging binary black holes](https://ui.adsabs.harvard.edu/abs/2021A&A...650A.107M)
[^bowler2010]: [Bowler (2010), Interpretation of observations of the circumbinary disk of SS 433](https://ui.adsabs.harvard.edu/abs/2010A%26A...521A..81B)
[^ivanova2013]: [Ivanova et al. (2013), Common envelope evolution: where we stand and how we can move forward](https://ui.adsabs.harvard.edu/abs/2013A&ARv..21...59I)

<!-- #### ➕ BONUS1: CASE B comparison!
Only if they had the time in minilab1 to do caseA.

#### ➕➕ BONUS2: Delayed mass transfer instability
Have them look into the timescale of when instability develops, and the shift in properties (mass and orbital separation) from RLOF to CE onset -->


</div>
