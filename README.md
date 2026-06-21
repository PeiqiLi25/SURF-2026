# SURF 2026
Repository for storing the files of SURF project.

****

## Requirements for SURF Students

All SURF students should use GitHub to upload and document their work progress during the project. The goal is not only to store files, but also to keep a clear record of what was changed, when it was changed, and why it was changed.

### 1. Before You Start

You need the following:

- A GitHub account: https://github.com/
- Git installed on your computer: https://git-scm.com/downloads
- A text/code editor, such as Visual Studio Code: https://code.visualstudio.com/download

If you have never used Git before, you may also install GitHub Desktop:

- GitHub Desktop: https://desktop.github.com/download/

GitHub Desktop is optional, but it is easier for beginners. Git is still recommended because many research and software projects use Git commands.

After installing Git, open a terminal and check that it works:

```bash
git --version
```

The first time you use Git, set your name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

Use the same email address as your GitHub account if possible.

### 2. Find This Repository on GitHub

1. Open https://github.com/.
2. Sign in to your GitHub account.
3. Search for the SURF 2026 repository, or open the repository link provided by the instructor.
4. Click `Fork` to create your own copy of the repository under your GitHub account.

Do not work directly on the main public repository unless the instructor has given you direct write permission.

### 3. Copy the Repository to Your Computer

Open your forked repository on GitHub, click `Code`, and copy the HTTPS link.

Then open a terminal on your computer and run:

```bash
git clone https://github.com/PeiqiLi25/SURF-2026

cd SURF-2026
```

Replace `YOUR-USERNAME` with your GitHub username. If the repository name is different, use the exact link shown by GitHub.

### 4. Organize Your Work

Create a folder for yourself inside the repository. For example:

```text
students/YourName/
```

You can store weekly reports, notes, code, figures, data descriptions, and results in your own folder. For example:

```text
students/YourName/week-01-progress.md
students/YourName/week-02-progress.md
students/YourName/code/
students/YourName/results/
```

(this step depends on how you organize your work)

Do not upload private information, passwords, large raw datasets, or files that should not be public.

### 5. Save Your Changes with Good Commit Messages

After editing files, check what changed:

```bash
git status
```

Add your changes:

```bash
git add .
```

Commit your changes with a clear message:

```bash
git commit -m "Add week 1 progress report"
```

A good commit message should briefly explain what you did. Examples:

- `Add week 1 progress report`
- `Update literature review notes`
- `Add initial simulation script`
- `Fix typo in project summary`
- `Add results from first experiment`

Avoid unclear messages such as:

- `update`
- `changes`
- `new file`
- `final`
- `stuff`

Each commit should represent one meaningful step of your work.

### 6. Upload Your Work to GitHub

Push your commits to your fork:

```bash
git push
```

If GitHub asks you to sign in, follow the browser login instructions.

### 7. Submit Your Work to the Main Repository

After pushing your work:

1. Open your fork on GitHub.
2. Click `Contribute`.
3. Click `Open pull request`.
4. Write a clear pull request title, such as `Add YourName week 1 progress`.
5. In the description, briefly summarize what you added or changed.
6. Submit the pull request.

The instructor or project maintainer will review the pull request before merging it into the main repository.

### 8. If You Use GitHub Desktop Instead

If you prefer GitHub Desktop:

1. Install GitHub Desktop from https://desktop.github.com/download/.
2. Sign in with your GitHub account.
3. Fork the repository on the GitHub website.
4. In your fork, click `Code`, then choose `Open with GitHub Desktop`.
5. Make or edit files on your computer.
6. In GitHub Desktop, write a clear summary in the commit message box.
7. Click `Commit`.
8. Click `Push origin`.
9. Go back to GitHub in your browser and open a pull request.

Even when using GitHub Desktop, the commit message must clearly describe the work you completed.
