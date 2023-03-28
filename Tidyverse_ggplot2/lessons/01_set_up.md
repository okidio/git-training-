---
title: "Tidyverse directory set-up"
author: "Mary Piper"
---

## Install the `tidyverse` package

If you have not done so already, please install the suite of packages called `tidyverse` using the code below.

	install.packages("tidyverse")
	install.packages("RColorBrewer")

## Setting up the working directory

Next, let's set up our working directory and bring in data:

1. Create new project called `Tidyverse`.
	
	- Open RStudio
	- Go to the `File` menu and select `New Project`.
	- In the `New Project` window, choose `New Directory`. Then, choose `Empty Project`. Name your new directory `Tidyverse` and then "Create the project as subdirectory of:" the Desktop (or location of your choice).
	- Click on `Create Project`.
	- After your project is completed, if the project does not automatically open in RStudio, then go to the `File` menu, select `Open Project`, and choose `Tidyverse.Rproj`.
	- When RStudio opens, you will see three panels in the window.
	- Go to the `File` menu and select `New File`, and select `R Script`. The RStudio interface should now look like the screenshot below.

	![RStudio interface](../img/generic_rstudio_interface.png)

2. Click on the "Files" tab to show the file directory, and click on the "New Folder" button.

	![RStudio interface](../img/generic_wd_setup.png)

	Create the following folders:
	
	- data
	- results
	- figures
	
3. Right-click [here](https://github.com/hbctraining/Training-modules/blob/master/Tidyverse_ggplot2/data/gprofiler_results_Mov10oe.csv?raw=true) to download the file to your `data` folder (choose to `Save Link As` or `Download Linked File As`). You should see it in `data` folder in the RStudio "Files" tab.

***

[Click here to go to next lesson](https://hbctraining.github.io/Intro-to-R-flipped/lessons/15_tidyverse.html)

***
