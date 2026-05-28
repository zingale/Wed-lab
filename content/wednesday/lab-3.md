---
weight: 3
author: Tryston Raecke, Josh Wanninger, Sunny Wong, Michael Zingale
math: true
disableKinds: "rss"
---
# Minilab 3: They all go broke

So far we have changed the nuclear net to include more reactions, and looked at the effect of Urca cooling from the $^{23}\rm{Na}$-$^{23}\rm{Ne}$ pair on the stellar structure. 

We have been using an accretion rate $\dot{M}= 10^{-6} M_{\odot} \rm{yr}^{-1}$ and weak reaction rates from Suzuki et al. 2016. But what if we have different accretion histories, or reaction rates? 

Now we will do a crowdsourcing to look at how the evolution changes with the accretion rate $\dot{M}$, reaction networks, and reaction rates. 
The goal is to look at how they change the core properties at the onset of oxygen ignition, because whether an electron-capture supernova undergoes a thermonuclear explosion or core-collapse (implosion) is extremely sensitive to the central density. 

## Crowdsourcing

### Step 0: Start up

| 📋 TASK 1 |
|:--------|
| **Download** the starting point from the [Google Drive]( https://drive.google.com/file/d/1T7yvdHgni1wipdopA925505jwcPr-MNk/view?usp=drive_link ) to a local working directory. |

The starting point is a very simple setup. It should look like:

{{< filetree/container >}}
  {{< filetree/folder name="lab3_start_point" >}} 
    {{< filetree/file name="inlist" >}} 
    {{< filetree/file name="inlist_common" >}} 
    {{< filetree/file name="inlist_accrete" >}} 
    {{< filetree/file name="inlist_net" >}} 
    {{< filetree/file name="inlist_rates" >}} 
    {{< filetree/file name="other things" >}} 
  {{< /filetree/folder >}}
{{< /filetree/container >}}

>[!NOTE]
> In this lab, you will only need to edit `inlist_accrete`, `inlist_net`, and `inlist_rates`. 

### Step 1: Pick a model

| 📋 TASK 2 |
|:--------|
|  Go to the spreadsheet [here]( https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0 ). Pick any combination of the accretion rate, reaction network and reaction rates provided. Users with more cores should pick more computationally expensive ones. |


### Step 2: Changing the accretion rate

| 📋 TASK 3 |
|:--------|
| **Edit `inlist_accrete`** to set the accretion rate that you chose. |



{{< details title="Hint: what inlist option needs to be changed?" closed="true" >}}

This is called `mass_change` in the `&controls` section. 

{{< /details >}}



{{< details title="Partial solution" closed="true" >}}

In `&controls` of your `inlist_accrete`, set `mass_change = <your value>`. 

{{< /details >}}

### Step 3: Set your network

You've done great work in labs 1 and 2 to implement custom networks, so here we will just supply the networks need. 

| 📋 TASK 3 |
|:--------|
| **Edit `inlist_net`** to have it use your specific network, which we supply in **`nets_lab3`**. |

{{< details title="Hint: which inlist options?" closed="true" >}}
You can easily search for this: 
```fortran
grep -r net $MESA_DIR/star/defaults
```
{{< /details >}}

{{< details title="Hint: partial solutions" closed="true" >}}
Add the following in your ``inlist_net``: 
```fortran
change_initial_net = .true.
new_net_name = 'nets_lab3/<name>.net'
```
{{< /details >}}

#### What are in these nets?

> [!NOTE]
> In ``ONeMg.net``, ``ONeMgNa.net`` and ``ONeMg2Na.net``, we added the $^{24}\rm{Mg}$ electron capture reactions, which are exothermic. 

{{< tabs items="ONe.net,ONeMg.net,ONeNa.net,ONeMgNa.net,ONeMg2Na.net" >}}

<!-- ONe.net -->
{{< tab name='ONe.net' >}}

Species: ${^{1}\rm{H}}$, ${^{4}\rm{He}}$, ${^{16}\rm{O}}$, ${^{20}\rm{Ne}}$, ${^{20}\rm{F}}$, ${^{20}\rm{O}}$, ${^{23}\rm{Na}}$, ${^{24}\rm{Mg}}$, ${^{25}\rm{Mg}}$, ${^{28}\rm{Si}}$

Reactions: 
- **[Oxygen burning]** ${^{16}\rm{O}} + {^{16}\rm{O}} \to \rm{products}$ (specifically, the simplified reaction ```r1616```)
- **[Ne20 EC chain]** ${^{20}\rm{Ne}} + {e^{-}} \to {^{20}\rm{F}} + \nu_{e}$, ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[Ne20 EC chain]** ${^{20}\rm{F}} + {e^{-}} \to {^{20}\rm{Ne}} + \nu_{e}$, ${^{20}\rm{O}} \to {^{20}\rm{F}} + {e^{-}} + \bar{\nu}_{e}$

{{< details title="Implementation of ``ONe.net`` (Not a task)" closed="true" >}}
Your net should have the following: 
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20
    ne20
    f20
    o20
    ! for other accreted species
    na23
    mg24
    mg25
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
)
```

{{< /details >}}

{{< /tab >}}

<!-- ONeMg.net -->
{{< tab name="ONeMg.net" >}}
Species: ${^{1}\rm{H}}$, ${^{4}\rm{He}}$, ${^{16}\rm{O}}$, ${^{20}\rm{Ne}}$, ${^{20}\rm{F}}$, ${^{20}\rm{O}}$, ${^{23}\rm{Na}}$, ${^{24}\rm{Mg}}$, ${^{24}\rm{Na}}$, ${^{24}\rm{Ne}}$, ${^{25}\rm{Mg}}$, ${^{28}\rm{Si}}$

Reactions: 
- **[Oxygen burning]** ${^{16}\rm{O}} + {^{16}\rm{O}} \to \rm{products}$ (specifically, the simplified reaction ```r1616```)
- **[Ne20 EC chain]** ${^{20}\rm{Ne}} + {e^{-}} \to {^{20}\rm{F}} + \nu_{e}$, ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[Ne20 EC chain]** ${^{20}\rm{F}} + {e^{-}} \to {^{20}\rm{Ne}} + \nu_{e}$, ${^{20}\rm{O}} \to {^{20}\rm{F}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Mg}} + {e^{-}} \to {^{24}\rm{Na}} + \nu_{e}$, ${^{24}\rm{Na}} \to {^{24}\rm{Mg}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Na}} + {e^{-}} \to {^{24}\rm{Ne}} + \nu_{e}$, ${^{24}\rm{Ne}} \to {^{24}\rm{Na}} + {e^{-}} + \bar{\nu}_{e}$

{{< details title="Implementation of ``ONeMg.net`` (Not a task)" closed="true" >}}
Your net should have the following: 
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20
    ne20
    f20
    o20
    ! for Mg24 - Na24 - Ne24
    mg24
    na24
    ne24
    ! for other accreted species
    na23
    mg25
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
    ! for Mg24 - Na24 - Ne24
    r_mg24_wk_na24
    r_na24_wk-minus_mg24
    r_na24_wk_ne24
    r_ne24_wk-minus_na24
)
```

{{< /details >}}
{{< /tab >}}

<!-- ONeNa.net -->
{{< tab name="ONeNa.net" >}}

Species to include: ${^{1}\rm{H}}$, ${^{4}\rm{He}}$, ${^{16}\rm{O}}$, ${^{20}\rm{Ne}}$, ${^{20}\rm{F}}$, ${^{20}\rm{O}}$, ${^{23}\rm{Na}}$, ${^{23}\rm{Ne}}$, ${^{24}\rm{Mg}}$, ${^{25}\rm{Mg}}$, ${^{28}\rm{Si}}$

Reactions:
- **[Oxygen burning]** ${^{16}\rm{O}} + {^{16}\rm{O}} \to \rm{products}$ (specifically, the simplified reaction ```r1616```)
- **[Ne20 EC chain]** ${^{20}\rm{Ne}} + {e^{-}} \to {^{20}\rm{F}} + \nu_{e}$, ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[Ne20 EC chain]** ${^{20}\rm{F}} + {e^{-}} \to {^{20}\rm{Ne}} + \nu_{e}$, ${^{20}\rm{O}} \to {^{20}\rm{F}} + {e^{-}} + \bar{\nu}_{e}$
- **[Na23 Urca pair]** ${^{23}\rm{Na}} + {e^{-}} \to {^{23}\rm{Ne}} + \nu_{e}$, ${^{23}\rm{Ne}} \to {^{23}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$

{{< details title="Implementation of ``ONeNa.net`` (Not a task)" closed="true" >}}
Your net should have the following: 
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20
    ne20
    f20
    o20
    ! for Na23 - Ne23
    na23
    ne23
    ! for other accreted species
    mg24
    mg25
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
    ! for Na23 - Ne23 pair
    r_na23_wk_ne23
    r_ne23_wk-minus_na23
)
```

{{< /details >}}

{{< /tab >}}

<!-- ONeMgNa.net -->
{{< tab name="ONeMgNa.net" >}}

Species: ${^{1}\rm{H}}$, ${^{4}\rm{He}}$, ${^{16}\rm{O}}$, ${^{20}\rm{Ne}}$, ${^{20}\rm{F}}$, ${^{20}\rm{O}}$, ${^{23}\rm{Na}}$, ${^{23}\rm{Ne}}$, ${^{24}\rm{Mg}}$, ${^{24}\rm{Na}}$, ${^{24}\rm{Ne}}$, ${^{25}\rm{Mg}}$, ${^{28}\rm{Si}}$

Reactions:
- **[Oxygen burning]** ${^{16}\rm{O}} + {^{16}\rm{O}} \to \rm{products}$ (specifically, the simplified reaction ```r1616```)
- **[Ne20 EC chain]** ${^{20}\rm{Ne}} + {e^{-}} \to {^{20}\rm{F}} + \nu_{e}$, ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[Ne20 EC chain]** ${^{20}\rm{F}} + {e^{-}} \to {^{20}\rm{Ne}} + \nu_{e}$, ${^{20}\rm{O}} \to {^{20}\rm{F}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Mg}} + {e^{-}} \to {^{24}\rm{Na}} + \nu_{e}$, ${^{24}\rm{Na}} \to {^{24}\rm{Mg}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Na}} + {e^{-}} \to {^{24}\rm{Ne}} + \nu_{e}$, ${^{24}\rm{Ne}} \to {^{24}\rm{Na}} + {e^{-}} + \bar{\nu}_{e}$
- **[Na23 Urca]** ${^{23}\rm{Na}} + {e^{-}} \to {^{23}\rm{Ne}} + \nu_{e}$, ${^{23}\rm{Ne}} \to {^{23}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$

{{< details title="Implementation of ``ONeMgNa.net`` (Not a task)" closed="true" >}}
Your net should have the following: 
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20
    ne20
    f20
    o20
    ! for Mg24 - Na24 - Ne24
    mg24
    na24
    ne24
    ! for Na23 - Ne23
    na23
    ne23
    ! for other accreted species
    mg25
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
    ! for Mg24 - Na24 - Ne24
    r_mg24_wk_na24
    r_na24_wk-minus_mg24
    r_na24_wk_ne24
    r_ne24_wk-minus_na24
    ! for Na23 - Ne23 pair
    r_na23_wk_ne23
    r_ne23_wk-minus_na23
)
```

{{< /details >}}

{{< /tab >}}

<!-- ONeMg2Na.net -->
{{< tab name="ONeMg2Na.net" >}}

Species: ${^{1}\rm{H}}$, ${^{4}\rm{He}}$, ${^{16}\rm{O}}$, ${^{20}\rm{Ne}}$, ${^{20}\rm{F}}$, ${^{20}\rm{O}}$, ${^{23}\rm{Na}}$, ${^{23}\rm{Ne}}$, ${^{24}\rm{Mg}}$, ${^{24}\rm{Na}}$, ${^{24}\rm{Ne}}$, ${^{25}\rm{Mg}}$, ${^{25}\rm{Na}}$, ${^{25}\rm{Ne}}$, ${^{28}\rm{Si}}$

Reactions:
- **[Oxygen burning]** ${^{16}\rm{O}} + {^{16}\rm{O}} \to \rm{products}$ (specifically, the simplified reaction ```r1616```)
- **[Ne20 EC chain]** ${^{20}\rm{Ne}} + {e^{-}} \to {^{20}\rm{F}} + \nu_{e}$, ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[Ne20 EC chain]** ${^{20}\rm{F}} + {e^{-}} \to {^{20}\rm{Ne}} + \nu_{e}$, ${^{20}\rm{O}} \to {^{20}\rm{F}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Mg}} + {e^{-}} \to {^{24}\rm{Na}} + \nu_{e}$, ${^{24}\rm{Na}} \to {^{24}\rm{Mg}} + {e^{-}} + \bar{\nu}_{e}$
- **[Mg24 EC chain]** ${^{24}\rm{Na}} + {e^{-}} \to {^{24}\rm{Ne}} + \nu_{e}$, ${^{24}\rm{Ne}} \to {^{24}\rm{Na}} + {e^{-}} + \bar{\nu}_{e}$
- **[Na23 Urca]** ${^{23}\rm{Na}} + {e^{-}} \to {^{23}\rm{Ne}} + \nu_{e}$, ${^{23}\rm{Ne}} \to {^{23}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$
- **[$A=25$ Urca]** ${^{25}\rm{Mg}} + {e^{-}} \to {^{25}\rm{Na}} + \nu_{e}$, ${^{25}\rm{Na}} \to {^{25}\rm{Mg}} + {e^{-}} + \bar{\nu}_{e}$
- **[$A=25$ Urca]** ${^{25}\rm{Na}} + {e^{-}} \to {^{25}\rm{Ne}} + \nu_{e}$, ${^{25}\rm{Ne}} \to {^{25}\rm{Ne}} + {e^{-}} + \bar{\nu}_{e}$

{{< details title="Implementation of ``ONeMg2Na.net`` (Not a task)" closed="true" >}}
Your net should have the following: 
```fortran
add_isos(
    h1
    he4
    o16
    ! for Ne20 - F20 - O20
    ne20
    f20
    o20
    ! for Mg24 - Na24 - Ne24
    mg24
    na24
    ne24
    ! for Na23 - Ne23
    na23
    ne23
    ! for Mg25 - Na25 - Ne25
    mg25
    na25
    ne25
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
    ! for Mg24 - Na24 - Ne24
    r_mg24_wk_na24
    r_na24_wk-minus_mg24
    r_na24_wk_ne24
    r_ne24_wk-minus_na24
    ! for Na23 - Ne23 pair
    r_na23_wk_ne23
    r_ne23_wk-minus_na23
    ! for Mg25 - Na25 - Ne25
    r_mg25_wk_na25
    r_na25_wk-minus_mg25
    r_na25_wk_ne25
    r_ne25_wk-minus_na25
)
```

{{< /details >}}

{{< /tab >}}


{{< /tabs >}}

### Step 4: Set reaction rate source

So far we have been using the Suzuki et al. rates, but with new experimental and theoretical data, some of these rates could change. In this crowdsourcing exercise, some of you will be implementing custom rates provided by us, or ask MESA to calculate weak reaction rates on the fly. 

Check the Google spreadsheet [here](https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0) to remind yourself which rates you picked. 

> [!NOTE]
> Not everyone will get to implement custom rates / MESA on-the-fly weak rates, but there will be plenty of time at the end of this lab. Come back here for bonus points! 

-----

{{< tabs items="Suzuki Rates,Custom Weak Rates,Special (on-the-fly) Weak Rates" >}}

<!-- Suzuki rates -->
{{< tab name="Suzuki Rates" >}}

#### Suzuki Rates

#### Step 4: Using Suzuki Rates

| 📋 TASK 5 |
|:--------|
| **Edit your ``inlist_rates``** to ask MESA to use Suzuki weak rates. |

{{< details title="Hint: which inlist option?" closed="true" >}}
You can easily search for this: 
```fortran
grep -r suzuki $MESA_DIR/star/defaults
```
{{< /details >}}

{{< details title="Partial solutions" closed="true" >}}
You need this one line in your ``star_job`` section of your ``inlist_rates``:
```fortran
use_suzuki_weak_rates = .true.
```
{{< /details >}}

> [!NOTE]
> The Suzuki tables only cover $A=17-28$. 

{{< /tab >}}

<!-- Custom weak rates -->
{{< tab name="Custom Weak Rates" default="true" >}}

#### Custom Weak Rates

You can supply your own tabulated weak rates to MESA. Here we will show you how to use this feature. 

> [!NOTE]
> You can also do this for *regular* reactions, but here we'll show you how to use custom *weak* reaction rates. 

#### Step 4a: Tell MESA to use a custom rate table

We first need to tell MESA the location of the directory (which we'll call `tables_custom`) to find the tabulated custom rates. This is an inlist option. 

| 📋 TASK 4a |
|:--------|
| Edit `inlist_rates` to have it use your custom rate table. |

{{< details title="Hint: how to find this inlist option?" closed="true" >}}
Look up ``rate_table`` in ``$MESA_DIR/star/defaults/``:
```bash
grep -r rate_table $MESA_DIR/star/defaults/
```
{{< /details >}}

{{< details title="Partial Solution" closed="true" >}}
Add the following to the ``star_job`` section of your inlist:
```fortran
rate_tables_dir = 'tables_custom'
```

You will also need to ask MESA to **not** use Suzuki weak rates, in the ``star_job`` section:
```fortran
use_suzuki_weak_rates = .false.
```

{{< /details >}}

#### Step 4b: Download data

| 📋 TASK 4b |
|:--------|
| **Download** the weak rate tables [here](https://drive.google.com/file/d/1qtQLwOf2qovA8pI5miiD6kxgNBJVe28x/view?usp=drive_link) to your working directory and **unzip** it. |

After that, your working directory should look like:

{{< filetree/container >}}
  {{< filetree/folder name="work directory" >}} .
    {{< filetree/file name="other things" >}} .
    {{< filetree/folder name="tables_custom" >}} .
        {{< filetree/file name="weak_rate_list.txt">}}  .
        {{< filetree/file name="on-the-fly_r_f20_wk_o20.h5">}} .
        {{< filetree/file name="other h5 files" >}} .
    {{< /filetree/folder >}}
  {{< /filetree/folder >}}
{{< /filetree/container >}}

Each ``h5`` file contains the rates for each weak reaction, for example, ``on-the-fly_r_f20_wk_o20.h5`` for the electron capture reaction ${^{20}\rm{F} + e^{-} \to {^{20}\rm{O}}}$. 

#### Step 4c: Edit weak_rates.list

Once we point MESA to `rates_dir`, it will look for `rate_list.txt` (for regular reactions, which we won't modify) and `weak_rate_list.txt` (for weak reactions), *if* they exist. 
These two lists tell MESA the reaction names and the corresponding file names. 

| 📋 TASK 4c |
|:--------|
| **Add** the following four reactions to  **`weak_rate_list.txt`**. Take a look at `weak_rate_list.txt` to see what is needed. |
- ${^{20}\rm{Ne}} + e^{-} \to {^{20}\rm{F}} + \nu_{e}$
- ${^{20}\rm{F}} \to {^{20}\rm{Ne}} + e^{-} + \bar{\nu}_{e}$
- ${^{20}\rm{F}} + e^{-} \to {^{20}\rm{O}} + \nu_{e}$
- ${^{20}\rm{O}} \to {^{20}\rm{F}} + e^{-} + \bar{\nu}_{e}$

> [!WARNING]
> We have already included the other weak reactions for you. Do *not* remove any of the other reactions. 

{{< details title="Hint" closed="true" >}}
The format is as follows:
```fortran
<reaction name> <h5 file name>
```
{{< details title="Hint: what is the reaction name format?" closed="true" >}}
For electron capture reactions ($X + e^{-} \to Y + \nu_{e}$), the format is `r_x_wk_y`. 
For beta decay reactions ($Y \to X + e^{-} + \bar{\nu}_{e}$), the format is `r_x_wk-minus_y`. 
{{< /details >}}
{{< /details >}}

{{< details title="Partial solution" closed="true" >}}
You need to add the following to `weak_rate_list.txt`: 
```fortran
r_ne20_wk_f20 'on-the-fly_r_ne20_wk_f20.h5'
r_f20_wk-minus_ne20 'on-the-fly_r_f20_wk-minus_o20.h5'
r_f20_wk_o20 'on-the-fly_r_f20_wk_o20.h5'
r_o20_wk-minus_f20 'on-the-fly_r_o20_wk-minus_f20.h5'
```
{{< /details >}}

{{< /tab >}}

<!-- Special rates -->
{{< tab name="Special (on-the-fly) rates" >}}

#### Special (On-the-fly) Weak Rates

MESA has the capability to calculate the weak reactions on-the-fly, if you supply the list of transitions and energy levels. 

#### Step 4a: Telling MESA to use special (on-the-fly) weak rates

This is an inlist option. 

<!-- Edit inlist -->
| 📋 TASK 4a |
|:--------|
| Edit `inlist_rates` to have MESA use special weak rates. |

{{< details title="Hint: What inlist option to look for?" closed="true" >}}

The on-the-fly capability is called `special_weak_rates` in MESA. You can search this as follows:

```bash
grep -r special_weak $MESA_DIR/star/defaults/
```

{{< /details >}}

{{< details title="Partial solution" closed="true" >}}

In `&star_job` of your `inlist_rates`, set
```fortran
use_special_weak_rates = .true.
```

You will also need to ask MESA to **not** use Suzuki weak rates, in the ``star_job`` section:
```fortran
use_suzuki_weak_rates = .false.
```

{{< /details >}}


#### Step 4b: Feeding MESA the states and transitions

For MESA to calculate the weak rates, it needs to know the nuclear states of the isotopes (energies and spins), and the halftimes of the transitions between these states. 

| 📋 TASK 4b |
|:--------|
| **Download** the states file and the transition file [here](https://drive.google.com/file/d/1JWbVpgbDwPfDwaaJ_LnmkfExNxZ4BAUY/view?usp=drive_link) and [here](https://drive.google.com/file/d/10wsOlGsfWX_vjepwX9Fk9gjvkix-o6ml/view?usp=drive_link) to your working directory. |

| 📋 TASK 4c |
|:--------|
| **Edit `inlist_rates`** to supply MESA with the states file and the transitions file. |

{{< details title="Hint: What inlist option to look for?" closed="true" >}}

You have probably found them if you followed the hints from task 5a. You can search this as follows:

```bash
grep -r special_weak $MESA_DIR/star/defaults/
```

{{< /details >}}

{{< details title="Partial solution" closed="true" >}}

In `&star_job`, set
```fortran
special_weak_states_file = 'weak.states'
special_weak_transitions_file = 'weak.transitions'
```

{{< /details >}}

{{< /tab >}}

{{< /tabs >}}

### Step 5: Change Log Directory

| 📋 TASK 5 |
|:--------|
| Finally, change the output directory to something you name. |

A suggested format is something like ``LOGS_<accretion rate>_<net name>_<weak rate name>``. 

{{< details title="Hint: what inlist option to use?" closed="true" >}}

This is called ``log_directory``. You can easily search this:
```bash
grep -r log_directory $MESA_DIR/star/defaults/
```

{{< /details >}}

{{< details title="Partial solution" closed="true" >}}

In `&controls` of your `inlist_rates`, set something like, 
```fortran
log_directory = "LOGS_1d-6_ONe_custom"
```

{{< /details >}}

Now you're ready to go!

### Step 6: Declaring Bankruptcy

| 📋 TASK 6 |
|:--------|
| The only thing stopping your white dwarf from getting bankrupt is just you hitting ``./rn``. **Record the central density of your model in the Google spreadsheet [here](https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0)** at the end of the run. |

> [!TIP]
> You can do the following sanity check to see if you're using the correct net: 
> In ``star_job`` in ``inlist_net``, set ``show_net_species_info = .true.`` and ``show_net_reactions_info = .true.``. 
> Then do ``./rn`` and let MESA run for a few steps. MESA will first print out the species and reactions in the net. 

> [!TIP]
> If you're using **custom rates**, when MESA first runs, you should see messages like ``reading user weak rate file tables_custom/on-the-fly_r_mg24_wk_na24.h5``. 

> [!WARNING]
> If you haven't yet, do ``./clean && ./mk`` first.


## Review reaction flow with pynucastro

We can easily visualize the reaction flow with the ``pynucastro`` python package and build up some intuition. 
Go to [this](https://drive.google.com/file/d/1I3NQMQpB3Vsf-c8Yd40nggisLtOOTlzF/view?usp=sharing) Google colab notebook and go through the exercises. 

## Bonus exercises 

We have done many things in this lab to ensure short runtimes. Here are a few suggested exercises you can try towards building a better model. 

Do **not** attempt these all at once! Your run will be unbearably slow. 


{{< tabs items="Bigger Net,Soft-wired Net,Time Resolution,Spatial Resolution,Skye EOS,Name Your Bison" >}}

<!-- bigger nets -->
{{< tab name="bigger net" >}}

### Bigger reaction network

#### Oxygen burning

In this lab, we asked you to use ``r1616`` for oxygen burning. What exactly does it do? 

| 📋 TASK |
|:--------|
| Look up ``r1616`` on ``$MESA_DIR/data/rates_data/reactions.list``.  |

{{< details title="Hint: How to search?" closed="true" >}}

```bash
grep -r r1616 $MESA_DIR/data/rates_data/reactions.list
```

{{< /details >}}

{{< details title="Partial solution" closed="true" >}}

You should see something like
```bash
r1616                               2 o16                             =>  1 he4     + 1 si28
```
which is exactly ${^{16}\rm{O}} + {^{16}\rm{O}} \to {^{4}\rm{He}} + {^{28}\rm{Si}}$. 
But if you open ``$MESA_DIR/data/rates_data/reactions.list`` and go to line 124, under ``info``, you'll see
```bash
o16+o16 => a + si28, a and p
```

{{< /details >}}

The ${^{16}\rm{O}} + {^{16}\rm{O}}$ reaction doesn't always give an alpha particle ($^{4}\rm{He}$) and ${^{28}\rm{Si}}$ as products. It sometimes returns a proton as a product (${^{16}\rm{O}} + {^{16}\rm{O}} \to {p} + {^{31}\rm{P}}$). To keep our nuclear net small, we purposely left out $^{31}\rm{P}$. The `r1616` rate combines both the `a` and `p` channels, but uses ${^{28}\rm{Si}}$ as the end point. This is also what built-in nets like ``co_burn.net`` do. 

An obvious improvement would be to include ${^{31}\rm{P}}$ in our net and add the reactions that connect it to ${^{16}\rm{O}} $. This will be covered in the next part. 

#### Other important reactions

What other important reactions have we missed? Here we will use ``pynucastro`` to find out. 

| 📋 TASK |
|:--------|
| **Go to [placeholder]() Google collab**. **Use ``pynucastro``** to find out what isotopes and reactions are missing. **Edit your net** accordingly. |


| 📋 TASK |
|:--------|
| Finally, ``./mk`` , ``./clean`` and ``./rn``. Observe if the reaction flow behaves differently. |

{{< /tab >}}

<!-- Soft-wired net -->
{{< tab name="Soft-wired Net" >}}

### Soft-wired Net

In this lab, we showed you how to hard-wire a list of isotopes and reactions into the net. You can also supply a list of isotopes to MESA and ask it to connect them with every possible reaction. Here we will do that. 

| 📋 TASK |
|:--------|
| Look up how built-in nets like ``mesa_49.net`` soft-wire the network. |

{{< details title="Hint: where do the built-in nets live?" closed="true" >}}

They live in ``$MESA_DIR/data/net_data/nets``. 

{{< /details >}}


{{< details title="Partial solutions" closed="true" >}}

The soft-wiring is done by 
```fortran
add_isos_and_reactions(
    <isotope name>
    <chemical name> <lower limit of mass number> <upper limit of mass number>
)
```
For example, 
```fortran
add_isos_and_reactions(
    h1 
    he 3 4
)
```
will include $^{1}\rm{H}$, $^{3}\rm{He}$, $^{4}\rm{He}$. 

{{< /details >}}


| 📋 TASK |
|:--------|
| Take the same isotopes as ``ONeMg2Na.net`` and soft-wire the network. |

{{< details title="Partial solutions" closed="true" >}}

```fortran
add_isos_and_reactions(
    h1
    he4
    o16
    o20
    f20
    ne20
    ne 23 25
    na 23 25
    mg 23 25
    si28
)
```

{{< /details >}}

| 📋 TASK |
|:--------|
| **Edit your inlist** to have MESA print out the list of isotopes and reactions. |

{{< details title="Hint: where to find this option?" closed="true" >}}

Look up ``show_net`` on ``$MESA_DIR/star/defaults/``:
```bash
grep -r show_net $MESA_DIR/star/defaults/
```

{{< /details >}}


{{< details title="Partial solutions" closed="true" >}}

In ``star_job`` in ``inlist_net``, set ``show_net_species_info = .true.`` and ``show_net_reactions_info = .true.``. 

{{< /details >}}

Now you're ready to run. 

| 📋 TASK |
|:--------|
| Finally, ``./mk`` , ``./clean`` and ``./rn``. Look at the terminal to find the list of species and reactions in your new net. Observe if the reaction flow behaves differently. |

{{< /tab >}}

<!-- time resolution -->
{{< tab name="Time Resolution" >}}

In this lab, we relaxed the time resolution when it comes to the white dwarf's surface luminosity and temperature:
```fortran
delta_lgL_limit = 0.2d0
delta_lgTeff_limit = 0.05d0
```

These two items mainly affect the time stepping that matters more for the accretion (because compressional heating changes the surface luminosity). But otherwise, we didn't do anything to relax the time resolution. 

#### Limiting changes in $T_{c}$ and $\rho_{c}$

Now, when the core undergoes weak reactions and oxygen ignition, we've seen that it undergoes rapid changes in $T_{c}$ and $\rho_{c}$. 

We will work through some useful controls to limit these changes. 

| 📋 TASK |
|:--------|
| Check out which inlist options are related to timestepping limited by changes in $T_{c}$ and $\rho_{c}$. **Add these controls** into your inlist. | -->

{{< details title="Hint: which inlist section?" closed="true" >}}

Time stepping controls are in the ``&controls`` section. Check out ``$MESA_DIR/star/defaults/controls.defaults``. 

{{< /details >}}

{{< details title="Partial solutions" closed="true" >}}

These are actually commented out in ``inlist_commons``. 

The options like
```fortran
delta_lgT_cntr_limit = 2d-2  ! default is 0.01d0
delta_lgRho_cntr_limit = 5d-3  ! default is 0.05d0
```

{{< /details >}}

| 📋 TASK |
|:--------|
| **Add these controls** into your inlist and run MESA again. What values to use? A good starting point is in the ``partial solutions`` above. |

> [!WARNING]
> Make sure you do ``./clean`` and ``./mk`` first. 


#### Limiting changes in nuclear burning luminosity

When oxygen burning starts, the temperature profile changes very rapidly. A good way to limit these changes would be to use 
```fortran
delta_lgT_max_limit = 0.01d0 ! default is -1, meaning this is turned off
delta_lgT_max_limit_lgT_min = 8.8d0 
```
This combination will limit the change in $\log_{10}T_{\rm max}$ by less than $0.01$ once $\log_{10}T_{\rm max} > 8.8$, and has no effect for lower temperatures. 

Alternatively, you can try to limit changes in the nuclear burning luminosity $L_{\rm nuc}$. Normally, you can use ``delta_lgL_nuc_limit`` to limit changes in $\log_{10}(L_{\rm nuc}/L_{\odot})$. The problem is, here the weak reactions produce a lot of cooling and cancel out the overall nuclear burning luminosity globally (but not locally). So, here we will show you how to do this with ``run_star_extras``, particularly with ``other_timestep_limit``. 

| 📋 TASK |
|:--------|
| Add ``use_other_timestep_limit = .true. `` in ``inlist_common``. |

Next, we will edit ``run_star_extras``. 

| 📋 TASK |
|:--------|
| Copy the subroutine in ``$MESA_DIR/star/other/other_timestep_limit.f90`` to your ``run_star_extras``. Rename it ``L1616_timestep_limit``. |

{{< details title="Partial solutions" closed="true" >}}

Your subroutine should look like this
```fortran
integer function L1616_timestep_limit( &
    id, skip_hard_limit, dt, dt_limit_ratio)
    use const_def, only: dp
    use star_def
    integer, intent(in) :: id
    logical, intent(in) :: skip_hard_limit
    real(dp), intent(in) :: dt
    real(dp), intent(inout) :: dt_limit_ratio
    L1616_timestep_limit = keep_going
end function L1616_timestep_limit
```

> [!WARNING]
> Because this is a Fortran ``function``, make sure you also change ``other_timestep_limit = keep_going`` to ``L1616_timestep_limit = keep_going`` in the second last line of this subroutine!

{{< /details >}}

> [!TIP]
> To check if you did this right, do ``./clean`` and ``./mk``. 

| 📋 TASK |
|:--------|
| Next, make sure that your star pointer points to this subroutine in ``run_star_extras``. |

{{< details title="Hint: where to point?" closed="true" >}}

This should be done in ``extra_controls``. 

{{< /details >}}

{{< details title="Partial solutions" closed="true" >}}

In your ``extra_controls``, add ``s% other_timestep_limit => L1616_timestep_limit``. Your ``extra_controls`` should look like
```fortran
    subroutine extras_controls(id, ierr)
         integer, intent(in) :: id
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
         ! this is the place to set any procedure pointers you want to change
         ! e.g., other_wind, other_mixing, other_energy  (see star_data.inc)


         ! the extras functions in this file will not be called
         ! unless you set their function pointers as done below.
         ! otherwise we use a null_ version which does nothing (except warn).

         s% extras_startup => extras_startup
         s% extras_start_step => extras_start_step
         s% extras_check_model => extras_check_model
         s% extras_finish_step => extras_finish_step
         s% extras_after_evolve => extras_after_evolve
         s% how_many_extra_history_columns => how_many_extra_history_columns
         s% data_for_extra_history_columns => data_for_extra_history_columns
         s% how_many_extra_profile_columns => how_many_extra_profile_columns
         s% data_for_extra_profile_columns => data_for_extra_profile_columns  

         s% how_many_extra_history_header_items => how_many_extra_history_header_items
         s% data_for_extra_history_header_items => data_for_extra_history_header_items
         s% how_many_extra_profile_header_items => how_many_extra_profile_header_items
         s% data_for_extra_profile_header_items => data_for_extra_profile_header_items

         !!! new
         s% other_timestep_limit => L1616_timestep_limit

      end subroutine extras_controls
```
{{< /details >}}

Now, we are ready to edit ``L1616_timestep_limit``. Our goal is to have MESA check the nuclear burning luminosity of the ${^{16}\rm{O}}+{^{16}\rm{O}}$ reaction, $L_{1616}$, and implement timestep limits based on how much $\log_{10} L_{1616}$ changes. 


<!-- Task :  -->
| 📋 TASK |
|:--------|
| Find the ``star_data`` variable that is related to nuclear burning luminosity of specific reaction categories. |

{{< details title="Partial solutions" closed="true" >}}

This is given by 
```fortran
! integrated eps_nuc_categories (ergs/sec)
real(dp), pointer :: luminosity_by_category(:,:) ! (num_categories, nz)
```

As you can see, the first index is for different categories of reactions, and the second index is for different zones in the stellar model. 

{{< /details >}}

<!-- Task :  -->
| 📋 TASK |
|:--------|
| Now that we have found the ``star_data`` variable for $L_{1616}$, edit ``L1616_timestep_limit`` to have it calculate $\log_{10} (L_{1616}/L_{\odot})$ at end of the current timestep, call it ``log_L1616``. |

<!-- Hint 1  -->
{{< details title="Hint: what is the index for ${^{16}\rm{O}}+{^{16}\rm{O}}$ in the burning categories?" closed="true" >}}

This is given by ``ioo``. However, ``ioo`` is defined elsewhere (in ``$MESA_DIR/chem/public/chem_def.f90``), and ``run_star_extras`` does not know about this definition beforehand. You need to declare
```fortran
use chem_def
```
at the top of the function. 

{{< /details >}}

<!-- Hint 2  -->
{{< details title="Hint: which zone do we want?" closed="true" >}}

Since we want the global change, we want the luminosity at the surface. The surface zone is zone `1`. 

{{< /details >}}

<!-- Hint 3  -->
{{< details title="Hint: how will the ``L1616_timestep_limit`` function know about the ``star_data``?" closed="true" >}}

First, you need to call ``star_ptr``, so that the ``L1616_timestep_limit`` function knows about the ``s`` pointer. This requires declaring
```fortran
type (star_info), pointer :: s
integer :: ierr
```
at the top of the function, and then adding 
```fortran
ierr = 0
call star_ptr(id, s, ierr)
if (ierr /= 0) return
```

{{< /details >}}

<!-- Solutions -->

{{< details title="Partial solutions" closed="true" >}}

Your function should look like
```fortran
integer function L1616_timestep_limit( &
    id, skip_hard_limit, dt, dt_limit_ratio)
    use const_def, only: dp
    use star_def
    use chem_def, only : ioo ! new
    integer, intent(in) :: id
    logical, intent(in) :: skip_hard_limit
    real(dp), intent(in) :: dt
    real(dp), intent(inout) :: dt_limit_ratio
    real(dp) :: log_L1616 ! new
    type (star_info), pointer :: s  ! new
    integer :: ierr
    ierr = 0
    call star_ptr(id, s, ierr) ! new
    if (ierr /= 0) return

    L1616_timestep_limit = keep_going

    ! new
    log_L1616 = safe_log10(s% luminosity_by_category(ioo,1)/lsun)

end function L1616_timestep_limit
```

{{< /details >}}

<!-- Task :  -->
| 📋 TASK |
|:--------|
| Edit ``L1616_timestep_limit`` to check whether $\log_{10}(L_{1616}/L_{\odot})$ is greater than a minimum value of $-1.0$. If not, have it return; we do not want to limit the timestep if the oxygen burning luminosity is tiny. Implement this minimum using the inlist ``x_ctrl`` options. |

<!-- Partial solution  -->
{{< details title="Partial solution" closed="true" >}}

Your function should look like
```fortran
integer function L1616_timestep_limit( &
    id, skip_hard_limit, dt, dt_limit_ratio)
    use const_def, only: dp
    use star_def
    use chem_def, only : ioo
    integer, intent(in) :: id
    logical, intent(in) :: skip_hard_limit
    real(dp), intent(in) :: dt
    real(dp), intent(inout) :: dt_limit_ratio
    real(dp) :: log_L1616
    type (star_info), pointer :: s
    integer :: ierr
    ierr = 0
    call star_ptr(id, s, ierr)
    if (ierr /= 0) return

    L1616_timestep_limit = keep_going

    log_L1616 = safe_log10(s% luminosity_by_category(ioo,1)/lsun)

    ! new
    if (log_L1616 <= s% x_ctrl(1)) return

end function L1616_timestep_limit
```

> [!WARNING]
> You should add ``x_ctrl(1) = -1d0`` somewhere in the ``&controls`` section of your inlist. 

{{< /details >}}

Now we are ready to calculate the change in $\log_{10}(L_{1616}/L_{\odot})$ at the start and end of the timestep. 

<!-- Task :  -->
| 📋 TASK |
|:--------|
| Edit ``L1616_timestep_limit`` to have it calculate $\log_{10} L_{1616}/L_{\odot}$ at **start** of the current timestep, call it ``log_L1616_start``. Then have the function calculate the change in $\log_{10} L_{1616}/L_{\odot}$ at this timestep. |

<!-- Hint  -->
{{< details title="Hint: is there a similar ``star_data`` variable for $L_{1616}$ at the start of timestep?" closed="true" >}}

Yes, this is called ``luminosity_by_category_start``. 

{{< /details >}}

<!-- Partial solution  -->
{{< details title="Partial solution" closed="true" >}}

Your function should look like
```fortran
integer function L1616_timestep_limit( &
    id, skip_hard_limit, dt, dt_limit_ratio)
    use const_def, only: dp
    use star_def
    use chem_def, only : ioo
    integer, intent(in) :: id
    logical, intent(in) :: skip_hard_limit
    real(dp), intent(in) :: dt
    real(dp), intent(inout) :: dt_limit_ratio
    real(dp) :: log_L1616, log_L1616_start, dlog_L1616 ! new
    type (star_info), pointer :: s
    integer :: ierr
    ierr = 0
    call star_ptr(id, s, ierr)
    if (ierr /= 0) return

    L1616_timestep_limit = keep_going

    log_L1616 = safe_log10(s% luminosity_by_category(ioo,1)/lsun)
    if (log_L1616 <= s% x_ctrl(1)) return

    ! new
    log_L1616_start = safe_log10(s% luminosity_by_category_start(ioo,1)/lsun)
    dlog_L1616 = abs(log_L1616_start - log_L1616)

end function L1616_timestep_limit
```

{{< /details >}}


<!-- Task -->
| 📋 TASK |
|:--------|
| Now take a look at ``$MESA_DIR/star/private/timestep.f90`` and try to understand how it limits the timestep based on changes in quantities. Go to line 1007 and look at the ``check_lgL`` integer function, particularly lines 1110 to 1131. Implement these in your ``L1616_timestep_limit`` function. We will limit the change in $\log_{10}(L_{1616}/L_{\odot})$ to some value ``lim = 0.02d0``, which you should implement with the inlist ``x_ctrl`` options. |

<!-- Partial solution  -->
{{< details title="Partial solution" closed="true" >}}

We will define a variable ``lim`` that stores the value of ``x_ctrl(2)``, which is the change in $\log_{10}(L_{1616}/L_{\odot})$ allowed between timesteps. 

Next, we will define a variable ``relative_excess = (dlog_L1616 - lim) / lim``, which is relatively how much the change in $\log_{10}(L_{1616}/L_{\odot})$ is, in excess of our target value ``lim``. 

Finally, we modify the value of ``dt_limit_ratio``, which is passed into and out of the function. This tells MESA how much we want to limit the timestep ``dt``. We calculate this following the functions in ``$MESA_DIR/star/private/timestep.f90``. 

Your function should look like
```fortran
integer function L1616_timestep_limit( &
    id, skip_hard_limit, dt, dt_limit_ratio)
    use const_def, only: dp
    use star_def
    use chem_def, only : ioo
    integer, intent(in) :: id
    logical, intent(in) :: skip_hard_limit
    real(dp), intent(in) :: dt
    real(dp), intent(inout) :: dt_limit_ratio
    real(dp) :: log_L1616, log_L1616_start, dlog_L1616
    real(dp) :: lim, relative_excess ! new
    type (star_info), pointer :: s
    integer :: ierr
    ierr = 0
    call star_ptr(id, s, ierr)
    if (ierr /= 0) return

    L1616_timestep_limit = keep_going

    log_L1616 = safe_log10(s% luminosity_by_category(ioo,1)/lsun)
    if (log_L1616 <= s% x_ctrl(1)) return
    log_L1616_start = safe_log10(s% luminosity_by_category_start(ioo,1)/lsun)
    dlog_L1616 = abs(log_L1616_start - log_L1616)

    ! new
    lim = s% x_ctrl(2)
    relative_excess = (dlog_L1616 - lim) / lim
    dt_limit_ratio = 1d0/pow(s% timestep_dt_factor,relative_excess)
    if (dt_limit_ratio <= 1d0) dt_limit_ratio = 0

end function L1616_timestep_limit
```
> [!WARNING]
> You should add ``x_ctrl(2) = 0.05d0`` somewhere in the ``&controls`` section of your inlist. 

{{< /details >}}


Now we are ready to run and test whether our ``L1616_timestep_limit`` function works. 
<!-- Task -->
| 📋 TASK |
|:--------|
| Compile and run MESA. Check the terminal to see if the timestep is limited by ``other_timestep`` towards the end. |

> [!WARNING]
> Did you set ``use_other_timestep_limit = .true.``, ``x_ctrl(1) = -1d0``, and ``x_ctrl(2) = 0.02d0`` in the ``&controls`` section of your inlist? 

> [!TIP]
> You can ask MESA to print out the value of $\log_{10}(L_{1616}/L_{\odot})$, to make sure that the changes in this quantity are small between timesteps towards the end. 

{{< /tab >}}


<!-- spatial resolution -->
{{< tab name="Spatial Resolution" >}}

### Spatial Resolution

#### Resolution around Urca shells

One thing we did do well is putting more spatial resolution around the Urca shells: 
| 📋 TASK |
|:--------|
| Take a look at ``inlist_common``. Check out which inlist options are related to resolution around the Urca shells. |

{{< details title="Partial solutions" closed="true" >}}

Here we opted to put more resolution where the Urca species are changing abundances. 

The options like
```fortran
xa_function_species(1) = 'na25'
xa_function_weight(1) = 15
xa_function_param(1) = 1d-4
```
track the mass fraction of particular species and put more resolution where more change is happening. In this example, we put more resolution where $^{25}\rm{Na}$ changes mass fraction and reaches $10^{-4}$. 

{{< /details >}}

> [!TIP]
> You would think that higher spatial resolution would slow your run down. While mostly true, this also **helps with convergence**. When encountering convergence problems, one generally useful technique is to ensure that you have enough spatial resolution. 

| 📋 TASK |
|:--------|
| **Comment out** the `xa_function*` options in ``inlist_common``, and **run MESA again**. Observe if the number of retries are higher. Note also the shape of the $T-\rho$ profile around the Urca shells. |

> [!WARNING]
> Be sure to do `./clean` and `./mk` first. 

Hope this exercise helps you appreciate the utility of higher spatial resolution.

#### ``mesh_delta_coeff``

Of course, we lowered the overall spatial resolution by setting a large ``mesh_delta_coeff = 2.5``. 

> [!TIP]
> For converged runs, try ``mesh_delta_coeff`` less than or equal to 1. 

| 📋 TASK |
|:--------|
| **Set ``mesh_delta_coeff = 1.0``** in ``inlist_common``, and **run MESA again**. Check if the evolution is any different. |

> [!WARNING]
> **Uncomment** the `xa_function*` options in ``inlist_common`` first. We want the resolution around Urca shells!

{{< /tab >}}

<!-- Skye EOS -->
{{< tab name="Skye EOS" >}}

#### Skye EOS

In this lab, we have turned off the Skye EOS, in favor of the HELM EOS. They both cover the degenerate region, but Skye EOS has better treatment of Coulomb effects in these dense regions. Sadly, better physics (thermodynamics) sometimes means more convergence issues. So to speed things up, we turned off Skye EOS. 

> [!IMPORTANT]
> For low accretion rates (like $10^{-8}M_{\odot}\rm{yr}^{-1}$), Urca cooling will cool the core sufficiently that it reaches crystallization. The thermodynamics of crystallization, and Coulomb effects under degenerate coniditions, are more properly treated with the Skye EOS, so it is important to consider using Skye EOS. 

| 📋 TASK |
|:--------|
| **Set ``use_Skye = .true.``** in ``inlist_common``, and **run MESA again**. Check if the evolution is any different. |

> [!WARNING]
> Make sure you do ``./clean`` and ``./mk`` first. 

> [!IMPORTANT]
> We recommend $\dot{M} > 10^{-7} M_{\odot} \rm{yr}^{-1}$ (``mass_change = 1d-7`` or greater). Lower values will result in hours-long runs because the Coulomb effects are stronger and there are more convergence issues. 

> [!NOTE]
> We also set ``mass_fraction_limit_for_Skye = 1d-10``. By default, this is ``1d-4``. We lowered this number so that the EOS considers even trace elements on the thermodynamics. We do not recommend even lower values. 

{{< /tab >}}

<!-- Name your bison -->
{{< tab name="Name your bison" >}}

Go to the Google spreadsheet [here](https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?usp=sharing) and add names of your bison. Our lecturer Mike will reward valuable MESA summer school points for his favorite name(s). 

{{< /tab >}}

{{< /tabs >}}


