# Odd uv behavior with .gitignore

Basically, uv behaves oddly when the `.gitignore` file has `/src` in it.

I realize that it's really weird to have `/src` in the `.gitignore`, but,
nonetheless, it is causing strange behavior.

When `.gitignore` has an entry for `/src`, uv does not include the `src`
directory in `sys.path` on build.

You can see that, in this example, after the `.venv` is built, the
`.pth` file that uv uses to set `sys.path` is empty.

This file should instead have the path to the `src` directory.

If you remove the entry from `.gitignore` things build just fine.


## Steps to reproduce

### 1. Clone the demo repo

First, clone this demo repository to your workstation:

```bash
git clone git@github.com:dusktreader/uv-gitignore-wtf.git
```

Then, change to the `uv-gitignore-wtf` folder:

```bash
cd uv-gitignore-wtf
```


### 2. Obverve the bad behavior

First, see that the `.gitignore` file has an entry for `/src`:

```bash
cat .gitignore
```

Next, try to import the `wtf` module. This should be included
in the `.venv` installed by uv. To both install the local
source package and attempt to import `wtf`, use one command:

```bash
uv run python -c "import wtf"
```

It will output something like this:

```
Using CPython 3.12.3
Creating virtual environment at: .venv
      Built uv-gitignore-wtf @ file:///home/dusktreader/git-repos/personal/uv-gitignore-wtf
Installed 1 package in 0.42ms
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'wtf'
```
Observe that it raises a ModuleNotFoundError


Next, check that the `.pth` file for the editable local package is empty:

```bash
cat .venv/lib/python3.12/site-packages/_uv_gitignore_wtf.pth
```

There should be no content in the file


### 3. Remove `/src` from `.gitignore`

Now, blank out the `.gitignore` file so that there is no longer a
`/src` entry in it.

```bash
echo "" > .gitignore
```


Next, remove the `.venv` that was created by the first `uv` command. The `uv`
cache must also be cleared. Since, apparently, it's not possible to clean
editable source distributions from uv's cache, you have to go nuclear to
make this work.

```bash
rm -r .venv/ && uv cache clean
```


Finally, run the first command again:

```bash
uv run python -c "import wtf"
```

It will produce output like this:

```
Using CPython 3.12.3
Creating virtual environment at: .venv
      Built uv-gitignore-wtf @ file:///home/dusktreader/git-repos/personal/uv-gitignore-wtf
Installed 1 package in 0.49ms
```

Notice that the final command completes without error.


Finally, check that the `.pth` file now has the path to the `src` directory:

```bash
cat .venv/lib/python3.12/site-packages/_uv_gitignore_wtf.pth
```

The file should now have contents that look something like this:

```
/home/dusktreader/git-repos/personal/uv-gitignore-wtf/src
```
