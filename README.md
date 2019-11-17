
# Getting Started with Containers on HPC

View this on [GitHub Pages](https://supercontainers.github.io/sc19-tutorial/).

## ECP Supercontainers Tutorial Session

<div style="text-align:center"><img src="images/ecp.jpg" width="250"></div>

## Details

Half-day Tutorial Session

Venue: Supercomputing Conference 2019 (SC19')

Date: Sunday, 17 November 2019 1:30pm - 5pm

Room: 207

Location: Denver, CO, USA

Link: [Container Computing for HPC and Scientific Workflows @ SC19](https://sc19.supercomputing.org/presentation/?id=tut129&sess=sess206)

Topic Area: Programming Models & Systems Software

Keywords: Containerized HPC, System Software and Runtime Systems, Scientific Software Development, DevOps

## EC2 Login

hostname: tutXX.supercontainers.org

user: tutorial

password: HPCLinux12!



## Abstract

Container computing has revolutionized the way applications are developed and delivered. It offers opportunities that never existed before for significantly improving efficiency of scientific workflows and easily moving these workflows from the laptop to the supercomputer. Tools like Docker, Shifter, Singularity and Charliecloud enable a new paradigm for scientific and technical computing. However, to fully unlock its potential, users and administrators need to understand how to utilize these new approaches. This tutorial will introduce attendees to the basics of creating container images, explain best practices, and cover more advanced topics such as creating images to be run on HPC platforms using various container runtimes. The tutorial will also explain how research scientists can utilize container-based computing to accelerate their research and how these tools can boost the impact of their research by enabling better reproducibility and sharing of their scientific process without compromising security. 

This is an updated version of the highly successful tutorial presented at SC16, SC17, SC18. It was attended by more than 100 people each year. The 2018 tutorial was very highly rated with 2.8 / 3 stars for “would recommend” and 4.3 / 5 stars for overall quality.

## Prerequisites

This is a hands-on tutorial. Participants should bring a laptop and load or pre-install a terminal and/or ssh client in advance to make best use of time during the tutorial.  We will be providing training user accounts to both pre-configured EC2 instances as well as the Cori Supercomputer at NERSC.

<div style="text-align:center"><img src="images/AWS_logo.png" width="250"></div>

This tutorial is supported by the Amazon AWS Machine Learning Research Awards. EC2 images and temporary login credentials will be distributed onsite at the tutorial.

After the tutorial, you can boot our tutorial image yourself on Amazon EC2 to run through the tutorial again. We recommend you use your own EC2 key and change the password.

US-West-Oregon: ami-0fe12765123c6a840 


### Optional Prerequisites

Users can also install Docker and Singularity prior to attending the tutorial session. Here, it may be beneficial to create a docker and sylabs (singularity) account in advance at https://cloud.docker.com/ and https://cloud.sylabs.io/ This accounts will be needed to create images on docker cloud/dockerhub and sylabs cloud.

[Install Singularity on Linux](https://sylabs.io/guides/3.3/user-guide/)

[Install Singualrity on Mac](https://repo.sylabs.io/desktop/) (Alpha)

[Install Docker for Desktop](https://www.docker.com/products/docker-desktop)

## Questions

You can ask questions verbally or with this [Google Doc](https://docs.google.com/document/d/11gMZ-T7iA5XiRWPLYIqX7Gqv7RMb-NF9kzGYHrnOi04/edit?usp=sharing).
Please append your question below the others in the document.

We have also created a Slack Team for this.  The invitation link is [here](https://join.slack.com/t/hpc-containers/shared_invite/enQtODI3NzY1NDU4OTk5LTUxOTgyOWJmYjIwOWI5YWU2MzBhZDI3Zjc1YmZmMjAxZjgzYzk4ZWEwNmFlNzlkOWI0MGNlZDNlMTBhYTBlOWY).

## Schedule


13:30 – 13:45 [Introduction to Containers in HPC](slides/sc19_tutorial_intro.pdf) (Younge)

13:45 – 14:15 [How to build your first Docker container](/01-hands-on.md) (Canon)

14:15 – 14:45 [How to deploy a container on a supercomputer](/02-hands-on.md) (Canon)

14:45 – 15:00 [Best Practices](slides/sc19_tutorial_bestpract.pdf) (Canon)

15:00 – 15:30              -- Break –

15:30 – 16:00 [Running an HPC app on the E4S container](slides/E4S_SC19.pdf) (Shende)

16:00 - 16:30 [How to build a Singularity container image](/03-hands-on.md) (Arango)

16:30 - 16:50 [Running Singularity on a supercomputer & adv features](/04-hands-on.md) (Arango)

16:50 - 17:00 [Success Stories & Summary](slides/sc19_tut_summary.pdf) (Canon)



