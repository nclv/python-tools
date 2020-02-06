# Virtual Environments with pipenv

Virtual environments give you a clean python setup. You create a virtual environment with pipenv by typing
```sh
pipenv install
```
You can use any ordinary pip command with pipenv. In order to add the current virtual environment to your jupyter kernels run
```sh
pipenv install ipykernel
pipenv shell
```
This will bring up a prompt similar to
```sh
(my-virtualenv-name) bash$
```
To install the kernel for jupyter notebooks run

```sh
python -m ipykernel install --user --name=my-virtualenv-name
```
