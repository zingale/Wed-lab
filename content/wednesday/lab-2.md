---
weight: 2
author: TODO
math: true
disableKinds: "rss"
---
# Minilab 2: Getting in Debt

## Introduction

In Lab 1 you built a minimal nuclear network for an accreting ONe white dwarf and watched the core march toward oxygen ignition.
But there is important physics we left out: **the URCA process**.

In sufficiently degenerate matter, certain nuclei can undergo cyclic electron captures and beta decays at a specific **threshold density** — a so-called *Urca shell*.
Each cycle emits two neutrinos that carry energy directly out of the star, providing a potentially significant cooling (or heating) mechanism that depends sensitively on the accretion rate.

In this lab you will:
1. Add the **A=23 Urca pair** (Na²³ ↔ Ne²³) to your nuclear network and observe the Urca shell in real time with pgstar.
2. Add the **A=25 Urca pair** (Mg²⁵ → Na²⁵ → Ne²⁵) and compare its effect against the A=23 run using a Google Colab notebook.
3. Estimate the **compressional heating and Urca cooling timescales** of the white dwarf.

### Helpful Links

The general Google Drive for these Wednesday labs can be found [HERE]( FIXLINK ).

More specifically, the files for Lab 2 can be found [HERE]( FIXLINK ).
The drive contains the starting point, partial solutions, and a full solution.

Consult the [MESA documentation](https://docs.mesastar.org/en/latest/) throughout this lab.

> [!NOTE]
> As in Lab 1, tasks are formatted as:
>
> | 📋 TASK N |
> |:--------|
> | (task description) |
>
> Values that need to be altered in the files are generally marked with `!!!!!`.

---

## Part 1: Adding the A=23 Urca Pair (Na²³/Ne²³)

### Step 0: Start Up

| 📋 TASK 1 |
|:--------|
| **Download** the Lab 2 starting point from the [Google Drive]( FIXLINK ) to a local working directory. |

The starting point is your completed Lab 1 setup, now configured to load a 1.1 M<sub>&#9737;</sub> O-Ne-Na white dwarf model.

After downloading, your working directory should look like:

{{< filetree/container >}}
  {{< filetree/folder name="Starting Point" >}}
    {{< filetree/file name="clean" >}}
    {{< filetree/file name="mk" >}}
    {{< filetree/file name="re" >}}
    {{< filetree/file name="rn" >}}
    {{< filetree/file name="history_columns.list" >}}
    {{< filetree/file name="profile_columns.list" >}}
    {{< filetree/file name="inlist" >}}
    {{< filetree/file name="inlist_common" >}}
    {{< filetree/file name="inlist_accrete" >}}
    {{< filetree/file name="inlist_pgstar" >}}
    {{< filetree/file name="ONe.net" >}}
    {{< filetree/file name="1.1Msun_ONeNa.mod" >}}
    {{< filetree/folder name="src" state="open" >}}
      {{< filetree/file name="run.f90" >}}
      {{< filetree/file name="run_star_extras.f90" >}}
    {{< /filetree/folder >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

> [!NOTE]
> `run_star_extras.f90` has been updated from Lab 1. It now computes the A=23 electron capture and beta decay rates ($\lambda_{\rm Na23 \to Ne23}$ and $\lambda_{\rm Ne23 \to Na23}$) as extra profile columns used by pgstar to show the Urca shell.


### Step 1: Build the ONeNa Network

The starting network (`ONe.net`) contains the Ne²⁰→F²⁰→O²⁰ electron capture chain and O¹⁶ burning, but no Na²³ or Ne²³.
To model the A=23 Urca pair we need to add both isotopes and their connecting weak reactions.

| 📋 TASK 2 |
|:--------|
| **Create a new file `ONeNa.net`** in your working directory. Starting from `ONe.net`, **add Na²³, Ne²³** and the two Urca reactions that connect them. |

> [!NOTE]
> The A=23 Urca pair consists of:
> - Electron capture: $^{23}_{11}\mathrm{Na} + e^- \to {^{23}_{10}\mathrm{Ne}} + \nu_e$
> - Beta decay: $^{23}_{10}\mathrm{Ne} \to {^{23}_{11}\mathrm{Na}} + e^- + \bar{\nu}_e$

{{< details title="Hint: What isotopes and reactions to add?" closed="true" >}}
Add to `add_isos(...)`:
- `na23`
- `ne23`

Add to `add_reactions(...)`:
- `r_na23_wk_ne23`  — electron capture Na23 + e⁻ → Ne23
- `r_ne23_wk-minus_na23` — beta decay Ne23 → Na23 + e⁻
{{< /details >}}

{{< details title="Partial Solution" closed="true" >}}
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20 electron capture chain
    ne20
    f20
    o20
    ! for A=23 Urca pair
    na23
    ne23
    ! for O ignition
    si28
    )

add_reactions(
    ! for oxygen ignition
    r1616
    ! for Ne20 - F20 - O20
    r_ne20_wk_f20
    r_f20_wk-minus_ne20
    r_f20_wk_o20
    r_o20_wk-minus_f20
    ! A=23 Urca pair: Na23 + e- <-> Ne23
    r_na23_wk_ne23
    r_ne23_wk-minus_na23
    )
```
{{< /details >}}


### Step 2: Update the Inlists

With the network in hand, update the inlists to use it and include Na²³ in the accreted material.

| 📋 TASK 3 |
|:--------|
| In `inlist_accrete`, **change the network** to `ONeNa.net` and **add Na²³** to the accreted composition. Use a mass fraction of 5% Na²³ (reducing Ne²⁰ from 50% to 45%). |

{{< details title="Hint: What variables need to be changed?" closed="true" >}}
In `&star_job`:
- `new_net_name = 'ONeNa.net'`

In `&controls`, update the accretion block:
- `num_accretion_species = 3`
- Add `accretion_species_id(3) = 'na23'` and `accretion_species_xa(3) = 0.05d0`
- Adjust Ne²⁰ fraction so that all fractions sum to 1.
{{< /details >}}

{{< details title="Partial Solution" closed="true" >}}
```fortran
! in &star_job
    new_net_name = 'ONeNa.net'   !!!!!

! in &controls
    num_accretion_species = 3           !!!!!
    accretion_species_id(1) = 'o16'
    accretion_species_xa(1) = 0.50d0
    accretion_species_id(2) = 'ne20'
    accretion_species_xa(2) = 0.45d0   !!!!!
    accretion_species_id(3) = 'na23'   !!!!!
    accretion_species_xa(3) = 0.05d0   !!!!!
```
{{< /details >}}

> [!WARNING]
> Remember to do `./clean && ./mk` after any change to a network file or `run_star_extras.f90`.


### Step 3: Run and Observe the A=23 Urca Shell

| 📋 TASK 4 |
|:--------|
| Compile and run with `./mk` then `./rn`. In the pgstar window labelled **"A=23 Urca Shell"**, observe the composition profile and the neutrino emissivity as the core density increases. |

You should see two windows open: the main grid overview (`Grid2`) and the Urca shell profile plot (`Profile_Panels1`).

In `Profile_Panels1`, look for:
- **Top panel** — the electron capture rate ($\lambda_{\rm Na23 \to Ne23}$, blue) and beta decay rate ($\lambda_{\rm Ne23 \to Na23}$, red) as functions of $\log\rho$.
  The **Urca shell** is located where these two rates are equal ($\lambda_{\rm EC} = \lambda_{\rm BD}$), corresponding to the threshold density.
- **Bottom panel** — the neutrino emissivity (`eps_nuc_neu_total`, solid) and thermal neutrino emission (`non_nuc_neu`, dashed).
  A localised peak in the nuclear neutrino emissivity marks the active Urca shell.

> [!NOTE]
> At what log density does the A=23 Urca shell sit? You can compare this to the theoretical threshold:
> $$\log_{10} \rho_{\rm thresh} \approx 9.0 \quad (^{23}\mathrm{Na}/^{23}\mathrm{Ne})$$

{{< details title="What if pgstar shows nothing in the Profile_Panels1 window?" closed="true" >}}
The Urca shell only activates once the central density exceeds the threshold (~10⁹ g cm⁻³).
Be patient — it can take several hundred model steps for accretion to compress the core to this density.
Check `log_cntr_Rho` in the text summary to track progress.
{{< /details >}}

> [!NOTE]
> If you want to see the final pgstar frame before MESA closes, you can add `pause_before_terminate = 'press enter to continue'` to the `&controls` section of `inlist_common`.

---

## Part 2: Adding the A=25 Urca Pair (Mg²⁵/Na²⁵/Ne²⁵)

The A=25 Urca pair operates at a higher threshold density than A=23.
It consists of a two-step electron capture chain:
$$^{25}_{12}\mathrm{Mg} + e^- \to {^{25}_{11}\mathrm{Na}} + \nu_e \quad \text{then} \quad {^{25}_{11}\mathrm{Na}} + e^- \to {^{25}_{10}\mathrm{Ne}} + \nu_e$$
and the reverse beta decays.


### Step 4: Build the ONeNaMg25 Network

| 📋 TASK 5 |
|:--------|
| **Create a new file `ONeNaMg25.net`** by extending `ONeNa.net` to include the A=25 species and reactions. Also add Mg²⁴ as a tracked species (it is accreted). |

{{< details title="Hint: What isotopes and reactions to add?" closed="true" >}}
Additional isotopes:
- `mg25`, `na25`, `ne25`
- `mg24` (tracked; no new Urca reactions needed for A=24 at this stage)

Additional reactions:
- `r_mg25_wk_na25`       — EC: Mg25 + e⁻ → Na25
- `r_na25_wk-minus_mg25` — BD: Na25 → Mg25 + e⁻
- `r_na25_wk_ne25`       — EC: Na25 + e⁻ → Ne25
- `r_ne25_wk-minus_na25` — BD: Ne25 → Na25 + e⁻
{{< /details >}}

{{< details title="Partial Solution" closed="true" >}}
```fortran
add_isos(
    h1
    he4
    o16
    ne20
    f20
    o20
    ! A=23 Urca pair
    na23
    ne23
    ! A=25 Urca pair
    mg25
    na25
    ne25
    ! other accreted species
    mg24
    ! for O ignition
    si28
    )

add_reactions(
    r1616
    ! Ne20 - F20 - O20
    r_ne20_wk_f20
    r_f20_wk-minus_ne20
    r_f20_wk_o20
    r_o20_wk-minus_f20
    ! A=23 Urca pair
    r_na23_wk_ne23
    r_ne23_wk-minus_na23
    ! A=25 Urca pair
    r_mg25_wk_na25
    r_na25_wk-minus_mg25
    r_na25_wk_ne25
    r_ne25_wk-minus_na25
    )
```
{{< /details >}}


### Step 5: Update the Inlists for the A=25 Run

| 📋 TASK 6 |
|:--------|
| Update `inlist_accrete` to use `ONeNaMg25.net`, load the `1.1Msun_ONeMg2Na.mod` starting model, set the LOGS directory to `LOGS_ONeNaMg25_1d-6`, and update the accretion composition to include Mg²⁴ (5%) and Mg²⁵ (1%). |

{{< details title="Partial Solution" closed="true" >}}
```fortran
! in &star_job
    load_model_filename = '../make_one_wd/1.1Msun_ONeMg2Na.mod'  !!!!!
    new_net_name = 'ONeNaMg25.net'                                !!!!!

! in &controls
    num_accretion_species = 5           !!!!!
    accretion_species_id(1) = 'o16'
    accretion_species_xa(1) = 0.50d0
    accretion_species_id(2) = 'ne20'
    accretion_species_xa(2) = 0.39d0   !!!!!
    accretion_species_id(3) = 'na23'
    accretion_species_xa(3) = 0.05d0
    accretion_species_id(4) = 'mg24'   !!!!!
    accretion_species_xa(4) = 0.05d0   !!!!!
    accretion_species_id(5) = 'mg25'   !!!!!
    accretion_species_xa(5) = 0.01d0   !!!!!

    log_directory = 'LOGS_ONeNaMg25_1d-6'  !!!!!
```
{{< /details >}}

> [!NOTE]
> `run_star_extras.f90` already computes the A=25 lambda rates, so the A=25 pgstar panel (`Profile_Panels2`) will appear automatically.
> You do **not** need to recompile unless you changed the Fortran source.


### Step 6: Run and Compare with pgstar

| 📋 TASK 7 |
|:--------|
| Run the A=25 case (`./rn`) and observe **both** `Profile_Panels1` (A=23 shell) and `Profile_Panels2` (A=25 shell). |

Look for:
- A second Urca shell appearing at a **higher density** than the A=23 shell.
- Any change in the neutrino emissivity profile — does the A=25 shell contribute noticeably?

> [!NOTE]
> Theoretical threshold density for the A=25 shell:
> $$\log_{10} \rho_{\rm thresh} \approx 9.3 \quad (^{25}\mathrm{Mg}/^{25}\mathrm{Na})$$


### Step 7: Notebook Comparison

| 📋 TASK 8 |
|:--------|
| Open the **Google Colab notebook** ([link]( FIXLINK )) and upload your two LOGS directories (`LOGS_ONeNa_1d-6` and `LOGS_ONeNaMg25_1d-6`). Follow the notebook to plot the central temperature, density, and neutrino luminosity from both runs. |

The notebook guides you through:
1. Plotting central T and ρ evolution for both runs.
2. Comparing neutrino luminosities.
3. Estimating compressional heating and Urca cooling timescales (see Part 3 below).
4. Profile snapshots of the Na²³/Ne²³ and Mg²⁵/Na²⁵ mass fractions at the Urca shells.

---

## Part 3: Compressional Heating and Urca Cooling Timescales

As the white dwarf accretes mass, compressional work heats the core.
The Urca shells remove this energy via neutrinos.
The competition between these rates governs the thermal evolution and ultimately the ignition conditions.

The relevant timescales are:

$$\tau_{\rm heat} = \frac{E_{\rm th}}{L_{\rm comp}} \approx \frac{k_B T_c \, M_{\rm WD}/m_u}{G M_{\rm WD} \dot{M} / R_{\rm WD}}$$

$$\tau_{\rm cool} = \frac{E_{\rm th}}{L_\nu}$$

where $L_\nu$ is the total neutrino luminosity from the Urca shells.

| 📋 TASK 9 |
|:--------|
| Using the notebook (Section 3), compute $\tau_{\rm heat}$ and $\tau_{\rm cool}$ at a late stage of your run. Which timescale is shorter? What does this imply for the white dwarf's thermal state as it approaches oxygen ignition? |

{{< details title="Discussion hint" closed="true" >}}
If $\tau_{\rm cool} \ll \tau_{\rm heat}$, the Urca shells cool the WD faster than compressional work can heat it — the core stays cold, favouring core collapse (cECSNe).

If $\tau_{\rm heat} \ll \tau_{\rm cool}$, the WD heats up, favouring thermonuclear runaway (ECSNe).

This transition is sensitive to $\dot{M}$ and the Urca pair threshold densities — exactly the crowdsourcing exercise you will explore in Lab 3!
{{< /details >}}

---

## Solution / End Point

The full solution for Lab 2 (which also serves as the starting point for Lab 3) can be downloaded [HERE]( FIXLINK ).

[^1]: Suzuki et al. 2016, ApJ 817, 163 — sd-shell electron capture and β-decay rates at stellar densities.
