# jbrowse-jupyter tutorial - locally with JupyterLab

## Introduction

jbrowse-jupyter is a method for running JBrowse 2 in a Jupyter notebook. In this
tutorial, we will look at how to use it inside JupyterLab (run locally on your
computer), and to load a dynamically generated bigwig track.

## Pre-requisites

- Python/pip installed
- Terminal
- Nodejs installed (used to launch a temporary HTTP server)

## Step 1. Create a python virtual env

This is common for most python workflows these days: a virtual env let's you
install python packages locally to a folder rather than to your system

```bash
mkdir project
cd project
python3 -m venv .
source bin/activate
```

Now when you run pip, it is aliased to "bin/pip" that installs locally. Re-run
`source bin/activate` whenever you are revisiting this project folder

## Step 2. Install and launch Jupyter Lab

I run the following commands in the `project` folder in the terminal

```bash
pip install jupyterlab
jupyter lab
```

A new browser window will be launched as a result

Then launch a new notebook

![](img/k1.png)

## Step 3. Install jbrowse-jupyter in the notebook

In one cell do

```
!pip install jbrowse-jupyter
```

And then in the next cell do

```python
from jbrowse_jupyter import launch, create

## basic hg38 linear genome view pre-loaded by selected genome='hg38'
hg38 = create('LGV', genome='hg38')

## random location
hg38.set_location("10:1000..35000")

## launch the server, specify an arbitrary port (this is a dash thing to use a port)
launch(hg38.get_config(), port=3013)
```

This will then open the LGV on hg38 with a given location. Note: you could
`pip install jbrowse-jupyter` on the command line instead of in a notebook cell
if you wanted to.

## Step 4. Launch a local HTTP server

Local files require JBrowse to access files over HTTP. One method of doing this
is starting up an HTTP server. The current jbrowse-jupyter docs refer to
something called the jupyter_proxy_server. I was unable to get this to work. A
simple alternative, is to launch a server manually.

In the `project` folder, in a new terminal tab or instance, you can run

```bash
npx serve . --cors
```

This launches a HTTP server in the `project` folder and allows CORS (allows
cross-origin, in this case, requests from a different port, to access the
files).

The `npx serve` command will start on port 3000 by default, or will print out
the port it is using if 3000 is occupied. Take note of this port.

## Step 5. Download files to your local HTTP server, and add as

In the `project` folder, in yet another terminal tab or instance, run

```bash
wget https://www.encodeproject.org/files/ENCFF303QSJ/@@download/ENCFF303QSJ.bigWig
```

This downloads the simple bigWig file (242Mb) from the ENCODE web server. Then
open up the Jupyter Lab in the browser, and add a new cell.

Enter:

```python
## reference the file, being served by our above npx serve command
hg38.add_track("http://localhost:3000/encff303qsj.bigwig", track_id="mybigwig", overwrite=true)

## open up the track by default by making it part of the default session
hg38.set_default_session(['mybigwig'], false)

## interesting region
hg38.set_location("10:27,369,085..27,494,654")

## important: different port than the first cell, otherwise you get an error.
launch(hg38.get_config(), port=8003)
```

## Step 6. Dynamically create a bigwig in python, write to file, then load it as a track

```python
import pandas as pd
import bioframe
import random

def makedf(n,windowSize):
    chroms = ["chr1"] * n
    starts = range(0, n * 10, windowSize)
    ends = [x + windowSize for x in starts]
    scores = random.choices(range(1, 101), k=n)

    df = pd.DataFrame({
        'chrom': chroms,
        'start': starts,
        'end': ends,
        'score': scores
    })

    return df

df=makedf(10000,10)
print(df)
chromsizes = bioframe.fetch_chromsizes('hg38')
bioframe.to_bigwig(df,chromsizes,"randomScores.bw")
```

## Add our new track, and re-launch an instance

In a new Jupyter Lab cell

```python
hg38.add_track("http://localhost:3000/randomScores.bw", track_id="randomscores", overwrite=True)

hg38.set_default_session(['randomscores'], False)
hg38.set_location("1:1..100,000")
launch(hg38.get_config(), port=8005)
```

## Result

The tutorial.ipynb contains the results of running this script

## Credit

Big thanks to @carolinebridge-oicr for doing all the work!
