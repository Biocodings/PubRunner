![Logo](logo.png)

![DOI: 10.5281/zenodo.545646](https://zenodo.org/badge/78131113.svg)

PubRunner's website is at http://www.pubrunner.org. This project was part of the [January 2017 NCBI Hackathon](https://www.ncbi.nlm.nih.gov/news/11-17-2016-biomedical-informatics-hackathon/)

# What is biomedical text mining?

Biomedical text-mining (natural language processing) tools are used for a variety of purposes related to health outcomes research. They can aggregate knowledge from large quantities of published academic literature, making the task of perusing the latest literature a much easier task. They help with guided search through PubMed, build protein-protein interaction networks automatically, find interactions between genes and diseases and lots more. But there's a big problem!

# What's the problem?

These analyses are only as accurate as the underlying text being analysed (generally abstracts from PubMed). And the problem is that there are new abstracts published daily. These analyses are rarely kept up-to-date with the latest publications. So we want to solve the problem of rerunning an analysis using the latest publications and doing it regularly.

# Why should we solve it?

We want text mining analyses to be a reliable method to peruse the latest publication and understand the latest knowledge on how different biomedical concepts (e.g. proteins. drugs. etc) interact. If this happens, then text mining could become a regular tool for biomedical researchers.

# What is PubRunner?

PubRunner is a framework which runs on a user defined schedule allowing you to download latest PubMed abstracts,
run them through your favorite text mining tool and then uploads the results to public FTP. Additionally the user has the option to post a link to their FTP on the public PubRunner website (www.pubrunner.org).

The overview below shows how PubRunner manages a "black-box text-mining tool".

![Overview diagram](overview.png)

# How to use PubRunner

## Installation options:

We provide two options for installing PubRunner: Docker or directly from Github.

### Docker

The Docker image contains PubRunner as well as a webserver and FTP server in case you want to deploy the FTP server. It does also contain a web server for testing the PubRunner main website (but should only be used for debug purposes).

1. `docker pull ncbihackathons/pubrunner` command to pull the image from the DockerHub
2. `docker run ncbihackathons/pubrunner` Run the docker image from the master shell script
3. Edit the configuration files as below

### Installing PubRunner from Github

1. `git clone https://github.com/NCBI-Hackathons/PubRunner.git`
2. Edit the configuration files as below
3. `sh server/pubrunner.sh` to test
4. Add cron job as required (to execute pubrunner.sh script)

### Configuration

#### Update the JSON file [server/tools.json](https://github.com/NCBI-Hackathons/PubRunner/blob/master/server/tools.json) with your tool's information


PubRunner keeps track of every tool in the tools.json file. If you use the website to add a tool, it will provide skeleton code that should be completed and added to the tools.json file. Below is an example of a completed tools.json file

```
[
    {
        "active": true,
        "authentication": "FAKE_AUTH_CODE",
        "codeurl": "https://github.com/NCBI-Hackathons/PubRunner/tree/master/server/tools/CountWords/0.1",
        "command": "python",
        "dataurl": "ftp://ftp.bcgsc.ca/public/jlever/pubrunner/CountWords/0.1/",
        "description": "Calculates word counts of abstracts",
        "email": "someemail@email.com",
        "lastRun": "04-13-2017",
        "main": "CountWords.py",
        "name": "CountWords",
        "success": true,
        "timeout": 100000,
        "version": "0.1"
    }
]
```

Some parameters in this file are supposed to be provided by the tool’s authors, while some are filled by PubRunner. Tool authors need to specify the name, description, version number and URLs for their tool. PubRunner also needs technical details to run the tool, such as the command to launch (python, perl, java…), the main file to be launched and flags (if any). By default, PubRunner kills any process that runs more than sixty minutes, but users are free to set this limit higher for their tool. The other parameters are filled by PubRunner and correspond to whether the tool ran successfully recently, when it was last run and if it is active. When a tool fails to run a few months in a row, automatic updates will be disabled by PubRunner.  

#### Add FTP credentials to [server/settings.py](server/settings.py)

The settings.py file (shown below) defines a few parameters in order for PubRunner to run. That includes the paths for data to be stored, i.e. the MEDLINE abstract (DB), the output of tools (OUTPUT), and the tools themselves (TOOLS). When tools fail running, PubRunner can allow more tries, in case the tool failure is due to some random error (e.g., an HTTP request that punctually timed out). The number of attempts PubRunner can give to each tool is set by the FAIL_LIMIT setting.

The only settings that need to be input are the FTP settings.

```
import os

"""
Definition of config parameters
"""

### Static, do not touch
VERSION = 0.1
ROOT = os.path.dirname(os.path.realpath(__file__)) + "/"

### General
DB = "medline/"
TOOLS = "tools/"
OUTPUT = "output/"
FAIL_LIMIT = 3

# Whether to use FTP or a local directory (that should be mounted as an FTP)
USE_FTP = True

### FTP
FTP_ADDRESS = "INSERT FTP ADDRESS HERE"
FTP_USERNAME = "INSERT FTP USERNAME HERE"
FTP_PASSWORD = "INSERT FTP PASSWORD HERE"

### LOCAL DIRECTORY
LOCAL_DIRECTORY = ""
```

# Testing

We tested three different scripts with PubRunner. They can be found in [server/tools/](server/tools/) . Two of the tools [CountWordsError](server/tools/CountWordsError/0.1) and [Error](server/tools/Error/0.1/Error.py) are designed to fail either randomly or consistently to make sure that PubRunner can manage failures. The third script [CountWords](server/tools/CountWords/0.1) does a basic word count of PubMed abstracts as a very simple example usage of PubRunner and should not fail.

# Additional Functionality
### DockerFile

PubRunner comes with a Dockerfile which can be used to build the Docker image.

  1. `git clone https://github.com/NCBI-Hackathons/PubRunner.git`
  2. `cd server`
  3. `docker build --rm -t pubrunner/pubrunner .`
  4. `docker run -t -i pubrunner/pubrunner`
  
### FTP

There is a Docker image that combines PubRunner with an FTP server so that you can host the data locally on the same machine if needed.

  1. `docker pull ftp_pubrunner` command to pull the image from the DockerHub (hyperlink
  2. `docker run -p 21:21 ftp_pubrunner` Run the docker image from the master shell script

### Website

There is also a Docker image for hosting the main website. This should only be used for debug purposes.

  1. `git clone https://github.com/NCBI-Hackathons/PubRunner.git`
  2. `cd Website`
  3. `docker build --rm -t pubrunner/website .`
  4. `docker run -t -i pubrunner/website`
