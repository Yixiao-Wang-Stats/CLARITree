# Publishing CLARITree

The PyPI distribution name is `claritree`. The Python import name is
`clari_tree`. They intentionally do not need to match:

```bash
pip install claritree
```

```python
from clari_tree import CLARITree
```

## 1. Prepare Git

Use this repository; do not create a separate repository for the package.
Prepare each release on a short-lived branch and commit the release changes
together:

```bash
git switch -c release/pypi-0.1.1
git status
git add .gitignore pyproject.toml CMakeLists.txt README.md PUBLISHING.md
git diff --cached
git commit -m "Prepare PyPI release 0.1.1"
git push -u origin release/pypi-0.1.1
```

Do not commit PyPI API tokens, `.pypirc`, `dist/`, or `build/`.

## 2. Create Accounts

Create separate accounts on:

- TestPyPI: <https://test.pypi.org/account/register/>
- PyPI: <https://pypi.org/account/register/>

Verify both email addresses, enable two-factor authentication, and create an
API token on each site. TestPyPI and PyPI accounts and tokens are separate.

## 3. Create a Release Environment

From the repository root:

```bash
python3 -m venv .release-venv
source .release-venv/bin/activate
python -m pip install --upgrade pip build twine
```

The local machine must also provide CMake, a C++17 compiler, and Eigen 3.
On Linux, install the wheel repair tools as well:

```bash
python -m pip install auditwheel patchelf
```

## 4. Build and Check

Remove only generated release artifacts, then build:

```bash
rm -rf build dist
python -m build
python -m twine check dist/*
```

Inspect the files:

```bash
ls -lh dist
python -m zipfile -l dist/*.whl
tar -tzf dist/*.tar.gz
```

The wheel must contain both `clari_tree/__init__.py` and a compiled
`clari_tree/_core` extension.

### Repair the Linux Wheel

PyPI rejects wheels with a plain `linux_x86_64` platform tag. On Linux, repair
the wheel before uploading:

```bash
rm -rf wheelhouse
mkdir -p wheelhouse
python -m auditwheel show dist/*-linux_x86_64.whl
python -m auditwheel repair dist/*-linux_x86_64.whl --wheel-dir wheelhouse
python -m twine check dist/*.tar.gz wheelhouse/*.whl
```

The repaired filename must contain a supported tag such as
`manylinux_2_34_x86_64`, not `linux_x86_64`. The exact manylinux version
depends on the system libraries used during compilation.

## 5. Test the Wheel Locally

```bash
python3 -m venv /tmp/claritree-release-test
source /tmp/claritree-release-test/bin/activate
python -m pip install --upgrade pip
python -m pip install wheelhouse/*.whl
python -c "from clari_tree import CLARITree; print(CLARITree)"
deactivate
source .release-venv/bin/activate
```

## 6. Upload to TestPyPI

```bash
PYTHON_KEYRING_BACKEND=keyring.backends.null.Keyring \
python -m twine upload --repository testpypi --verbose \
    dist/*.tar.gz wheelhouse/*.whl
```

When prompted, use `__token__` as the username and the complete TestPyPI token
as the password. Disabling keyring avoids SecretService/DBus warnings on
headless login servers.

Test the uploaded package in another environment. Install dependencies from
the real PyPI first because TestPyPI does not mirror them:

```bash
python3 -m venv /tmp/claritree-testpypi
source /tmp/claritree-testpypi/bin/activate
python -m pip install numpy
python -m pip install --index-url https://test.pypi.org/simple/ --no-deps claritree
python -c "from clari_tree import CLARITree; print(CLARITree)"
```

## 7. Upload to PyPI

Only after the TestPyPI installation succeeds and production-compatible
binary wheels have been built:

```bash
source .release-venv/bin/activate
PYTHON_KEYRING_BACKEND=keyring.backends.null.Keyring \
python -m twine upload --verbose dist/*.tar.gz wheelhouse/*.whl
```

Use `__token__` and the real PyPI token. A published filename/version cannot
be replaced. If a release is wrong, fix it and increment the version, for
example from `0.1.0` to `0.1.1`.

## 8. Tag the Release

After the production upload succeeds:

```bash
git tag -a v0.1.1 -m "CLARITree 0.1.1"
git push origin v0.1.1
```

## Binary Wheel Scope

This project contains a compiled C++ extension. A local build produces a wheel
only for the current operating system, CPU architecture, and Python version.
A wheel named `*-linux_x86_64.whl` built on an arbitrary Linux host is rejected
by PyPI and must not be uploaded. `auditwheel repair` can produce an accepted
manylinux tag, but a wheel repaired to `manylinux_2_34` only supports systems
with glibc 2.34 or newer.

Before uploading to production PyPI, use a wheel-building service such as
`cibuildwheel` in CI to build and test wheels for every supported Python and
operating-system combination. Linux wheels should be repaired and tagged as
manylinux-compatible. The source distribution remains useful for users who
have CMake, a C++17 compiler, and Eigen 3 installed.
