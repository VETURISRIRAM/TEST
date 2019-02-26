# A Developer's Gudie to Platform Data Science

# Table of Contents

1.  [Overview](#org61a1b49)
2.  [Versions, Libraries and Dependencies](#org9d95c6b)
    1.  [Python 3.6](#org5ee7d3e)
        1.  [Why](#org370a412)
        2.  [Where](#org8c54488)
    2.  [Conda](#orgeab39b0)
        1.  [What](#org50ebc62)
        2.  [Why](#org1db65b3)
        3.  [Where](#org3fc9f7b)
    3.  [Attrs and Cattrs](#orgb844c67)
        1.  [What](#org66175a2)
        2.  [Why](#orged7ff76)
        3.  [Where](#org05f6746)
    4.  [Dill](#org42e4302)
        1.  [What](#orgc4b9c87)
        2.  [Why](#org428d272)
        3.  [Where](#orgea7dc53)
3.  [On Platform Types](#org79415ed)
    1.  [Platform types](#org1c9dfe4)
    2.  [Adding a new data source](#org60fab28)
4.  [The Model Runner](#orgd587c7e)
    1.  [Overview](#org9a4fa75)
    2.  [Adapters](#org75f8672)
    3.  [Adding support for a new model type](#org04b22a1)
    4.  [The thing about pickle and pickled models](#org3877720)
5.  [HTTP Status Codes and Their Uses](#orgeb7aa3d)
    1.  [400](#org37ff4d2)
    2.  [500](#org3e20dbe)
    3.  [503](#org2fc11a9)
6. [JIRA Guide to Submit Feedback](#jira_add)




<a id="org61a1b49"></a>

# Overview

This document is a collection of articles describing portions of the UptakeIO
and Python Model Runner codebase. While I've included a few code samples and
brief descriptions, the bulk of this document attempts to explain the *why*
rather than the *what*, as reasons that led to a decision are the most often to
time and change. Importantly, I describe here what dependencies UptakeIO
requires (aside from sckit-learn and Orange 3) and the rationale for why they
were included.


<a id="org9d95c6b"></a>

# Versions, Libraries and Dependencies


<a id="org5ee7d3e"></a>

## Python 3.6


<a id="org370a412"></a>

### Why

Python 3.6 is required because the Orange-Text add on strictly requires Python
3.6 and the Uptake Orange extensions require Orange-Text. The Uptake Orange
extensions require Orange-Text because of some of the API client classes were
used to interact with the Uptake Platform APIs, and the Orange-Text extensions
were used as a reference when building the Uptake Extensions.

Python 3.6 is also required to use the more concise version of `attrs`. In
Python 3.5, type annotations are only allowed in method argument definitions. In
Python 3.6, the places where type annotations are allowed was expanded to
include class attribute deviations. For that reason, the current version of
UptakeIO is incompatible with 3.5.

In short, Python 3.6 is required because of dependency hell. It should be
possible to escape the hell, but it would require re-implementing the
Orange-Text classes that are currently being used.


<a id="org8c54488"></a>

### Where

Python 3.6.X is used as the required version for UptakeIO and as the version of
the base image in the Python Model Runner.


<a id="orgeab39b0"></a>

## Conda


<a id="org50ebc62"></a>

### What

We all know it, for better or for worse.


<a id="org1db65b3"></a>

### Why

We use conda to manage the environment because it's the recommended way to
install Orange and it allows a lot of the packages to interoperate.


<a id="org3fc9f7b"></a>

### Where

Conda is used to manage the environment of both UptakeIO users and the Python Model Runner.


<a id="orgb844c67"></a>

## Attrs and Cattrs


<a id="org66175a2"></a>

### What

`attrs` (and the complementary package `cattrs`) is a library for concisely
specifying data classes in Python with an emphasis on reducing boilerplate. Data
classes refer to classes in OOP that are strictly named containers with few pure
methods. The idea behind data classes is that any objects of that class are
transparent representations of the data they contain. Data classes are easiest
to reason about as souped-up named tuples. Other stricter programming languages
have similar concepts baked in: for example in Scala, data classes are called
**case classes**. The DSE and Model Manager both make heavy use of **case classes**
so it's worthwhile to become familiar with the concept.


<a id="orged7ff76"></a>

### Why

Data classes are used in cases where an object is identical to the data it
contains, and they are especially useful when representing a schema for an
object that needs to be representable in JSON. UptakeIO makes heavy use of data
classes (i.e. **attrs**) because they're a great fit for representing platform
schemas like Model Data Requirements that need to be serialized to JSON and are
shared across Platform services. They also reduce the amount of boilerplate
needed to define a class in Python significantly.


<a id="org05f6746"></a>

### Where

Attrs is used to represent essential Platform objects that are shared between
UptakeIO, Model Manager and APM. In particular it's used to represent
`ModelDataReqs`, `ReadingsDataSource` and `AssetQuery`.


<a id="org42e4302"></a>

## Dill


<a id="orgc4b9c87"></a>

### What

`Dill` is a library for serializing python objects that augments the functionality
in the base Python `pickle` library. It offers the exact same interface as `pickle`.


<a id="org428d272"></a>

### Why

`Dill` is needed to pickle object that use or reference lambdas, and to pickle
class defintions along with the instance of an object. See the section further
down for a more in-depth discussion of the merits of `pickle` vs `dill`.


<a id="orgea7dc53"></a>

### Where

Dill is used in the UptakeIO Python package and in the Orange extensions to
serialize models, and in the python model runner to deserialize models.


<a id="org79415ed"></a>

# On Platform Types


<a id="org1c9dfe4"></a>

## Platform types

Platform types are typed python data classes that match shared platform objects.
They define concepts like:

-   Model Data Requirements
-   Readings Data Source
-   Asset Query

Some types, like data sources, require a small bit of implementation before they are usable
with the platform.

Platform types are defined using data classes or `attrs` in Python and case classes in Scala.


<a id="org60fab28"></a>

## Adding a new data source

The first step to adding a new data source type is to ensure that the data source type is
supported and recognized by both Model Manager and the DSE. Talk to the Insights Engines team
to verify if your data source type is supported.

The next step is to define your data source type's schema.

The next, next step is to ensure that the ModelDataRequirements `sources` member allows your type.
In most cases this means adding your new data source type to a union type of data sources.

    import attr
    from typing import Union
    from uptakeio import ReadingsDataSource, MyDataSource, AssetQuery

    # DataSource here is a type alias
    DataSource = Union[ReadingsDataSource, MyDataSource]

    @attr.s
    class ModelDataReqs:

        sources: DataSource = attr.ib()
        asset_query: AssetQuery = attr.ib()
        # ... etc ...

Next, register a dispatcher:

    # dispatchers.py

    # ...

    class MyDataSourceDispatcher(AbstractDispatcher):

        def fetch(self, args):
    	# ... stuff ...

        def get_metadata(self):
    	# ... stuff ...

    # ...etc...

The `fetch` method should return a dataframe-like object.

Finally, register the dispatcher with UptakeIO.

    # uptakeio.py
    class UptakeIO:
        def __init__(self, api_key: str = None, gateway_key: str = None):
    	# ... stuff ...
    	self.use_dispatcher(ReadingsDataSource, ReadingsDispatcher())
    	# Add this:
    	self.use_dispatcher(MyDataSource, MyDataSourceDispatcher())

And you're done!


<a id="orgd587c7e"></a>

# The Model Runner


<a id="org9a4fa75"></a>

## Overview

The python model runner has a storied history, dating back to the Panduit
POM. Around that time, Panduit had the intention to pilot a few models that
didn't have native support in Uptake Core. To that end, we worked with Tom
Wessels to design a platform to quickly prototype and deploy algorithms that
could side step the complexity of writing PFA. Out of that was born the first
design of the Python model runner, which Uli on the IE team spiked. The POC was
first designed using RabbitMQ to transport messages between the Python model
and the DSE.

As the vision for the Platform product developed, it became apparent that the
platform would need the facility to deploy custom models (often Greg Goff
refers to this as Bring Your Own Model). The original Panduit design was
re-purposed to handle arbitrary python models, which I and Tony worked on over
the course of two weeks to get into a working state.

The current design uses native TCP sockets using the Python 3 standard library
and the Scala standard library to pass messages back and forth within the
context of the DSE Kubernetes pod.

What makes this setup possible is that Kubernetes always schedules containers in
the same pod on the same host machine, meaning that you can do things like share
physical resources like memory, as well as communicate over local sockets. While
we managed to reach a working point with the current TCP model runner, there's
quite a bit more opportunity here to optimize the message passing. Some examples
include using UDP instead of TCP, using Unix sockets or trying Apache Arrow as a
framework for interacting with shared memory.


<a id="org75f8672"></a>

## Adapters

The Python Model Runner has a registered set of adapters that transform DSE
inputs and model outputs and calls the appropriate method from the deserialized
model. They are built into the DSE [here](https://git.uptake.com/projects/DSE/repos/dse-x/browse/infrastructure/containers/python-model-runner/server.py) 
 
We built the model runner this way to standardize dictionary/map-to-table
transformations, and in the process arrived at an interface that was more
generalizeable.

The `BaseModelAdapter` simply passes both input and output data through as-is,
and calls the `__call__` method on whatever object is passed in at init. All
other adapters inherit from it.

The `PxModelArrayAdapter` is the bread-and-butter adapter that turns the native
input map and transforms it into a numpy array, suitable for most ML applications
in Python. This adapter covers 95% of cases, and is probably what you first want
to reach for when subclassing a new adapter.

The `SkModelAdapter`, `KerasModelAdapter` and `OrangeModelAdapter` are for adapting the scikit-learn, keras,
and Orange model scoring methods respectively.

Consult [this blog post](https://sourcemaking.com/design_patterns/adapter) for more information on the Adapter pattern which this
design is based on.


<a id="org04b22a1"></a>

## Adding support for a new model type

Adding support for a new model type essentially boils down to adding the
model types dependencies to the docker image and writing a appropriate
model adapter.

Suppose for example Wes McKiney wants to jump on the hype train and releases
a new deep learning framework called PandasFlow that only accepts pandas
dataframes and we want to add support for it:

1.  First add `pandasflow` to our Docker image build, either by installing in the
    Dockerfile itself or by adding it to the conda environment (preferably the latter).

2.  Add a new `PandasFlowAdapter` class to python model runner code in the dse-x
    repo (the file is named `server.py`).

    For example

        # server.py
        #
        # ... stuff ...

        class PandasFlow(BaseModelAdapter):

            def __init__(self, model):
        	self.super().__init__()
        	self._model = model

            def pre_hook(self, input):
        	return pd.DataFrame(input)

            def run(self, input):
        	return self._model.predict(input)

1.  Add a check for an instance of `PandasFlow` to `wrap_model()`

        # server.py
        from pandasflow import PandasFlow

        # ... stuff ...

        def wrap_model(model_obj):
            # ... stuff ...
            if isinstance(model_obj, PandasFlow):
        	return PandasFlowAdapter(model_obj)
            # ... other checks ...

Adding a check to `wrap_model()` ensures that the model object gets
dispatched to the correct adapter.


<a id="org3877720"></a>

## The thing about pickle and pickled models

Pickle is a module in the Python standard library that serializes and deserializes
Python objects to and from a binary format. There are a few limitations on what can
and can't be pickled. Namely:

1.  A lambda (anonymous) function is not pickleable
2.  References to other objects are done by name. For example:

		import io
		import pickle

		k = 6

		def add_k(x):
			return k + x

		buf = io.BytesIO()
		pickle.dump(add_k, buf)
		buf.seek(0)
		del k

		fn = pickle.load(buf)
		fn(4)

This fails because `k` cannot be found in the environment. Note that `pickle` makes
no attempt to save `k` anywhere.

`Dill` is a third party open source library that expands on pickle and attempts to
serialize more of the environment where possible. The above snippet still doesn't
work in `dill`, but `dill` can in fact handle lambdas.

Another key difference between `pickle` and `dill` is that `dill` attempts to pickle
class definitions when pickling instances whereas `pickle` cannot.

For example, the following works with `dill` but not `pickle`.

    # save.py
    #!/usr/bin/env python
    import sys

    class AddK:

        def __init__(self, k=4):
    	self.k = 4

        def add(self, x):
    	return self.k + x

    def main():
        _, lib, pkl = sys.argv
        if lib == 'dill':
    	import dill as lib
        else:
    	import pickle as lib

        add_k = AddK()

        with open(pkl, 'wb') as f:
    	lib.dump(add_k, f)

    if __name__ == '__main__':
        main()

    # load.py
    #!/usr/bin/env python
    import sys

    def main():
        _, lib, pkl = sys.argv
        if lib == 'dill':
    	import dill as lib
        else:
    	import pickle as lib

        with open(pkl, 'rb') as f:
    	func = lib.load(f)

        print(func.add(5))

    if __name__ == '__main__':
        main()


<a id="orgeb7aa3d"></a>

# HTTP Status Codes and Their Uses

This portion of the guide is intended to assist with debugging status codes
that you may see from time to time when using or developing UptakeIO. Unfortunately,
there's a lot of misunderstanding on what each status code means and when they are
supposed to be used, and many service you'll interact with play fast and loose with codes
or ignore their purpose entirely. This is just the reality of working at a startup. To
that end I've provided descriptions of a few common codes you'll see in the wild and
some leads of things to look for when debugging.


<a id="org37ff4d2"></a>

## 400

   A 400 status code means you dun' goofed. Usually it means there was something wrong with
the data you sent, such as missing parameters or mis-specified arguments. You're most likely
going to see 400s from mistaken user input or when a new version of a service API was
released with a breaking change.


<a id="org3e20dbe"></a>

## 500

A 500 status code implies that the service you are communicating with panicked when
trying to serve your request. It's the equivalent of exiting with a non-zero exit code
in unix. A 500 is *supposed* to be used only to indicate that the service raised an unexpected
error, and should never be used to indicate invalid data. In reality, you may see service
maintainers cut corners here. I strongly recommend notifying channels when you see a 500. Even if
the error occurred dude to bad input, services should exclusively use 4XX codes and never rely
on 500s.


<a id="org2fc11a9"></a>

## 503

A 503 is a gateway status code. It means that a proxy service is not able to
talk to the target downstream service it's supposed to proxy to. Some common reasons
for receiving a 503:

-   You didn't set an Apigee gateway key header or the Apigee gateway key is invalid
-   The service you are trying to talk to is straight up dead

<a id="jira_add"></a>

# GUIDE About How to Submit Feedback and Create Tickets on JIRA

If you are facing any problem in any of the services that Uptake offers like if you think some service is not responding to your request as it was expected to, or if you think something might be considered as a bug in the services, or anything in the similar lines, the best way to bring this to notice is by creating a JIRA ticket explaining the situation. Here is a step by step guide for creating tickets in JIRA. <br>

1) Go to the mentioned site https://jira.uptake.com/ and log in with your Uptake credentials and visit the dashboard. <br>
2) On the top section of the page, you can find an option ‘Create’. Click on it. <br>
3) A new window would pop up which says ‘Create Issue’. In that window, there would be several options mentioned below which you have to select in order to create the issue ticket. <br>
	a) Project : Depending upon the project that you are working on, select the most appropriate project option.<br>
Issue Type : Depending upon the type of problem you are facing, select one of the types. For example, you can just select ‘Bug’.
Dev Team : Depending upon the team that you wish to assign the ticket to, select one of the options. For platform feedback, you can assign it to James Lamb.
Description : Fill in the description about the issue that you are facing. You might want to include the step by step process of what you tried to achieve and what resulted. You can also add your suggestion about how you wish the feature to work.
Environment : You must include your environment details like the operating system you are using and other relevant hardware and software details that might concern with the issue.
Attachments (Strongly Recommended) : You can add some of the screenshots or documents about the issue so that it becomes easy for the person who would attending your ticket to have a complete idea about the issue.
Assignee : You can start typing the person whom you might want to assign the ticket to solve. As you type, the person’s name would appear as a dropdown list. You can assign it to James Lamb if you are have an issue related to the platform and are not sure about the asignee.
Hit ‘Create’ at the bottom of that window to issue the ticket. 
 
All other fields have description about them and you can fill them if you wish to. The ticket would be generated and you can view your tickets under ‘Issues’ tab on the top of the homepage.

