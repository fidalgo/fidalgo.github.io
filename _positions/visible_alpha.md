---
employer: Visible Alpha
title: Software Engineer
start_date: 2017-10-01
technologies: Ruby, Rails, PostgreSQL, ElasticSearch, ActiveMQ, Google PubSub, Cloudformation
---

### Context
Remote backend software engineer working on a platform to allow to connect research providers with research consumers.
When a investment bank creates a research document on some company shares or asset, they need to track who is consuming their research and by regulation,
they need to be payed. From the analyst perspective, also by regulation, he needs to have a track record of consumed research. Visible Alpha provides an even broader platform to also
collect some insights on models (to predict and beat the street value).

### Projects
* Working on the research processing and indexing system, that works by receiving the Research Reports, mainly in a PDF form, extract the information and store it for analysis and user consumption.
  * The information extracted is used to create correlations with companies and assets available in stock exchange markets and also on other metadata like the industry the company operates and it's markets.
  * Another vector are the Events, where the companies and research providers create to discuss assets, doing some roadshows or present the company results. Here we also process the Event information and it's metadata
   to enrich it with useful associated data.
  * The tech stack include Ruby, Rails, PostgreSQL, ElasticSearch, Cloudformation, Google PubSub, ActiveMQ, Redis and AWS.
