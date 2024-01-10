+++
title = 'DF Jobs Architecture'
date = 2024-01-10T12:10:46-07:00
draft = false
+++

# DF Jobs 
Data First Jobs is a job board for data professionals across industries, tech stacks, and more.

It pulls data from third-party job board apis, parses their data and maps it to my apps models using a json schema. 

I've built an internal as well as an external API. The internal API servers the jobs to my front end, using just vanilla js, whereas the external one will just be used to expose jobs that are posted directly on DF Jobs (none currently).
## DF Jobs Architecture

I understand many of this will be overkill, but this project is particualry motivating and showcases my set of skills from development skills, DevOps, data engineering, as well as analytic engineering and data analysis. Perfect for a portfolio piece, and hopefully a nice little income stream!

I started out as a Data Analyst where I've done anything from click attribution to measure entrypoints > product sign ups, to a predictive model prediciting user success chat volumes - to 92% accuracy! 

But my true ability is my development skill set. I've taught myself to code a few years ago, and have build various apps that are live! Data engineering is the perfect subset between development skills, and analytical prowness, and an ever evolving industry.

This is the design of DF Jobs production server as well as a seperate server hosting data pipelines and other tooling for an end to end application to data visualization.

Visit [Data First Jobs](https://datafirstjobs.com) website to see it live!

![DF Jobs Architecture](/images/DFJobs.png)
