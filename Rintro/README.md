# R intro workshop

Planned for late Sept/early Oct 2016 at UofA.

## Audience

* ABMI-ers
* BAM affiliates
* Students from Boutin/Bayne
* Other interested colleagues.
 
## Level

* Basic level programming is assumed, but we'll run over the basics (and it will be part of the handout).
* Some table/database knowledge is assumed.

## Need feedback on

1. Have you ever used any scripting languages or command line tool? (Yes/No)
2. Have you ever used R? (Yes/No)
3. What OS do you use most often for work? (Windows/OS X/Linux/Other)

## Software

* We will use R (command line and Rstudio)
* ODBC and database connections, depending on preferred platform for participants (probably MS Access, but setting up connections is outside of the scope)
* Some text editor (Atom, Notepad2, etc)

## Outline

Morning (9-12): R programming basics

* Navigating the jungle (search tools, help, demos, vignettes, task views, examples, mailing lists)
* Installing libraries
* Data types and structures
* I/O (reading and writing data), maybe database connections and queries
* Understanding basic behaviours (indexing, droping, recycling, factors, names)
* Programming constructs (loops)
* Functions, OO concepts, methods
* Repeated function calls in loops and in vectorized expressions (apply)

Afternoon (13-16): high performance computing

* Manipulating big data: long and wide formats (pivot tables, dense vs. sparse representations)
* Checking data, summaries
* Aggregating big tables, summarizing proportional data (e.g. percent cover)
* Manipulating factor levels
* Manipulating attributes (gsub, grep, substr, strsplit)
* Combining tables (merge operations, filtering, matching, subsetting)
* Using multiple cores (mostly focusing on socket clusters): distributed computing concepts
* Running scripts from command line
