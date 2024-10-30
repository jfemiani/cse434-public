# Lab 0: Setting Up Anaconda, VSCode, and `cse434` Environment

## Objective
Ensure you have a proper setup to work with Python, Jupyter notebooks, and PyTorch using Anaconda and VSCode, and demonstrate your setup by exporting your work as a PDF.


---

## Steps

### 1. Install Anaconda
- Download and install Anaconda from the provided link.
- **Windows Users**: During installation, check the box to add Anaconda to your PATH environment variable. Always use the **Anaconda Prompt** (not the regular Command Prompt) for running the following commands.
- **Apple Users**: If you encounter a “Permission Denied” error during installation, run the installer with admin privileges. You may also need to update `.zshrc` or `.bash_profile` to include the path to Anaconda.

### 2. Set Up VSCode
- Download and install **Visual Studio Code** from the provided link.
- Install the **Python extension** for VSCode from the Extensions marketplace.
- **Windows Users**: Open VSCode from Command Prompt or PowerShell by typing `code .` (make sure VSCode is added to your PATH during installation). If you’re using Bash-like commands, consider installing **Git Bash**, **Cygwin**, or **Windows Subsystem for Linux (WSL)** to use Linux commands.
- **Apple Users**: You may need to add VSCode to your PATH manually by typing:
  ```bash
  export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
  ```

### 3. Create the `cse434` Conda Environment
- Open the **Anaconda Prompt** (Windows) or **Terminal** (Apple) and run:
  ```bash
  conda create --name cse434 python=3.11
  conda activate cse434
  ```

### 4. Install Required Packages
- While in the `cse434` environment, install packages:
  ```bash
  conda install scipy pytorch torchvision torchaudio -c pytorch
  conda install jupyterlab ipython
  ```

### 5. Select the `cse434` Conda Environment in VSCode
- Open VSCode and press **Ctrl+Shift+P** (Windows) or **Cmd+Shift+P** (Mac) to open the Command Palette.
- Type `Python: Select Interpreter` and hit Enter.
- Select the Python interpreter that shows `cse434`. It should look something like `Python 3.11.0 ('cse434': conda)`.
- Verify the `cse434` environment is selected by checking the bottom-left corner of VSCode, where it should display `cse434`.

### 6. Verify Installation in Jupyter Notebook
- Launch JupyterLab by typing `jupyter lab` in your terminal (with `cse434` activated).
- Create a new Python notebook and execute:
  ```python
  import torch
  import scipy
  
  print(f"PyTorch version: {torch.__version__}")
  print(f"SciPy version: {scipy.__version__}")
  ```
- Take a screenshot of VSCode showing the terminal with the activated `cse434` environment and the output of the version check.

### 7. Add the Screenshot to the Notebook
- In Jupyter notebooks, add your screenshot by creating a new **Markdown cell** and pasting your screenshot.

### 8. Install Pandoc
- **Windows**: Run the following in Anaconda Prompt:
  ```bash
  conda install -c conda-forge miktex
  conda install -c conda-forge "pandoc<3.2.1"
  ```
- **Mac**: Run:
  ```bash
  brew install pandoc
  ```
  _(Install Homebrew if necessary)_

### 9. Export Your Notebook as a PDF
- In JupyterLab, go to **File > Export Notebook As > PDF**.
- Review the PDF to ensure none of the content is clipped or cut off.

### 10. Submit Your Work
Submit the generated PDF file, ensuring it includes:
- Output of the code showing PyTorch and SciPy versions.
- Screenshot of VSCode setup with the `cse434` environment embedded within the notebook.
