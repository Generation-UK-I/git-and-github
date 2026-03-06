# CI/CD Pipeline project

[This presentation](https://docs.google.com/presentation/d/1kJZ5C-dcgNSMhy-t_laETJUGx4WeWqYgAw3cobkiPoE/edit?usp=sharing) provides some background and context for the project.

## Pre-requisites

- AWS Account
- GitHub Account
- Website i.e. some HTML/CSS/JS and media, saved on your local computer
  - See the tutorials in the [WebDev Trinity Repository](https://github.com/Generation-UK-I/WebDev-Trinity)  if you do not have these yet

## Architecture

*Diagram needed*

The goal is to create a simple CI/CD pipeline which demonstrates a number of key technologies and concepts, as well as offering great scope for further development.

The overall workflow is:

**Local Repository** --*Git*--> **GitHub Repository** --*GitHub Actions*--> **S3 Bucket**

## Steps

### Intalling Git

Git is required to push your local files to your GitHub Repository, find a [Git tutorial here](https://github.com/Generation-UK-I/git-and-github).

There are quite a few guides in there, but the key ones for now are:

- [Introduction to Git](https://github.com/Generation-UK-I/git-and-github/blob/main/git-github.md) for understanding what it does and how it works. Then the following guides to deploy it yourself.
- [Installing Git](https://github.com/Generation-UK-I/git-and-github/blob/main/installing-git.md)

### Syncronising Your Repository

Open the folder containing your local project files in Visual Studio Code, then open a VSCode Terminal.

Create a repository in GitHub to contain your work. Once created you are provided with a few options, look for the code block under the heading `...or create a new repository on the command line`.

The code will look like the example below but personalised for your own repo, copy it into your VSCode terminal and run it.

```sh
echo "[Repository_name]" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/[url_to_your_repo]
git push -u origin main
```

If successful your GitHub repository should be connected and you should see some assets copied across, but it may not immediately synchronise fully. If so, on the left of VSCode you have a `Source Control` button. Click it, add a commit message (descriptive message explaining the purpose of the commit), then click the down arrow to the right of the `Commit` button, and select `commit & push`.

### Create an IAM User

You now need to create an IAM user in your AWS account, which is able to access the bucket; this user account is not for a human, but for GitHub Actions, so it only needs an `access key` and `secret access key` not `CLI` access.

Create the user in IAM, the user does NOT need Management Console access, and then select the option to `attach policies directly`. Search for and attach the following policies: `AmazonS3FullAccess` and `SecretsManagerReadWrite`.

Once created select the new user, navigate to `Security credentials` and generate a set of access keys. Save these keys, **you cannot access them again**, so you can add them to your repository's secrets.

### GitHub Actions

Set up GitHub Actions, and add your secrets by following the below guide.

- [GitHub Actions](https://github.com/Generation-UK-I/git-and-github/blob/main/github-actions.md) tutorial.
