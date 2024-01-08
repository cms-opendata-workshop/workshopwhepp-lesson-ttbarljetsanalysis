---
title: "Have some Coffea"
teaching: 20
exercises: 20
questions:
- "Why do we use coffea?"
- "What are coffea main components?"
- "What are schemas?"
objectives:
- "Learn what coffea is and what the logic is of its different compoonets"
- "Learn about what schemas are used for and how to modify them"
- "Make your own histogram!"
keypoints :
- "Coffea is a framework which builds upons several tools to make columnar analysis more efficient in HEP."
- "Schemas are a simple way of rearranging our data content so it is easier to manipulate"
---

## Coffea

Awkward arrays let us access data in a columnar fashion, but that's just the first part of doing an analysis. [Coffea](https://coffeateam.github.io/coffea/) builds upon this foundation with a variety of features that better enable us to do our analyses. These features include:

* **Hists** give us ROOT-like histograms. Actually, this is now a [standalone package](https://hist.readthedocs.io/en/latest/), but it has been heavily influenced by the (old) coffea hist subpackage, and it's a core component of the coffea ecosystem.

* **NanoEvents** and **Schemas** allows us to apply a schema to our awkward array. This schema imposes behavior that we would not have in a simple awkward array, but which makes our (HEP) lives much easier. On one hand, it can serve to better organize our data by providing a structure for naming, nesting, and cross-referencing fields; it also allows us to add physics object methods (e.g., for LorentzVectors).

* **Processors** are coffea's way of encapsulating an analysis in a way that is deployment-neutral. Once you have a Coffea analysis, you can throw it into a processor and use any of a variety of executors (e.g. Dask, Parsl, Spark) to chunk it up and run it across distributed workers. This makes scale-out simple and dynamic for users.

* **Lookup tools** are available in Coffea for any corrections that need to be made to physics data. These tools read a variety of correction file formats and turn them into lookup tables.

In summary, coffea's features enter the analysis pipeline at every step. They improve the usability of our input (NanoEvents), enable us to map it to a histogram output (Hists), and allow us tools for scaling and deployment (Processors).




### Coffea NanoEvents and Schemas: Making Data Physics-Friendly

Before we can dive into our example analysis, we need to spruce up our data a bit.

Let's turn our attention to `NanoEvents` and `schemas`. *Schemas* let us better **organize our file and impose physics methods** onto our data. There exist **schemas for some standard file formats**, most prominently **NanoAOD** (which will be, in the future, the main format in which CMS data will be made open), and there is a `BaseSchema` which operates much like uproot. The coffea development team welcomes community development of other schemas.

For the purposes of this tutorial, **we already have a schema**.  Again, this was prepared already by Mat Adamec for the [IRIS-HEP AGC Workshop 2022](https://indico.cern.ch/e/agc-tools-2). Here, however, we will try to go into the details.

Before we start, don't forget to include the libraries we need, including now the coffea ones:

~~~
import uproot
import awkward as ak
import hist
from hist import Hist
from coffea.nanoevents import NanoEventsFactory, BaseSchema
~~~
{: .language-python}

Remember the output we had above.  After loading the file, we saw a lot of branches.  Let's zoom in on the muon-related ones here:

~~~
'numbermuon', 'nmuon_e', 'muon_e', 'nmuon_pt', 'muon_pt', 'nmuon_px', 'muon_px', 'nmuon_py', 'muon_py', 'nmuon_pz', 'muon_pz', 'nmuon_eta', 'muon_eta', 'nmuon_phi', 'muon_phi', 'nmuon_ch', 'muon_ch', 'nmuon_isLoose', 'muon_isLoose', 'nmuon_isMedium', 'muon_isMedium', 'nmuon_isTight', 'muon_isTight', 'nmuon_isSoft', 'muon_isSoft', 'nmuon_isHighPt', 'muon_isHighPt', 'nmuon_dxy', 'muon_dxy', 'nmuon_dz', 'muon_dz', 'nmuon_dxyError', 'muon_dxyError', 'nmuon_dzError', 'muon_dzError', 'nmuon_pfreliso03all', 'muon_pfreliso03all', 'nmuon_pfreliso04all', 'muon_pfreliso04all', 'nmuon_pfreliso04DBCorr', 'muon_pfreliso04DBCorr', 'nmuon_TkIso03', 'muon_TkIso03', 'nmuon_jetidx', 'muon_jetidx', 'nmuon_genpartidx', 'muon_genpartidx', 'nmuon_ip3d', 'muon_ip3d', 'nmuon_sip3d', 'muon_sip3d'
~~~
{: .output}

By default, uproot (and `BaseSchema`) treats all of the muon branches as distinct branches with distinct data. This is not ideal, as some of our data is redundant, e.g., all of the `nmuon_*` branches better have the same counts. Further, we'd expect all the `muon_*` branches to have the same shape, as each muon should have an entry in each branch.

The first benefit of instating a schema, then, is a **standardization of our fields**. It would be more succinct to create a general `muon` **collection** under which all of these branches (in NanoEvents, fields) with identical size can be housed, and to scrap the redundant ones. We can use `numbermuon` to figure out how many muons should be in each subarray (the counts, or offsets), and then fill the contents with each `muon_*` field. We can repeat this for the other branches.

We will, however, use a custom schema called `AGCSchema`, whose implementation resides in the `agc_schema.py` file you just downloaded.

Let's open our example file again, but now, instead of directly using uproot, we use the `AGCSchema` class.  

~~~
from agc_schema import AGCSchema
agc_events = NanoEventsFactory.from_root('root://eospublic.cern.ch//eos/opendata/cms/upload/od-workshop/ws2021/myoutput_odws2022-ttbaljets-prodv2.0_merged.root', schemaclass=AGCSchema, treepath='events').events()
~~~
{: .language-python}

For *NanoEvents*, there is a slightly different syntax to access our data. Instead of querying `keys()` to find our fields, we query fields. We can still access specific fields as we would navigate a dictionary (`collection[field]`) or we can navigate them in a new way: `collection.field`.

Let's take a look at our fields now:

~~~
agc_events.fields
~~~
{: .language-python}

~~~
['muon', 'fatjet', 'jet', 'photon', 'electron', 'tau', 'met', 'trig', 'btag', 'PV']
~~~
{: .output}

We can confirm that no information has been lost by querying the fields of our event fields:

~~~
agc_events.muon.fields
~~~
{: .language-python}

~~~
['pt', 'px', 'py', 'pz', 'eta', 'phi', 'ch', 'isLoose', 'isMedium', 'isTight', 'isSoft', 'isHighPt', 'dxy', 'dz', 'dxyError', 'dzError', 'pfreliso03all', 'pfreliso04all', 'pfreliso04DBCorr', 'TkIso03', 'jetidx', 'genpartidx', 'ip3d', 'sip3d', 'energy']
~~~
{: .output}

So, aesthetically, everything is much nicer. If we had a messier dataset, the schema can also **standardize our names to get rid of any quirks**. For example, every physics object property in our tree has an `n*` variable which, if you were to check their values, you would realize that they repeat.  They give just the number of objects in the field.  We need only one variable to check that, and for the muons would be `numbermuon`. This sort of features are irrelevant after the application of the schema, so we don't have to worry about it.

There are also other benefits to this structure: as we now have a collection object (`agc_events.muon`), there is a **natural place to impose physics methods**. By default, this collection object does nothing - it's just a category. But we're physicists, and we often want to deal with Lorentz vectors. Why not treat these objects as such?

This behavior can be built fairly simply into a schema by specifying that it is a `PtEtaPhiELorentzVector` and having the appropriate fields present in each collection (in this case, `pt`, `eta`, `phi` and `e`). This makes mathematical operations on our muons well-defined:

~~~
agc_events.muon[0, 0] + agc_events.muon[0, 1]
~~~
{: .language-python}

~~~
<LorentzVectorRecord ... y: -17.7, z: -11.9, t: 58.2} type='LorentzVector["x": f...'>
~~~
{: .output}

And it gives us access to all of the standard LorentzVector methods, like $$\Delta R $$:

~~~
agc_events.muon[0, 0].delta_r(agc_events.muon[0, 1])
~~~
{: .language-python}

~~~
2.512794926098977
~~~
{: .output}

We can also access other LorentzVector formulations, if we want, as the conversions are built-in:

~~~
agc_events.muon.x, agc_events.muon.y, agc_events.muon.z, agc_events.muon.mass

~~~
{: .language-python}

~~~
/usr/local/venv/lib/python3.10/site-packages/awkward/_connect/_numpy.py:195: RuntimeWarning: invalid value encountered in sqrt
  result = getattr(ufunc, method)(
(<Array [[50.7, 0.0423], ... [21.1, -1.38]] type='15090 * var * float32'>, <Array [[-16.9, -0.791], ... [-22.6, 3.31]] type='15090 * var * float32'>, <Array [[-7.77, -4.14], ... [-11, -6.83]] type='15090 * var * float32'>, <Array [[0.103, 0.106], ... [0.105, 0.106]] type='15090 * var * float32'>)
~~~
{: .output}

NanoEvents can also impose other features, such as **cross-references** in our data; for example, linking the muon jetidx to the jet collection. This is not implemented in our present schema.

### Let's take a look at the agc_schema.py file
Anyone, in principle, can write a schema that suits particular needs and that could be *plugged* into the coffea insfrastructure.  Here we present a challenge to give you a feeling of the kind of arrangements schemas can take care of.

> ## **Challenge**: Finding the corrected jet energy in the LorentzVector
>
> If you check the variables above, you will notice that the `jet` object has an energy `e` recorded but also, as you learn from the physics objects lesson, `corre`, which is the corrected energy.  You can also realize about this if you dump the fields for the jet:
> 
> ~~~
> agc_events.jet.fields
> ~~~
> {: .language-python}
>
> You should find that you can see the `corre` variable.
> 
> Inspect and study the file `agc_schema.py` to see how the `corre` energy was added as the energy to the LorentzVector and not the uncorrected `e` energy.  The changes for `_e` should remain valid for the rest of the objects though.  Note that you could correct for the `fatjet` also in the same line of action.
>
{: .challenge}


> ## Challenge! Make your own plot!
>
> You've now seen how to make basic plots of the variables in these ROOT files, as well as how to mask
> the data based on other variables. 
> 
> You are challenged to make at least one plot, making use of the data file and tools we
> learned about in this lesson. It doesn't matter what it is...you can even just reproduce the plots
> we asked you to make in this lesson. To save the plot, you'll want to add something like 
> the following line of code after you make any of the plots.
> 
> ~~~
> plt.savefig('my_plot.png')
> ~~~
> {: .language-python}
> The figure will show up in the directory in both docker and on your local machine.
>
> After you make your plot, add it to [these Google Slides](https://docs.google.com/presentation/d/16bN4acR78S0JB7L3NI78I7R640beRtQjvhuYoYgrDek/edit?usp=sharing). Edit the slides to add your name so that we can credit you for your great work! :)
> 
> Good luck!
>
{: .challenge}
