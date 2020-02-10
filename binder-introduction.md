# Quick-start with Binder hub materials
{: .no_toc}

- TOC
{:toc}

## Loading the notebook

At the top of the lesson page you will find a link that will launch the [Jupyter Notebook](https://jupyter.org/) for that lesson using [Binder](https://mybinder.org/):

![Launch Binder](/assets/images/binder-launch.png)

This will start the notebook.  While Binder is setting up the environment you will see a loading page, which will automatically disappear when the notebook is ready:

![Loading](/assets/images/binder-loading.png)

You will then see the notebook:

![Main view](/assets/images/binder-main-view.png)

## Running the code

The text and code are in 'cells'.  If you click on a code cell and press Shift and Enter together, the code will run and the output (if any) will be displayed.  If the code has been run then the empty square brackets will have a number (indicating the sequence in which the code cells were executed).  The notebooks in this course assume you run the cells in order from top to bottom.

## Working in the notebook

Feel free to modify, test and play with the examples (or add your own) but beware that the Binder environment is *temporary*.  **ANYTHING YOU SAVE WILL BE LOST WHEN YOU CLOSE IT**

You can download the notebook, if you want to keep it, from the 'File' menu which will let you save the notebook as a PDF or download the actual ipynb file.  Note that you will need a Jupyter Notebook installation with C++ support or your own GitHub repository setup for Binder if you want to use the ipynb files.

## Restarting/resetting

If you make a mistake and get stuck, or your notebook starts complaining (e.g. if you try to change a declaration after running it once) then you may need to restart the environment.  The best way to do this is to click on the Kernel menu and choose 'Restart & Clear Output' which will reset the environment and clear all the output displayed in your notebook (effectively resetting it to how you started).  If you get really stuck, you can just close the notebook entirely and click the link to start a complete new session.

![Reset kernel](/assets/images/binder-restart-and-clear.png)
