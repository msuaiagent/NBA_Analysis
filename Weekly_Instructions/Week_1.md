# Week 1: Getting Started

Hey everyone, welcome to week 1 of the NBA Draft Prediction model!

This week, we'll cover data manipulation skills in Pandas, as well as some basic exploratory data analysis before we begin training our model next week. 

## Part 1: Setting up your development environment

### Creating a new Git Repository

Begin by creating a new git repository on GitHub titled whatever you want but making sure it's informative (something like "NBA_Draft_Prediction_Model). Add a README as well, we can worry about filling it out later on. 

Then, like as described in the Week 0 instructions, clone the repository to your local device.

### Creating a .venv

Open the repository you just cloned in VSCode and initialize a virtual environment. Open a new terminal window to ensure your virtual environment is active, you should see something like `.venv` at the the start of each new line. In VSCode, your .venv should activate automatically, but in case it doesn't, type `source .venv/bin/activate` on MacOS/Linux or `.venv\Scripts\Activate.ps1` on Windows into your terminal. 

With your virtual environment activated, its time to install all of the packages you will need for this project. In the Github page for this project, you will find a file titled `requirements.txt`. Download this file and upload it to the directory you are currently working in. Now run `pip install -r requirements.txt` in your terminal. This command will install all of the packages listed in the requirements file (by the way `-r` is what is known as a flag variable, a way to tell the `pip install` command that `requirements.txt` is a file with packages that need to be installed). 

### adding a .gitignore.

If you recall from the Week 0 instructions, it is important to have a gitignore to make sure git is only tracking the files that you care about (as to not overload the git tracking system). 

Now that you have your development environment configured, its time to actually explore our data. Copy this [gitignore template](https://github.com/github/gitignore/blob/main/Python.gitignore) add add it to a `.gitignore` file in your main directory. Then commit your changes, and push to your remote git repository. You should see both the `requirements.txt` file and the `.gitignore` in the remote repo. 


## Part 2: Exploratory Data Analysis with Pandas and Jupyter Notebook

Cool, now its time to actually take a look at our NBA draft prospect data. 
