---
weight: 1
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
| **Download** the starting point from the [Google Drive]( FIXLINK ) to a local working directory. |

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

You've done great work in labs 1 and 2 to implement custom network, so here we will just supply the networks need. 

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

### Step 4: Set reaction rate source

So far we have been using the Suzuki et al. rates, but with new experimental and theoretical data, some of these rates could change. In this crowdsourcing exercise, some of you will be implementing custom rates provided by us, or ask MESA to calculate weak reaction rates on the fly. 

Check the Google spreadsheet [here](https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0) to remind yourself which rates you picked. 

> [!NOTE]
> Not everyone will get to implement custom rates / MESA on-the-fly weak rates, but there will be plenty of time at the end of this lab. Come back here for bonus points! 

-----

{{< tabs items="Suzuki Rates,Custom Weak Rates,Special (on-the-fly) Weak Rates" >}}

<!-- Suzuki rates -->
{{< tab name="Suzuki Rates" >}}

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

### Step 6: Declaring Bankrupcy

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
Go to [this](blah) Google colab notebook and go through the exercises. 

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
| **Go to [this]() Google collab**. **Use ``pynucastro``** to find out what isotopes and reactions are missing. **Edit your net** accordingly. |


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

{{< /tabs >}}


