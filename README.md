# EnKF_seir

This code uses an extended SEIR model with age classes in an ensemble data
assimilation system for predicting the spreading of the Coronavirus.  The ensemble
method is the ESMDA (Ensemble Smoother with multiple data assimilation). The
measurements are the accumulated number of deaths and the daily number of
hospitalized.

<p align="center">
  <a href="#installation">Installation</a> *
  <a href="#code-standards">Code standards</a> *
  <a href="#setting-up-an-experiment">Experiment setup</a> *
  <a href="#git-instructions">Git instructions</a> *
  <a href="#definition-of-time">Defintion of time</a> *
  <a href="https://github.com/geirev/EnKF_seir/blob/master/LICENSE">License</a> 
</p>

For model, system description, and examples, see 
https://www.medrxiv.org/content/10.1101/2020.06.11.20128777v1

Initially, model parameters are sampled from prescribed normal distributions.  The
ensemble is integrated forward in time to make a prior ensemble prediction.  The
ESMDA is then used to update the ensemble of model parameters.  After that, the
posterior ensemble is integrated a final time to produce the posterior prediction.

<p align="center">
<img src="doc/HDlog.png" width="300"> <img src="doc/RENS.png" width="300">
</p>


---
# Installation:
## 1. Accessing the code
If you plan to collaborate or contribute anything to the project go for the best installation option.

### 1a. Simple installation option
Make a directory where you will install the following
```bash
git clone git@github.com:geirev/EnKF_seir.git
git clone git@github.com:geirev/EnKF_analysis.git
git clone git@github.com:geirev/EnKF_sampling.git
```

### 1b. Best installation option
Make a personal github account unless you already have one.
Fork the three repositorys listed above.
Next clone the repositories and set upstream to the original repositories where
you need to replace <userid> with your github userid
```bash
git clone git@github.com:<userid>/EnKF_seir.git
pushd EnKF_seir
git remote add upstream https://github.com/geirev/EnKF_seir
#or, if you have set up git-ssh
#git remote add upstream git://github.com:geirev/EnKF_seir
popd

git clone git@github.com:<userid>/EnKF_analysis.git
pushd EnKF_analysis
git remote add upstream https://github.com/geirev/EnKF_analysis
#or, if you have set up git-ssh
#git remote add upstream git://github.com:geirev/EnKF_analysis
popd

git clone git@github.com:<userid>/EnKF_sampling.git
pushd EnKF_sampling
git remote add upstream https://github.com/geirev/EnKF_sampling
#or, if you have set up git-ssh
#git remote add upstream git://github.com:geirev/EnKF_sampling
popd
```

If you are new to git, read the section <a href="#git-instructions">Git instructions</a>


## 2. Install blas, lapack, libfftw3, and Fortran95
<a href="#Mac-OS-X-installation">Additional instructions for installation on Mac OS X</a>
```bash
sudo apt-get update
sudo apt-get install libblas-dev liblapack-dev libfftw3-dev gfortran
```

## 3. Compile the EnKF_sampling library
```bash
cd EnKF_sampling/lib
make BUILD=../../EnKF_seir/build
```
This will put all the .o files as well as libanalysis.a in the same dir as you will use when compiling EnKF_seir

## 4. Compile the EnKF_analysis library
```bash
cd EnKF_analysis/lib
make BUILD=../../EnKF_seir/build
```
This will put all the .o files as well as libanalysis.a in the same dir as you will use when compiling EnKF_seir.
Note that EnKF_analysis depends on EnKF sampling.

## 5. cd EnKF_seir/src
Compile and install executable in target install dir which defaults to BINDIR=/$HOME/bin
```bash
make BINDIR=$HOME/bin
cd ../run
seir
```

## 6. Plotting
If you have tecplot (tec360) there are .lay and .mcr files in run.
For some included plotting options check python/enkf_seir/plot:
```bash
python ../python/enkf_seir/plot/plot.py
jupyter-notebook ../python/enkf_seir/plot/covid.ipynb
```

---
# Setting up an experiment
The following explains how to set up and configure your simulations.

Make a directory where you will run the code.

Initially you only need the file 
```bash
infile.in
```
copy the example file from the run catalogue and run
```bash
seir
```

This command will then run an ensemble prediction without any data assimilation
and a number of files are generated.

## Measurements of deaths, hospitalized and number  of positive cases:
The observations are stored in the file corona.in. See example file for Norway in the run directory.
The file contains the following columns:
```bash
dd/mm-yyyy  "Accumulated number of deaths" "current number of hospitalized" "Accumulated number of cases"
```
A negative or zero data value will not be used.

## Population:
Norwegian population numbers are hard coded and output to two template files.
```bash
population.template  : two columns with population of males and females for each age (0-100++)
agegroups.template   : total population for each agegroup
```
You should edit either one of these with your numbers and save the chosen file to either
```bash
population.in or agegroups.in
```

## Fraction of mild, severe, and fatally ill per age group:
The file
```bash
pfactors.in          
```
is generated with some default values, which can be edited and changed.


## R-reproduction in between age groups:
The model contains 11 age-groups and allows for using different R numbers between different age groups.
Currently, three matrices are generated all with the number 1.00 for all transsmission. These can later be
edited to represent different scenarios.
```bash
Rmatrix_01.in
Rmatrix_02.in
Rmatrix_03.in
```
The actual R-numbers are R = p%R(k) * Rmatrix_k  where P%R(k) comes from the infile.in

## Output files:
```bash
susc_0.dat   susc_1.dat   : Prior and posterior susceptability (accumualted)
recov_0.dat  recov_1.dat  : Prior and posterior recovered (accumualted)
case_0.dat   case_1.dat   : Prior and posterior cases (accumualted)
dead_0.dat   dead_1.dat   : Prior and posterior susceptability (accumualted)
expos_0.dat  expos_1.dat  : Prior and posterior susceptability (current)
hosp_0.dat   hosp_1.dat   : Prior and posterior susceptability (current)
infec_0.dat  infec_1.dat  : Prior and posterior susceptability (current)
obsD.dat                  : Death observations for plotting
obsH.dat                  : Hospitalization observations for plotting
par0.dat par1.dat         : Prior and posterior ensemble of parameters
Rens_0.dat Rens_1.dat     : Prior and posterior ensemble of R(t)
```


---
# Code standards
If you plan to change the code note the following:

I always define subroutines in new modules:

```Fortran90
module m_name_of_routine
! define global variables here
contains
subroutine name_of_sub
! define local variables here
...
end subroutine
end module
```
in the main program you write

```Fortran90
program seir
use m_name_of_routine
call  name_of_routine
end program
```

The main program then has access to all the global variables defined in the module, and
knows the header of the subroutine and the compiler checks the consistency between the call
and the subroutine definition.

   make new  -> updates the dependencies for the makefile
   make tags -> runs ctags (useful if you use vim)

For this to work install the scripts in the ./bin in your path and install ctags



---
# Git instructions
When working with git repositories other than the ones you own, and when you expect to contribute to the code,
a good way got organize your git project is described in https://opensource.com/article/19/7/create-pull-request-github

This organization will allow you to make changes and suggest them to be taken into the original code through a pull request.

So, you need a github account.
Then you fork the repository to your account (make your personal copy of it) (fork button on github.com).
This you clone to your local system where you can compile and run.

```bash
git clone https://github.com/<YourUserName>/EnKF_seir
git remote add upstream https://github.com/geirev/EnKF_seir
git remote -v                   #   should list both your local and remote repository
```

To keep your local master branch up to date with the upstream code (my original repository)
```bash
git checkout master             #   unless you are not already there
git fetch upstream              #   get info about upstream repo
git merge upstream/master       #   merges upstream master with your local master
```

If you want to make changes to the code:
```bash
git checkout -b branchname      #   Makes a new branch and moves to it
```
Make your changes
```bash
git add .                       #   In the root of your repo, stage for commit
git status                      #   Tells you status
git commit                      #   Commits your changes to the local repo
```

Push to your remote origin repo
```bash
git push -u origin branchname   #   FIRST TIME to create the branch on the remote origin
git push                        #   Thereafter: push your local changes to your forked  origin repo
```


To make a pull request:
1. Commit your changes on the local branch
```bash
git add .                       #   In the root of your repo, stage for commit
git status                      #   Tells you status
git commit                      #   Commits your changes to the local repo
```

2. Update the branch where you are working to be consistent with the upstream master
```bash
git checkout master             #   unless you are not already there
git fetch upstream              #   get info about upstream repo
git merge upstream/master       #   merges upstream master with your local master
git checkout brancname          #   back to your local branch
git rebase master               #   your branch is updated by adding your local changes to the updated master
```

3. squash commits into one (if you have many commits)
```bash
git log                      #   lists commits
git rebase -i indexofcommit  #   index of commit before your first commit
```
Change pick to squash for all commits except the latest one.
save and then make a new unified commit message.
```bash
git push --force             #   force push branch to origin
```

4. open github.com, chose your branch, make pullrequest, check that there are no conflicts

Then we are all synced.

If you manage all this you are a git guru.
Every time you need to know something just search for git "how to do something" and there are tons of examples out there.

For advanced users:
Set the sshkey so you don't have to write a passwd everytime you push to your remote repo: check settings / keys tab
Follow instructions in
   https://help.github.com/en/github/using-git/changing-a-remotes-url

To make your Linux terminal show you the current branch in the prompt include the follwoing in your .bashrc
```bash
parse_git_branch() {
     git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}
export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[1;31m\]\w\[\033[0;93m\]\$(parse_git_branch)\[\033[0;97m\]\$ "
```

---
# Definition of time
The model uses the following definitions for time:

"Startdate" e.g., 01/03-2020  denote that the model runs from 00:00am 01/03-2020, and this time is
set to t=0.0. (see the table below)

The system integreates the ensemble of models forward one day at the time, starting from 00:00 to
24:00 and thus outputs the solutions at midnight end of the day. Thus, the output files will
contain entries like 

```
 model time                                                         Output date
 0.00000E+00  Corresponds to 00:00 in the morning of the start day  (29/02-2020)
 0.10000E+01  Corresponds to 24:00 in the night of the start day    (01/03-2020)
 0.20000E+01  Corresponds to 24:00 in the night of day 2            (02/03-2020)
 0.30000E+01  Corresponds to 24:00 in the night of day 3            (03/03-2020)
 0.40000E+01  Corresponds to 24:00 in the night of day 4            (04/03-2020)
 0.50000E+01  Corresponds to 24:00 in the night of day 5            (05/03-2020)
```

## Intervention times:
If an intevention time is defined as 15/03-2020 the intervention is avtive (through values of R)
from the morning on the 15/03 at 00:00am, which corresponds to model time t>14.0.  This means that
R(t) switches from R1 to R2 from the morning of 15/03. Accordingly ir swiches from 1 to 2 for using
the correct Rmat(ir).

## Measurement times
A measurement for a particular day is located at midnight that day. Thus, given a measurement from
corona.in, e.g.,
```bash
 03/03-2020    7   154   2206
```
will be located at the same time as the output of day 3, i.e., the end of the day.

---
# Mac OS X installation
These are just amendments to GE’s instructions for those on Mac OS X
(need git and homebrew)

Instead of doing the apt-get for libraries do:

brew install gcc
brew install fftw
brew install openblas
brew install lapack

Follow instructions on the git, with cloning etc, until step 5:
Still do: cd EnKF_seir/src

But then go into the makefile and delete “/usr/lib/x86_64-linux-gnu/libfftw3.so.3” and replace that with “-lfftw3”

(mine is linked here: https://github.com/clairevalva/EnKF_seir/blob/master/src/makefile)

So the line should look like: “LIBS =  ./libsampling.a ./libenkfanalysis.a -llapack -lblas  -llapack -lfftw3”

Continue following the instructions where GE left off, except you run executables on the mac as ./seir rather than seir
