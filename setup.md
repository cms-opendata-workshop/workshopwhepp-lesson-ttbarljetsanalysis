---
title: Setup
---
> ## Prerequisites
>
> To follow this lesson you need to start the [*python tools Docker container*](https://cms-opendata-workshop.github.io/workshopwhepp-lesson-docker/03-docker-for-cms-opendata/index.html#python-tools-container) from the pre-exercises:
>
> If you have already successfully installed and run the Docker example with python tools, then you need only execute the following command.
> ~~~
> docker start -i my_python  #give the name of your container
> ~~~
> {: .language-bash}
> If this doesn't work, return to the [*python tools Docker container*](https://cms-opendata-workshop.github.io/workshopwhepp-lesson-docker/03-docker-for-cms-opendata/index.html#python-tools-container) lesson to work through how to start the container.  
{: .prereq}

> ## Additional prerequisites - extra libraries
> In order to use the `coffea` framework for our analysis, we need to install these additional packages directly in our container.  We are adding
> `cabinetry` as well because we will use it later in our last episode. *This can take a few minutes to install.*
>
> Once you have launched the Docker container, type the following commands in that environment (terminal).
> ~~~
> pip install vector hist mplhep coffea=0.7 cabinetry
> ~~~
> {: .language-bash}
>
> Also, download [this file](https://raw.githubusercontent.com/cms-opendata-workshop/workshop2022-lesson-ttbarljetsanalysis-payload/master/trunk/agc_schema.py), which is our starting schema.  Directly in your `/code` area (or locally in your `cms_open_data_python` directory) you can simply do:
>
> ~~~
> wget  https://raw.githubusercontent.com/cms-opendata-workshop/workshop2022-lesson-ttbarljetsanalysis-payload/master/trunk/agc_schema.py
> ~~~
> {: .language-bash}
>
{: .prereq}


{% include links.md %}
