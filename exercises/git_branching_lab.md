# Git Branching

## Build a Feature with Branching

### Initialize a Git Project

Create a new git repository with:

```bash
mkdir git-practice
cd git-practice
git init
```

### Create a Basic HTML File and Commit it

```bash
echo "<!DOCTYPE html><html><head><title>My Site</title></head><body><h1>Welcome</h1></body></html>" > index.html
git add index.html
git commit -m "Initial commit with basic HTML"
```

### Create and Switch to a New Branch for a Feature

```bash
git branch feature-footer
git checkout feature-footer
```

### Add a Footer to the HTML

Edit `index.html` and add this before `</body>`:

```bash
<footer><p>© 2025 My Site</p></footer>
```

Then commit...

```bash
git add index.html
git commit -m "Added footer section"
```

### Merge the Feature into Main

```bash
git checkout main
git merge feature-footer
```

### Delete the Feature Branch

```bash
git branch -d feature-footer
```

## Bonus Challenges

- Create another branch called feature-navbar and add a navigation bar.
- Use git log to view your commit history.
- Try git status and git diff to inspect changes.
