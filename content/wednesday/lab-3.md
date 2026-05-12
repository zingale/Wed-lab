---
weight: 1
author: Tryston Raecke, Josh Wanninger, Sunny Wong, Michael Zingale
math: true
disableKinds: "rss"
---
# Minilab 3: They all go broke

So far we have changed the nuclear net to include more reactions, and looked at the effect of Urca cooling from the $^{23}\rm{Na}$-$^{23}\rm{Ne}$ pair on the stellar structure. 

Now we will do a crowdsourcing to look at how the evolution changes with the accretion rate $\dot{M}$, reaction networks, and reaction rates. 
The goal is to look at how they change the core properties at the onset of oxygen ignition, because whether an electron-capture supernova undergoes a thermonuclear explosion or core-collapse (implosion) is extremely sensitive to the central density. 

### Step 0: Start up

| 📋 TASK 1 |
|:--------|
| **Download** the starting point from the [Google Drive]( FIXLINK ) to a local working directory. |

The starting point is a very simple setup. 


### Step 1: Pick a model

| 📋 TASK 2 |
|:--------|
|  Go to the spreadsheet [here]( FIXLINK ). Pick any combination of the accretion rate, reaction network and reaction rates provided. Users with more cores should pick more computationally expensive ones. |


### Step 2: Changing the accretion rate

| 📋 TASK 3 |
|:--------|
| Edit `inlist_accrete` to set the accretion rate that you chose. |



{{< details title="What variable needs to be changed?" closed="true" >}}

{{< /details >}}



{{< details title="Partial solution" closed="true" >}}

In `&controls`, set `mass_change = <your value>`. 

{{< /details >}}

### Step 3: Build your network

| 📋 TASK 3 |
|:--------|
| Edit `<name of your net>.net` to add the nuclear species and reactions connecting them. **Click on the tabs below** to review the instructions for your specific net. |

{{< details title="ONe.net" closed="true" >}}
The network should include ${^{16}\rm{O}},{^{20}\rm{Ne}}$. 
{{< /details >}}

### Step 4: Use your network

| 📋 TASK 4 |
|:--------|
| Edit `inlist_accrete` to have it use your specific network. |

{{< details title="ONe.net" closed="true" >}}
Nothing to do. 
{{< /details >}}


