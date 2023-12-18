---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

![](/assets/img/ttbar_diagram.png){:width="50%"}.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## What is this lesson about?
>
> Welcome. In this lesson you will:
> - Learn how to connect everything you have learned so far in order to exercise a full (or almost full) analysis example
> - Learn how to implement the most usual ingredients of an analysis and the construct an analysis chain
>   - Select subsets of the data and Monte Carlo that are aimed at maximizing the significance of your signal
> - Be able to produce plots to show the results
> - Learn about estimating systematics and statistical aspects of the analysis
> - Use all of this to measure some physical quanity and estimate the uncertainties on your measurement
{: .objectives}

**During the live WHEPP workshop this session will focus on equipping you to analyze a POET ROOT file in python and make a basic plot of a kinematic quantity.
Follow the ``Bonus Material" episodes to incorporate multiple simulated samples and apply the analysis selection criteria discussed in the [Event Selection](https://cms-opendata-workshop.github.io/workshopwhepp-lesson-selection/) lesson!**

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

{% include links.md %}