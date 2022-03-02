---
author: Christopher Paciorek
layout: default
title: 'Databases and SQL workshop'
---

# 1 Preparation

## 1.1 Required

- Install either the RSQLite R package or the sqlite3 Python package on your laptop.
- Download [this data file](http://www.stat.berkeley.edu/share/paciorek/stackoverflow-2016.db) onto your laptop.

Alternatively, if you have an SCF account you can plan to use one of the SCF machines via ssh or the SCF Jupyterhub.

## 1.2 Optional

If you'd like to be able to replicate the demo I do of creating a PostgreSQL database, you'll need to install PostgreSQL on your computer. Probably the easiest way to enable this is to install Docker, which will allow you to start up a Linux virtual machine on your laptop. See the [Mac instructions](https://docs.docker.com/docker-for-mac/install/) or the [Windows instructions](https://docs.docker.com/docker-for-windows/install/) for how to install Docker. We'll set up PostgreSQL within Docker during the workshop.

For replicating the demo of creating a database (either in SQLite or PostgreSQL) please download and unzip [this file](http://www.stat.berkeley.edu/share/paciorek/tutorial-wikistats-data.zip).

# 2 Schedule 

- 9 am - 10 am: (Session 1; optional) review/introduction to databases, database schema and normalization, and very basic SQL
- 10 am - 11 am: (Session 2) various SQL topics (GROUP BY, JOIN, strings, dates)
- 11 am - noon: (Session 3) set operations and subqueries
- noon - 1 pm: lunch (on your own
- 1 pm - 2:30 pm: (Session 4) window functions, practice problems, indexes, efficiency
- 2:30 pm - 3:30 pm: (Session 5) database administration demo

# 3 Material

The tabs at the top will guide us through the material to be covered in the workshop. We'll primarily be accessing the material in the [SCF Databases tutorial](https://berkeley-scf.github.io/tutorial-databases), interspersed with time to work on SQL practice problems.
