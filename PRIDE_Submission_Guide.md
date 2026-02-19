# PRIDE Proteomics Data Submission Guide
### A Step-by-Step Walkthrough for Bench Scientists

> **Who is this guide for?**
> This guide is written for wet-lab researchers who need to submit proteomics data to the [PRIDE Archive](https://www.ebi.ac.uk/pride/) but have limited experience with the command line. Each step is explained in plain language, with commands you can copy and paste directly.

---

## ⚠️ Why This Guide Exists — Important Background

The standard PRIDE submission tool (the **PX Submission Tool**) is a Java-based desktop application. However, **Oracle changed its Java licence**, meaning the version of Java bundled with university-managed computers is no longer permitted to run third-party tools like this. You will likely see an error if you try to double-click the tool as you normally would.

To get around this, we install a **free, open-source version of Java** (OpenJDK) through a package manager called Conda, and launch the tool from there.

Additionally, the university firewall **blocks the network port** that the PX Submission Tool normally uses to upload data directly to PRIDE. This means the built-in upload function inside the tool simply will not work from a university machine.

The solution is to use **Globus**, a secure file transfer service that the university supports, to perform the actual upload.

---

## 🗺️ Overview of the Workflow

```
1. Install Miniforge (Conda)
2. Create a Conda environment with Java
3. Install the pride-checksum tool
4. Install Globus Connect Personal
5. Prepare your submission files (raw, search results, .px file)
6. Run the PX Submission Tool to create your submission package
7. Compute checksums with pride-checksum
8. Register a new submission on the PRIDE website
9. Upload your files via Globus
10. Finalise your submission on the PRIDE website
```

---

## 💡 A Note on File Locations

> **Best practice:** If your raw data files are stored on a **remote network drive or server**, copy them to your **local computer** before starting this process. Running the submission tool or the checksum tool against files on a network drive is unreliable and likely to fail, especially the checksum step. Use a local folder (e.g., `C:\Users\YourName\Documents\submission_pride\`) throughout this guide.

---

## Step 1 — Install Miniforge (Conda Package Manager)

Conda is a tool that lets you install software packages and manage different software environments without needing administrator rights or affecting other programs on your computer. **Miniforge** is a lightweight, free version of Conda.

### Download and install

1. Download the Windows installer from this link:
   **[Miniforge3-Windows-x86_64.exe](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Windows-x86_64.exe)**

   *(Full project page: https://github.com/conda-forge/miniforge)*

2. Run the installer. Accept the defaults — you do **not** need to install it system-wide.

3. Once installed, search for **"Miniforge Prompt"** in your Windows Start menu and open it. This is the command-line window you will use for all subsequent steps.

### Verify the installation

In the Miniforge Prompt, type:

```bash
conda -V
```

You should see a version number printed, e.g. `conda 24.x.x`. If you do, Conda is working correctly.

---

## Step 2 — Create a Dedicated Environment for PRIDE Submission

A Conda **environment** is an isolated workspace with its own set of software. This keeps your PRIDE tools separate from anything else on your computer and avoids conflicts.

```bash
conda create -n pride
```

- `-n pride` gives your environment the name `pride`. You can call it anything you like.
- When prompted, type `y` and press Enter to confirm.

**Activate the environment** (you must do this every time you open a new Miniforge Prompt):

```bash
conda activate pride
```

You will see `(pride)` appear at the beginning of your prompt, confirming you are inside the environment.

---

## Step 3 — Install Java (OpenJDK)

This installs a free, licence-compatible version of Java that is allowed to run the PX Submission Tool on university computers.

```bash
conda install conda-forge::openjdk
```

- This fetches OpenJDK directly from the community-maintained `conda-forge` repository.
- When prompted, type `y` and press Enter.

---

## Step 4 — Install pip and pride-checksum

**pip** is a package installer for Python tools. Install it into your environment:

```bash
conda install pip
```

Then use pip to install **pride-checksum**, a tool that computes the file checksums required by PRIDE:

```bash
pip install pride-checksum
```

> **Why not use the checksum function inside the PX Submission Tool?**
> The built-in checksum function in the PX Submission Tool will fail if your files are on a network drive, and can also be unreliable in other situations. `pride-checksum` is a dedicated command-line tool that is more robust and is the recommended approach when using Globus for upload.

---

## Step 5 — Install Globus Connect Personal

Globus is a secure, university-supported file transfer service. It is required because the university firewall blocks the port that the PX Submission Tool uses for direct uploads to PRIDE.

1. Go to: **https://www.globus.org/globus-connect-personal**
2. Download and install **Globus Connect Personal** for Windows.
3. Sign in with your **university credentials** (select your institution from the list).
4. Follow the setup wizard to create a **personal endpoint** — this represents your local computer in the Globus system.

> Once set up, Globus runs quietly in the background and makes your local computer visible as a file transfer endpoint. You will use the Globus web interface later to move files to PRIDE.

---

## Step 6 — Run the PX Submission Tool

The PX Submission Tool is the official PRIDE application for assembling your submission package. It guides you through associating raw files, search result files, and metadata, and it generates a `.px` submission file.

> **Download the tool** from: https://www.ebi.ac.uk/pride/help/archive/submission/px-submission-tool
> Unzip it to a location on your local computer (e.g., `C:\Users\YourName\Downloads\px-submission-tool-2.11.2\`).

With your `pride` Conda environment active, start the tool using `java -jar`:

```bash
java -jar C:\Users\YourName\Downloads\px-submission-tool-2.11.2\px-submission-tool-2.11.2.jar
```

> 📝 **Adjust the path** to match wherever you saved the tool on your computer. The version number in the filename may also differ.

**Inside the tool:**

- Add your **raw data files** (e.g., `.raw`, `.wiff`, `.d` folders).
- Add your **search result files** (e.g., `.mzid`, `.dat`, `.pep.xml`).
- Fill in the required **metadata** (species, instrument, contact information, etc.).
- Complete the wizard steps until you reach the submission/upload step.

> ⚠️ **Do not attempt to upload directly from within the tool.** The upload will fail due to the university firewall. Instead, use the tool only to generate the submission package (the `.px` file and the accompanying file manifest). You will upload via Globus in a later step.

When finished, the tool will produce a folder (e.g., `submission_pride`) containing your files and a `submission.px` manifest. Note the location of this folder.

---

## Step 7 — Compute Checksums with pride-checksum

PRIDE requires a checksum file to verify data integrity. With your `pride` environment active, run:

```bash
pride_checksum --out_path submission_pride --files_dir submission_pride
```

- `--files_dir submission_pride` — the folder containing all your submission files (raw data, search results, `.px` file). **Replace `submission_pride` with the actual path to your folder if it differs.**
- `--out_path submission_pride` — the folder where the checksum file will be written. Usually the same as `--files_dir`.

This will generate a checksum file (e.g., `checksum.txt`) inside your submission folder. This must be included in your Globus upload.

> ✅ After this step, your submission folder should contain:
> - Raw data files (`.raw`, `.wiff`, etc.)
> - Search result files (`.mzid`, `.dat`, etc.)
> - The submission manifest (`.px` file)
> - The checksum file (`checksum.txt` or similar)

---

## Step 8 — Register a New Submission on the PRIDE Website

1. Log in to PRIDE at: **https://www.ebi.ac.uk/pride/login**
   *(Use your PRIDE account credentials. Create an account if you do not have one.)*

2. Navigate to **"Submit Data"** and start a **new submission**.

3. Complete the metadata form (project title, abstract, keywords, etc.) and proceed until PRIDE asks you to upload your files.

4. PRIDE will generate a **private upload space** on Globus for your submission. You will receive a link or a folder identifier for this space — keep note of it.

---

## Step 9 — Upload Files via Globus

1. Go to the **Globus web interface**: **https://www.globus.org/**

2. Log in with your university credentials.

3. You will see a **two-panel file manager**:
   - In one panel, navigate to your **personal endpoint** (your local computer) and browse to your `submission_pride` folder.
   - In the other panel, search for the **PRIDE upload endpoint** that was assigned to you in Step 8 (PRIDE will provide the endpoint name, usually something like `ebi#pride`).

4. Select all files in your submission folder and click **"Start Transfer"**.

5. Globus will handle the secure upload in the background. You will receive an email confirmation when the transfer is complete.

---

## Step 10 — Finalise Your Submission on PRIDE

1. Return to the PRIDE submission page: **https://www.ebi.ac.uk/pride/login**

2. Confirm that all files have been uploaded successfully (PRIDE will show the file list).

3. Complete any remaining submission steps and click **Submit**.

4. You will receive a confirmation email with your **PXD accession number** — the permanent identifier for your dataset, which you will use in your manuscript.

---

## 🔁 Quick Reference — Commands Summary

| Step | Command |
|------|---------|
| Check Conda is installed | `conda -V` |
| Create the pride environment | `conda create -n pride` |
| Activate the environment | `conda activate pride` |
| Install OpenJDK | `conda install conda-forge::openjdk` |
| Install pip | `conda install pip` |
| Install pride-checksum | `pip install pride-checksum` |
| Launch PX Submission Tool | `java -jar C:\path\to\px-submission-tool-X.X.X.jar` |
| Compute checksums | `pride_checksum --out_path submission_pride --files_dir submission_pride` |

---

## 🆘 Troubleshooting

**"Java not found" or Java errors when launching the tool**
→ Make sure you have activated the `pride` Conda environment (`conda activate pride`) before running `java -jar ...`. The Java installed by Conda is only available inside this environment.

**Checksum step fails or hangs**
→ Ensure all your files are on your **local drive**, not a network or remote drive. Network file access during checksum computation is a known cause of failures.

**Globus transfer endpoint not found**
→ Make sure Globus Connect Personal is running on your computer (check the system tray). Also confirm you are signed in with your university credentials.

**The PX Submission Tool closes or crashes**
→ Check that you are using the OpenJDK version installed by Conda and not any other Java version on the system. If the Miniforge Prompt shows `(pride)` at the start of the line, you are using the correct Java.

---

*Guide prepared for internal use — University of Dundee, School of Life Sciences.*
*For questions, contact your local bioinformatics support team.*
