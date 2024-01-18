Snjob+++
title = 'Data Pipeline'
date = 2024-01-16T17:50:54-07:00
draft = true
+++


# Project Overview
This project will simulate a end to end data pipeline for [Data First Jobs](www.datafirstjobs.com). It will consist of the following tools:
    * Kafka
    * Airflow
    * DBT
    * Metabase

Since the website doesn't have too much data at the moment, I will need to generate fake data to push through Kafka to simulate some real time stream processing, with flink thrown in there to handle the more "complex" transformation. We will feed this fake data through Kafka + Flink and save the results to a Postgres DB that will serve as an analytics DB as well as for meta data.

From there we will schedule DAGS with Airflow utilizing DBT for the transformation layer, that will then connect to a GCP BigQuery instance, and from there the data will be feed through Metabase ready for you to view!

I will look to add pipeline reporting tools such as Grafana, and promethesus, for the pipeline. I will host this on a server so that you can view it. 

I am working on a plan that will allow you click a few buttons that will simulate data being generated where you can see in real time as it flows through the pipeline. For now it will just have to live in a public repo.


