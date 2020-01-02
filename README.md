# Installation pour le développement
Ci-dessous la procédure d'installation d'un environnement de développement avec ```pipenv ```.
```sh
# Install dependencies
pipenv install --dev

# Setup pre-commit and pre-push hooks (optional)
pipenv run pre-commit install -t pre-commit
pipenv run pre-commit install -t pre-push
```
Les packages suivants sont directements disponibles :
```sh
black, isort, flake8, pytest (-s -v), pytest --cov --cov-fail-under=100
```

Voici maintenant quelques indications :
 - pour exécuter un package ```pipenv run <package>```.
 *Tous les packages sont exécutés avant chaque commit et chaque push si les hooks correspondants sont activés.*
 - pour passer outre les hooks pre-commit et pre-push ```git commit/push --no-verify```.

### Profiling
Il est utile d'évaluer le temps d'exécution de chaque fonction lors de la génération de la carte pour optimiser le programme.
```sh
bash performances.sh perf.prof pyhack.py