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

The starting point is a very simple setup. 


### Step 1: Pick a model

| 📋 TASK 2 |
|:--------|
|  Go to the spreadsheet [here]( https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0 ). Pick any combination of the accretion rate, reaction network and reaction rates provided. Users with more cores should pick more computationally expensive ones. |


### Step 2: Changing the accretion rate

| 📋 TASK 3 |
|:--------|
| Edit `inlist_accrete` to set the accretion rate that you chose. |



{{< details title="What variable needs to be changed?" closed="true" >}}

{{< /details >}}



{{< details title="Partial solution" closed="true" >}}

In `&controls`, set `mass_change = <your value>`. 

{{< /details >}}

### Step 3: Set your network

You've done the hard work in labs 1 and 2 to implement custom networks. So here we will supply the networks you will need. 

| 📋 TASK 3 |
|:--------|
| **Edit `inlist_rates`** to have it use your specific network, which we supply in **`nets_lab3`**. |

> [!TIP]
> You can do the following sanity check: 
> In ``star_job`` in ``inlist_common``, set ``show_net_species_info = .true.`` and ``show_net_reactions_info = .true.``. 
> Then do ``./rn`` and let MESA run for a few steps. MESA will first print out the species and reactions in the net. 
> Once you see that, just do ``ctrl+c`` to stop. 

> [!WARNING]
> If you haven't yet, do ``./clean && ./mk`` first.



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
| **Edit your inlist** to ask MESA to use Suzuki weak rates. |

{{< details title="Hint: which inlist option?" closed="true" >}}
You can easily search for this: 
```fortran
grep -r suzuki $MESA_DIR/star/defaults
```
{{< /details >}}

{{< details title="Partial solutions" closed="true" >}}
You need this one line in your ``star_job`` section of your inlist:
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
rate_table = 'tables_custom'
```

You will also need to ask MESA to **not** use Suzuki weak rates, in the ``star_job`` section:
```fortran
use_suzuki_weak_rates = .false.
```

{{< /details >}}

#### Step 4b: Download data

| 📋 TASK 4b |
|:--------|
| **Download** the weak rate tables [here]() to your working directory and **unzip** it. |

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
{{< details title="What is the reaction name format again?" closed="true" >}}
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

In `&star_job`, set
```fortran
use_special_weak_rates = .true.
```

{{< /details >}}


#### Step 4b: Feeding MESA the states and transitions

For MESA to calculate the weak rates, it needs to know the nuclear states of the isotopes (energies and spins), and the halftimes of the transitions between these states. 

| 📋 TASK 4b |
|:--------|
| **Download** the states file and the transition file [here]() to your working directory. |

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
special_weak_states_file = 'special_weak_rates.states'
special_weak_transitions_file = 'special_weak_rates.transitions'
```

{{< /details >}}

{{< /tab >}}

{{< /tabs >}}



Now you're ready to go!

### Step 5: Declaring Bankrupcy

| 📋 TASK 5 |
|:--------|
| The only thing stopping your white dwarf from getting bankrupt is just you hitting ``./rn``. **Record the central density of your model in the Google spreadsheet [here](https://docs.google.com/spreadsheets/d/15PK9myW3oriuTeZvGFNGRKHqqphOHUFQoOcShtuME-g/edit?gid=0#gid=0)** at the end of the run. |

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
The ``o16+o16`` reaction doesn't always give an alpha particle ($^{4}\rm{He}$) as a product. It sometimes returns a proton as a product (${^{16}\rm{O}} + {^{16}\rm{O}} \to {p} + {^{31}\rm{P}}$), but the `r1616` combines both the `a` and `p` channels in the energy released. To keep our nuclear net small, we left out $^{31}\rm{P}$, 



{{< /details >}}


#### Other important reactions

So far we've told you what isotopes and reactions are important to include, but what other important reactions have we missed? Here we will use ``pynucastro`` to find out. 

| 📋 TASK |
|:--------|
| **Go to [this]() Google collab**. **Use ``pynucastro``** to find out what isotopes and reactions are missing. **Edit your net** accordingly. |

{{< /tab >}}

{{< /tabs >}}


