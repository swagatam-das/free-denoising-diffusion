# Creating the GitHub repository

Step-by-step, from the unpacked archive to a public URL you can cite in the paper.

---

## 1. Prerequisites

You need `git` and a GitHub account. Optionally the GitHub CLI (`gh`), which
removes a couple of manual steps.

```bash
git --version
gh --version        # optional
```

Configure your identity once, if you have not already:

```bash
git config --global user.name  "Swagatam Das"
git config --global user.email "swagatam.das@isical.ac.in"
```

---

## 2. Unpack and check the code runs

```bash
unzip free-denoising-diffusion.zip
cd free-denoising-diffusion

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

make env                           # record the versions
make test                          # 22 tests, ~15 s, all must pass
make quick                         # all nine experiments, a few minutes
```

Do not push until `make test` passes. If it does not, the cause is almost
always a NumPy version difference in `numpy.trapezoid` (added in NumPy 2.0); on
NumPy 1.x replace it with `numpy.trapz` throughout `freeddpm/functionals.py`.

---

## 3. Fill in the two placeholders

Three files contain `<user>`, which must become your GitHub username or
organisation:

```bash
grep -rn "<user>" .
```

Edit `README.md` (clone URL) and `CITATION.cff` (`repository-code`). If you
prefer, do it in one pass:

```bash
sed -i 's|<user>|YOURNAME|g' README.md CITATION.cff
```

Check the author block in `CITATION.cff` and the copyright line in `LICENSE`
while you are there.

---

## 4. Initialise the repository

```bash
git init -b main
git add .
git status                         # read this before committing
git commit -m "Reference implementation for 'Free denoising diffusion models'"
```

`git status` should show the source, the metadata files, and the contents of
`figures/` and `results/`. It should **not** show `.venv/`, `__pycache__/` or
`.pytest_cache/`; if it does, `.gitignore` was not picked up, so remove the
offending paths with `git rm -r --cached <path>` and commit again.

Committing the figures is deliberate: a referee can then see the output without
running anything. If you would rather not, uncomment the last two lines of
`.gitignore` before the first `git add`.

---

## 5. Create the remote and push

**With the GitHub CLI:**

```bash
gh repo create free-denoising-diffusion --public --source=. --remote=origin --push
```

**Without it:** create an empty repository named `free-denoising-diffusion` on
github.com — no README, no `.gitignore`, no licence, since the repository
already has all three — then:

```bash
git remote add origin https://github.com/YOURNAME/free-denoising-diffusion.git
git push -u origin main
```

---

## 6. Tag the version that the paper refers to

A bare link to a branch points at a moving target. Tag the state of the code at
submission, so the paper cites something fixed:

```bash
git tag -a v1.0.0 -m "Version accompanying the EJP submission"
git push origin v1.0.0
```

Then, on GitHub, go to **Releases → Draft a new release**, choose the `v1.0.0`
tag, and publish. The release page gives a permanent snapshot.

---

## 7. Optional: a DOI via Zenodo

Journals increasingly prefer an archived, versioned artefact to a repository
URL, and Zenodo gives one for free.

1. Sign in at zenodo.org with your GitHub account.
2. Under **GitHub**, switch on `free-denoising-diffusion`.
3. Publish a new release on GitHub (or re-publish `v1.0.0`); Zenodo archives it
   and mints a DOI.
4. Copy the DOI badge into `README.md` and the DOI itself into `CITATION.cff`
   as a top-level `doi:` field.

If you do this, cite the DOI in the paper rather than the bare GitHub URL.

---

## 8. Check what a stranger sees

```bash
cd /tmp
git clone https://github.com/YOURNAME/free-denoising-diffusion.git
cd free-denoising-diffusion
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
make test && make quick
```

If that works from a clean clone, the repository is submission-ready. The
GitHub Actions workflow in `.github/workflows/tests.yml` runs exactly this on
every push, across Python 3.10 and 3.12, and uploads the figures as artefacts;
the green tick on the repository front page is worth having before a referee
looks.

---

## 9. Add the link to the paper

Insert the following before `\begin{thebibliography}` in
`free_ddpm_ejp3.tex`, adjusting the URL:

```latex
\section*{Data availability}
The code reproducing every figure and every numerical value reported in
Section~\ref{sec:numerics} is available at
\url{https://github.com/YOURNAME/free-denoising-diffusion}
(release \texttt{v1.0.0}).
```

and replace the sentence at the end of the opening paragraph of
Section~\ref{sec:numerics},

> Code to reproduce all figures accompanies the paper.

with

```latex
Code reproducing every figure and every reported value is available at
\url{https://github.com/YOURNAME/free-denoising-diffusion}.
```

`\url` needs `\usepackage{hyperref}` or `\usepackage{url}`; `ejpecp` loads
`hyperref` already, so nothing further is required.

---

## 10. Before the paper goes out

- [ ] `make test` passes from a clean clone
- [ ] no `<user>` placeholders remain (`grep -rn "<user>" .`)
- [ ] the tag `v1.0.0` exists and is pushed
- [ ] `README.md` "Known discrepancies" agrees with what the manuscript now says
- [ ] the figures in `figures/` are the ones in the PDF
- [ ] the repository is public
